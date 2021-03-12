---
layout: single
title:  "signal"
date:   2020-04-20 10:50:46 +0800
permalink: /golang/part1/signal
toc: true
toc_sticky: true
---



[TOC]



# go signal 包

## 信号类型

`SIGKILL`  和 `SIGSTOP` 信号可能不会被程序捕获，因此不受 singal 包的影响。

*同步信号*是由程序执行中的错误触发的信号:SIGBUS、SIGFPE 和 SIGSEGV。只有在由程序执行引起时才会认为是同步的，而使用 `os.Process` 发送时则不会。

杀死或杀死程序或一些类似的机制。一般来说，除了下面讨论的，Go程序将同步信号转换为运行时恐慌。

其余的信号是*异步信号*。它们不是由程序错误触发的，而是由内核或其他程序发送的。

在异步信号中，当程序失去其控制终端时发送 `SIGHUP` 信号。当用户在控制终端按下中断字符(默认为^C (Control-C))时，SIGINT信号被发送。当用户在控制终端按下quit字符(默认为^\ (control -反斜杠))时发送SIGQUIT信号。通常，您可以通过按^C使一个程序退出，并且您可以通过按^\使它退出堆栈转储。

## 在Go程序实现中信号的默认行为

默认情况下，同步信号被转换为 *runtime panic*。

SIGHUP、SIGINT 或 SIGTERM 信号会导致*程序退出*。

SIGQUIT、SIGILL、SIGTRAP、SIGABRT、SIGSTKFLT、SIGEMT 或 SIGSYS 信号会导致*程序通过堆栈dump退出*。

SIGTSTP、SIGTTIN 或 SIGTTOU 信号获取系统默认行为(shell使用这些信号进行作业控制)。SIGPROF 信号由 Go 运行时直接处理，以实现runtime. cpuprofile。其他信号将被捕捉，但不会采取任何行动。

如果 Go 程序在启动时忽略了 SIGHUP 或 SIGINT (信号处理程序设置为SIG_IGN)，它们将继续被忽略。

如果 Go 程序以一个非空信号掩码开始，那么通常会采用该掩码。但是，有些信号是显式解除阻塞的:同步信号 SIGILL、SIGTRAP、SIGSTKFLT、SIGCHLD、SIGPROF，以及在GNU/Linux上的信号32 (SIGCANCEL)和33 (SIGSETXID) (SIGCANCEL和SIGSETXID由glibc内部使用)。由os启动的子进程。或由os/exec 包继承修改后的信号掩码。

## 在Go程序中改变信号的行为

singal 包中的函数允许程序改变 Go 程序处理信号的方式。

Notify 禁用给定一组异步信号的默认行为，而是通过一个或多个已注册的通道交付它们。具体来说，它适用于信号 SIGHUP、SIGINT、SIGQUIT、SIGABRT 和 SIGTERM。它还适用于作业控制信号 SIGTSTP、SIGTTIN 和 SIGTTOU，在这种情况下，系统默认行为不会发生。它也适用于不引起其他动作的信号:SIGUSR1、SIGUSR2、SIGPIPE、SIGALRM、SIGCHLD、SIGURG、SIGXCPU、SIGXFSZ、SIGVTALRM、SIGWINCH、SIGIO、SIGPWR、SIGSYS、SIGINFO、SIGTHR、SIGWAITING、SIGLWP、SIGFREEZE、SIGTHAW、SIGLOST、SIGXRES、SIGJVM1、SIGJVM2，以及系统上使用的任何实时信号。

注意，并非所有这些信号都在所有系统上可用。

如果程序启动时忽略了 SIGHUP 或 SIGINT，并且为其中一个信号调用 Notify，则将为该信号安装一个信号处理程序，该信号将不再被忽略。如果稍后为该信号调用 Reset 或 Ignore，或在传递给该信号的所有通道上调用 Stop，则该信号将再次被忽略。

Reset 将恢复该信号的系统默认行为，而Ignore将导致系统完全忽略该信号。

如果程序以非空信号掩码开始，一些信号将如上所述显式解除阻塞。如果为阻塞信号调用 Notify，它将被解除阻塞。如果稍后为该信号调用 Reset，或在所有传递的用于通知该信号的通道上调用 Stop，则该信号将再次被阻塞。



## SIGPIPE

当一个 Go 程序写入一个损坏的管道时，内核将发出一个 SIGPIPE 信号。

如果程序没有调用 Notify 来接收 SIGPIPE 信号，则行为取决于文件描述符的编号。在文件描述符1或2(标准输出或标准错误)上写入损坏的管道将导致程序以SIGPIPE信号退出。对其他文件描述符上的破管道的写操作将不会对SIGPIPE信号采取任何操作，并且写操作将会失败并产生一个 EPIPE 错误。

如果程序调用 Notify 来接收 SIGPIPE 信号，则文件描述符的编号并不重要。SIGPIPE 信号将被发送到通知通道，写操作将失败，并出现EPIPE错误。

这意味着，在默认情况下，命令行程序的行为将与典型的 Unix 命令行程序类似，而其他程序在写入一个封闭的网络连接时不会因为使用 SIGPIPE 而崩溃。 



