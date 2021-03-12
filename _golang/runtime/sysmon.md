# 监视器

[TOC]





Go 语言的系统监控在 runtime 中起到了很重要的作用，它在随着程序一起启动，并运行在独立的内核线程上。它会在 Go程序的声明周期内不断的循环，轮询网络、抢占长期运行或者处于系统调用的 Goroutine 以及触发垃圾回收。

```go
func main() {
	...
	if GOARCH != "wasm" {
		systemstack(func() {
			newm(sysmon, nil)
		})
	}
	...
}
```

在 `runtime.main` 启动监视器，通过 `runtie.newm` 函数，会创建新的内核线程 M 来运行 sysmon，这个与普通的 G 不同，监视器的运行不涉及 P。



监视器通过主体逻辑在一个 for 循环中，在循环中会不断通过 usleep 方法进行休眠，初始的休眠时间是 20μs, 最终休眠时间会稳定在 10ms。

```go
func sysmon() {
	...
	delay := uint32(0)
	for {
		if idle == 0 { // start with 20us sleep...
			delay = 20
		} else if idle > 50 { // start doubling the sleep after 1ms...
			delay *= 2
		}
		if delay > 10*1000 { // up to 10ms
			delay = 10 * 1000
		}
		usleep(delay)
       	...
    }
}
```



调取器的主要工作有：

- 运行计时器 

- 轮询网络  netpoll

- 抢占处理器 retake

- 垃圾回收 forcegc

## 检查死锁 

在进入核心的 for 循环之间，监视器会调用 `runtime.checkdead` 检查是否存在死锁。



```go
// go 1.15
func sysmon() {
	lock(&sched.lock)
	sched.nmsys++
	checkdead()
	unlock(&sched.lock)

	lasttrace := int64(0)
	idle := 0 // how many cycles in succession we had not wokeup somebody
	delay := uint32(0)
	for {
        ...
    }        
}        
```



## 网络轮询

如果上一次轮询网络已经过去了 10ms，系统监控还会`非阻塞`调用 `runtime.netpoll` 检查网络轮询器是已经 ready 的网络连接， 它会返回一个 `gList` 对象，里面是一系列可运行的 Goroutine 列表。gList 会被 `runtime.injectglist` 将所有处于就绪状态的 Goroutine 加入全局运行队列中, 并 G 的状态从 `_Gwaiting` 切换至 `_Grunnable`。

```go
func sysmon() {
	...
	for {
		...
		// poll network if not polled for more than 10ms
		lastpoll := int64(atomic.Load64(&sched.lastpoll))
		if netpollinited() && lastpoll != 0 && lastpoll+10*1000*1000 < now {
			atomic.Cas64(&sched.lastpoll, uint64(lastpoll), uint64(now))
			list := netpoll(0) // 传入 0 进行非阻塞调用，返回可运行的网络连接的 goroutine
			if !list.empty() {
				incidlelocked(-1)
				injectglist(&list)
				incidlelocked(1)
			}
		}
		...
	}
}


```



## 抢占调度

sysmon 依赖 sysmontick 来记录 P 的调度信息，每个 P 都有一个 sysmontick。

```go
// 记录每次检查的信息
type sysmontick struct {
    schedtick   uint32 // 处理器 P 的调度次数
    schedwhen   int64  // 处理器 P 上次调度时间
    syscalltick uint32 // 系统调用的次数
    syscallwhen int64  // 系统调用的时间
}

type p struct {
    ...
    schedtick   uint32     // incremented on every scheduler call
	syscalltick uint32     // incremented on every system call
	sysmontick  sysmontick // last tick observed by sysmon
    ...
}
```



系统监控会在循环中调用 `runtime.retake` 遍历所有的 P，抢占运行时间较长的 G 以及因为系统调用而阻塞的 P。

当处理器处于 `_Prunning` 或者 `_Psyscall` 状态时，并且已经运行超过 10ms，会通过通过 `runtime.preemptone` 抢占当前处理器；

当处理器处于 `_Psyscall` 状态时，在满足以下两种情况下会调用 `runtime.handoffp `来解绑 P 和 M：

- 当处理器的运行队列不为空或者不存在空闲处理器时；
- 当系统调用时间超过了 10ms 时；

```go

func sysmon(){
    ...
    if retake(now) != 0 {
        idle = 0
    } else {
        idle++
    }
    ...
}



// 抢占的时间阈值 10ms
const forcePreemptNS = 10 * 1000 * 1000 


func retake(now int64) uint32 {
    ...
	for i := 0; i < len(allp); i++ { // 遍历所有的 P
        _p_ := allp[i]
        if _p_ == nil {
            continue
        }
        pd := &_p_.sysmontick
        s := _p_.status
        if s == _Prunning || s == _Psyscall {
            // Preempt G if it's running for too long.
            t := int64(_p_.schedtick)
            if int64(pd.schedtick) != t {
                pd.schedtick = uint32(t)
                pd.schedwhen = now // 记录当前调度时间
            // 如果运行超过 10ms，触发进行抢占。
            } else if pd.schedwhen+forcePreemptNS <= now {
                preemptone(_p_)
            }
        }

        // 系统调用        
		if s == _Psyscall {
			if runqempty(_p_) && atomic.Load(&sched.nmspinning)+atomic.Load(&sched.npidle) > 0 && pd.syscallwhen+10*1000*1000 > now {
				continue
			}
			if atomic.Cas(&_p_.status, s, _Pidle) { // P 状态变为为空闲状态
				n++
				_p_.syscalltick++
				handoffp(_p_) // 强制卸载P, 然后用 startm 启动新的 M 来运行 P上的其他 G
			}
		}    
        ...           
}

```



## 垃圾回收

在监视器的每个循环周期还会调用  `runtime.gcTrigger.test` 判断是否要出发 gc 回收。触发 GC 回收的类型为 `gcTriggerTime`。

如果需要触发垃圾回收，会通过 `injectglist` 将用于垃圾回收的 `Goroutine` 也就是 `forcegc.g` 加入全局队列，让调度器选择合适的处理器去执行。

```go
 

func sysmon() {
	...
	for {
		...
		if t := (gcTrigger{kind: gcTriggerTime, now: now}); t.test() && atomic.Load(&forcegc.idle) != 0 {
			lock(&forcegc.lock)
			forcegc.idle = 0
			var list gList
			list.push(forcegc.g)
			injectglist(&list)
			unlock(&forcegc.lock)
		}
		...
	}
}

```



