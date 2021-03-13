---

---

## tcpdump

[TOC]

- tcp数据抓包格式
	- 源 > 目的：标志
	- 14:51:52.153303 IP localhost.40387 > localhost.italk: Flags [S], seq 2540388307, win 43690, options [mss 65495,sackOK,TS val 1448079 ecr 0,nop,wscale 7], length 0

#### 选项

- -n 用IP代替主机名，用数字端口代替服务名
- -i 指定监听的网卡，-i any 任意网卡
	- tcpdump -n -i lo
	- tcpdump -n -i eth0 host 192.168.31.147 and 114.114.114.114   （截取本机（192.168.31.147）和主机114.114.114.114之间的数据）
- -v 像是稍详细的信息
-  -t 不打印时间信息
-  -c 抓取指定数量的数据报
-  -e 显示以太网帧头部信息
-  -x 十六进制显示
-  -S 以绝对值显示序号，而不是相对值
-  -w 将抓包数据重定向到文件中

**需要抓取正确的网卡才能抓到数据**

#### 过滤
- src 指定数据报发送端
	- tcpdump -n -i lo src port 12345
- dst 指定数据报目的端
	- tcpdump -n -i eth0 dst 192.168.31.147
	- tcpdump -n -i eth0 dst 192.168.31.147  or  192.168.31.157
- 抓取进入端口12345的数据
	- tcpdump  dst port 12345
- 指定目标协议
	- tcpdump tcp
	- tcpdump icmp
- 可以用逻辑操作符 or  和 and  not (&& || ! )配合使用即可筛选出更好的结果
	- 从本机出去的数据包
	- tcpdump -n -i eth0 src 192.168.31.147 or 192.168.31.157 and port ! 22 and tcp


#### 标志

**Flags**

- S SYN
- P PSH
- F FIN
- R RST

**seq** 序号值

**ack**  确认序号

**win** 接收通告窗口大小

**options** tcp选项

- options [mss 65495,sackOK,TS val 1448079 ecr 0,nop,wscale 7]
	- mss 能接收的最大报文长度
	- sackOK 发送端支持并同意使用SACK选项
	- TS val 发送端的时间戳
	- ecr 时间戳回显应答
	- nop 空操作选项
	- wscale 发送端窗口扩大因子
	- length 数据长度

127.0.0.1.40387 > 127.0.0.1.italk: Flags [S], seq 2540388307, win 43690, options [mss 65495,sackOK,TS val 1448079 ecr 0,nop,wscale 7], length 0

127.0.0.1.italk > localhost.40387: Flags [S.], seq 2853158127, ack 2540388308, win 43690, options [mss 65495,sackOK,TS val 1448079 ecr 1448079,nop,wscale 7], length 0

127.0.0.1.40387 > localhost.italk: Flags [.], ack 1, win 342, options [nop,nop,TS val 1448079 ecr 1448079], length 0

### 抓包 UDP

```
tcpdump -i lo0 udp port 9098

tcpdump -i lo0 udp port 9098 -w ./udp.cap
```

客户端发送 “client say hello” 服务端回复 “server say hi”，抓包：

```bash
$tcpdump -i lo0  -v udp port 9098
22:43:55.354641 IP (tos 0x0, ttl 64, id 47017, offset 0, flags [none], proto UDP (17), length 44, bad cksum 0 (->c515)!)
    localhost.53991 > localhost.9098: UDP, length 16
22:43:55.355013 IP (tos 0x0, ttl 64, id 30623, offset 0, flags [none], proto UDP (17), length 41, bad cksum 0 (->523)!)
    localhost.9098 > localhost.53991: UDP, length 13
```



```bash
$tcpdump -i lo0  -vv udp port 9098
22:47:07.052199 IP (tos 0x0, ttl 64, id 57630, offset 0, flags [none], proto UDP (17), length 44, bad cksum 0 (->9ba0)!)
    localhost.51184 > localhost.9098: [bad udp cksum 0xfe2b -> 0x66ca!] UDP, length 16
22:47:07.052478 IP (tos 0x0, ttl 64, id 31386, offset 0, flags [none], proto UDP (17), length 41, bad cksum 0 (->228)!)
    localhost.9098 > localhost.51184: [bad udp cksum 0xfe28 -> 0xbfa3!] UDP, length 13
```











## NetCat（简称nc）

- nc 默认以客户端范式运行
- nc 127.0.0.1 12345 连接服务端
- nc -l 127.0.0.1 12345 监听端口
- 

## ps命令

- -a  显示现行终端机下的所有进程，包括其他用户的进程；
- -x 显示无控制终端的进程，通常与 a 这个参数一起使用，可列出较完整信息。
- -u ：以用户为主的进程状态 
- -r 只显示正在运行地进程
- -j 显示与作业有关的信息：父进程ID、进程组、会话ID、
- -m 显示线程信息

排序
- ps jax --sort=uid,-ppid,+pid
- ps -axj --sort=ppid
	- 根据父进程ID排序