## func

### Notify()

```go
func Notify(c chan<- os.Signal, sig ...os.Signal)
```



Notify 使包信号将传入信号转发给通道 c，如果没有信号，则将所有传入信号转发给c，否则仅发送提供的信号。

包信号不会阻塞发送给 c: 调用者必须确保 c 有足够的缓冲区空间来保持预期的信号速率。对于只用于通知一个信号值的通道，大小为1的缓冲区就足够了。

允许使用同一通道多次调用 Notify: 每次调用扩展发送到该通道的信号集。从集合中移除信号的唯一方法是调用 Stop。

允许使用不同的通道和相同的信号多次调用通知:每个通道分别接收传入信号的副本。



```go
func sig1() {
  fmt.Println("pid = ", os.Getpid())
  
	// 传入的通道要有足够的缓冲区，否则信号可能丢失
	c := make(chan os.Signal, 1)
	signal.Notify(c,  syscall.SIGINT, syscall.SIGTERM)

	// Block until a signal is received.
	s := <-c
	fmt.Println("Got signal:", s) // Got signal: interrupt
}
```

发送信号：

```bash
#kill -s SIGINT 72984
#kill -s SIGTERM 73005
pid =  73005
Got signal: terminated
```

**默认全部信号**

```go
func sigAll() {
   c := make(chan os.Signal, 1)

   // 没有传具体都信号参数，将会默认把所有信号都传给 c
   signal.Notify(c)

   for {
      s := <-c
      fmt.Println("Got signal:", s)
   }

}
//测试
//Got signal: urgent I/O condition
//Got signal: window size changes
//Got signal: urgent I/O condition
//Got signal: window size changes
//Got signal: interrupt
//Got signal: hangup
```



### Stop()

```go
func Stop(c chan<- os.Signal)
```

Stop 使包信号停止向 c 中继传入信号，解除之前所有调用的效果，使用c通知。当Stop返回时，保证c不会再收到信号。



# unix 信号

查看信号：

```bash
$ kill -l
1) SIGHUP 2) SIGINT 3) SIGQUIT 4) SIGILL
5) SIGTRAP 6) SIGABRT 7) SIGBUS 8) SIGFPE
9) SIGKILL 10) SIGUSR1 11) SIGSEGV 12) SIGUSR2
13) SIGPIPE 14) SIGALRM 15) SIGTERM 17) SIGCHLD
18) SIGCONT 19) SIGSTOP 20) SIGTSTP 21) SIGTTIN
22) SIGTTOU 23) SIGURG 24) SIGXCPU 25) SIGXFSZ
26) SIGVTALRM 27) SIGPROF 28) SIGWINCH 29) SIGIO
30) SIGPWR 31) SIGSYS 34) SIGRTMIN 35) SIGRTMIN+1
36) SIGRTMIN+2 37) SIGRTMIN+3 38) SIGRTMIN+4 39) SIGRTMIN+5
40) SIGRTMIN+6 41) SIGRTMIN+7 42) SIGRTMIN+8 43) SIGRTMIN+9
44) SIGRTMIN+10 45) SIGRTMIN+11 46) SIGRTMIN+12 47) SIGRTMIN+13
48) SIGRTMIN+14 49) SIGRTMIN+15 50) SIGRTMAX-14 51) SIGRTMAX-13
52) SIGRTMAX-12 53) SIGRTMAX-11 54) SIGRTMAX-10 55) SIGRTMAX-9
56) SIGRTMAX-8 57) SIGRTMAX-7 58) SIGRTMAX-6 59) SIGRTMAX-5
60) SIGRTMAX-4 61) SIGRTMAX-3 62) SIGRTMAX-2 63) SIGRTMAX-1
64) SIGRTMAX
```

列表中，编号为1 ~ 31的信号为传统UNIX支持的信号，是不可靠信号(非实时的)，编号为32 ~ 63的信号是后来扩充的，称做可靠信号(实时信号)。不可靠信号和可靠信号的区别在于前者不支持排队，可能会造成信号丢失，而后者不会。



1) SIGHUP

本信号在用户终端连接(正常或非正常)结束时发出, 通常是在终端的控制进程结束时, 通知同一session内的各个作业, 这时它们与控制终端不再关联。

登录Linux时，系统会分配给登录用户一个终端(Session)。在这个终端运行的所有程序，包括前台进程组和 后台进程组，一般都属于这个 Session。当用户退出Linux登录时，前台进程组和后台有对终端输出的进程将会收到SIGHUP信号。这个信号的默认操作为终止进程，因此前台进 程组和后台有终端输出的进程就会中止。不过可以捕获这个信号，比如wget能捕获SIGHUP信号，并忽略它，这样就算退出了Linux登录，wget也 能继续下载。

此外，对于与终端脱离关系的守护进程，这个信号用于通知它重新读取配置文件。

2) SIGINT

程序终止(interrupt)信号, 在用户键入INTR字符(通常是Ctrl-C)时发出，用于通知前台进程组终止进程。

3) SIGQUIT

和SIGINT类似, 但由QUIT字符(通常是Ctrl-\)来控制. 进程在因收到SIGQUIT退出时会产生core文件, 在这个意义上类似于一个程序错误信号。

4) SIGILL

执行了非法指令. 通常是因为可执行文件本身出现错误, 或者试图执行数据段. 堆栈溢出时也有可能产生这个信号。

5) SIGTRAP

由断点指令或其它trap指令产生. 由debugger使用。

6) SIGABRT

调用abort函数生成的信号。

7) SIGBUS

非法地址, 包括内存地址对齐(alignment)出错。比如访问一个四个字长的整数, 但其地址不是4的倍数。它与SIGSEGV的区别在于后者是由于对合法存储地址的非法访问触发的(如访问不属于自己存储空间或只读存储空间)。

8) SIGFPE

在发生致命的算术运算错误时发出. 不仅包括浮点运算错误, 还包括溢出及除数为0等其它所有的算术的错误。

9) SIGKILL

用来立即结束程序的运行. 本信号不能被阻塞、处理和忽略。如果管理员发现某个进程终止不了，可尝试发送这个信号。

10) SIGUSR1

留给用户使用

11) SIGSEGV

试图访问未分配给自己的内存, 或试图往没有写权限的内存地址写数据.

12) SIGUSR2

留给用户使用

13) SIGPIPE

管道破裂。这个信号通常在进程间通信产生，比如采用FIFO(管道)通信的两个进程，读管道没打开或者意外终止就往管道写，写进程会收到SIGPIPE信号。此外用Socket通信的两个进程，写进程在写Socket的时候，读进程已经终止。

14) SIGALRM

时钟定时信号, 计算的是实际的时间或时钟时间. alarm函数使用该信号.

15) *SIGTERM*

程序结束(terminate)信号, 与SIGKILL不同的是该信号可以被阻塞和处理。通常用来要求程序自己正常退出，shell命令kill缺省产生这个信号。如果进程终止不了，我们才会尝试SIGKILL。

17) SIGCHLD

子进程结束时, 父进程会收到这个信号。

如果父进程没有处理这个信号，也没有等待(wait)子进程，子进程虽然终止，但是还会在内核进程表中占有表项，这 时的子进程称为僵尸进程。这种情 况我们应该避免(父进程或者忽略SIGCHILD信号，或者捕捉它，或者wait它派生的子进程，或者父进程先终止，这时子进程的终止自动由init进程 来接管)。

18) SIGCONT

让一个停止(stopped)的进程继续执行. 本信号不能被阻塞. 可以用一个handler来让程序在由stopped状态变为继续执行时完成特定的工作. 例如, 重新显示提示符

19) SIGSTOP

停止(stopped)进程的执行. 注意它和terminate以及interrupt的区别:该进程还未结束, 只是暂停执行. 本信号不能被阻塞, 处理或忽略.

20) SIGTSTP

停止进程的运行, 但该信号可以被处理和忽略. 用户键入SUSP字符时(通常是Ctrl-Z)发出这个信号

21) SIGTTIN

当后台作业要从用户终端读数据时, 该作业中的所有进程会收到SIGTTIN信号. 缺省时这些进程会停止执行.

22) SIGTTOU

类似于SIGTTIN, 但在写终端(或修改终端模式)时收到.

23) SIGURG

有”紧急”数据或out-of-band数据到达socket时产生.

24) SIGXCPU

超过CPU时间资源限制. 这个限制可以由getrlimit/setrlimit来读取/改变。

25) SIGXFSZ

当进程企图扩大文件以至于超过文件大小资源限制。

26) SIGVTALRM

虚拟时钟信号. 类似于SIGALRM, 但是计算的是该进程占用的CPU时间.

27) SIGPROF

类似于SIGALRM/SIGVTALRM, 但包括该进程用的CPU时间以及系统调用的时间.

28) SIGWINCH

窗口大小改变时发出.

29) SIGIO

文件描述符准备就绪, 可以开始进行输入/输出操作.

30) SIGPWR

Power failure

31) SIGSYS

非法的系统调用。

在以上列出的信号中，程序不可捕获、阻塞或忽略的信号有：SIGKILL, SIGSTOP

不能恢复至默认动作的信号有：SIGILL,SIGTRAP

默认会导致进程流产的信号有：SIGABRT,SIGBUS,SIGFPE,SIGILL,SIGIOT,SIGQUIT,SIGSEGV,SIGTRAP,SIGXCPU,SIGXFSZ

默认会导致进程退出的信号有：SIGALRM,SIGHUP,SIGINT,SIGKILL,SIGPIPE,SIGPOLL,SIGPROF,SIGSYS,SIGTERM,SIGUSR1,SIGUSR2,SIGVTALRM

默认会导致进程停止的信号有：SIGSTOP,SIGTSTP,SIGTTIN,SIGTTOU

默认进程忽略的信号有：SIGCHLD,SIGPWR,SIGURG,SIGWINCH

此外，SIGIO在SVR4是退出，在4.3BSD中是忽略；SIGCONT在进程挂起时是继续，否则是忽略，不能被阻塞





































