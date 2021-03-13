---
layout: single
title:  "一些linux系统TCP参数配置"
date:   2020-08-15 11:22:26 +0800
permalink: /network/tcp/linux-tcp-sysctl
toc: true
toc_sticky: true
---

### linux 系统TCP参数配置

 `sysctl -a`

**修改**

动态修改如 `sysctl -w net.ipv4.ip_forward=1` 重启失效 `echo 1000 >/proc/sys/net/core/somaxconn`
久保留配置，可以修改/etc/sysctl.conf文件 ， 载入生效 `sysctl -p /etc/sysctl.conf`



### 网络相关
**socket: Too many open files**
文件句柄数限制，查看 `ulimit -a`; 动态修改`ulimit -n 4096`

**tcp读缓存区**

- `net.ipv4.tcp_rmem = 4096	131072	6291456` 最小，默认，最大

**tcp写缓存区**

- `net.ipv4.tcp_wmem = 4096	16384	4194304`

**半连接SYN参数**

- `net.ipv4.tcp_max_syn_backlog = 1024` 通过`netstat -ltn`时Recv-Q查看
- `net.ipv4.tcp_syn_retries = 6`
- `net.ipv4.tcp_synack_retries = 2`
- `net.ipv4.tcp_syncookies = 1` 表示开启SYN Cookies。未开启时当出现SYN等待队列溢出时，直接丢弃SYN包，开启后新SYN也不进入队列，但是会计算出cookie会给客户端，后续根据cookie恢复连接。启用cookies来处理，可防范少量SYN攻击。由于cookie占用正常序列号空间，导致TCP可选功能失效，具体……
- `net.ipv4.tcp_abort_on_overflow = 0` 当 tcp 建立连接的 3 路握手完成后，将连接置入 ESTABLISHED 状态并交付给应用程序的 backlog 队列时，会检查 backlog 队列是否已满。若已满，通常行为是将连接还原至 SYN_ACK 状态，以造成 3 路握手最后的 ACK 包意外丢失假象 —— 这样在客户端等待超时后可重发 ACK —— 以再次尝试进入 ESTABLISHED 状态 —— 作为一种修复/重试机制。如果启用 tcp_abort_on_overflow 则在检查到 backlog 队列已满时，直接发 RST 包给客户端终止此连接

**最大ACCEPT完成握手队列**

- `net.core.somaxconn = 128` 内核已经完成连接建立，但是应用还没accept处理，不时ESTABLISHED连接上限

**快速回收TIME_WAIT**

- `net.ipv4.tcp_tw_reuse = 0` 可选值0, 1, 2

**建立连接时，本地端口可用范围**

- `net.ipv4.ip_local_port_range = 32768	60999`

**TCP快速打开**

- `net.ipv4.tcp_fastopen = 0` 高版本内核默认1开启，它通过握手开始时的SYN包中的TFO cookie（一个TCP选项）来验证一个之前连接过的客户端。如果验证成功，它可以在三次握手最终的ACK包收到之前就开始发送数据，这样便跳过了一个绕路的行为，更在传输开始时就降低了延迟。这个加密的Cookie被存储在客户端，在一开始的连接时被设定好。然后每当客户端连接时，这个Cookie被重复返回

**TCP发送接收缓冲区大小**

- `net.ipv4.tcp_rmem = 4096	87380	629145` tcp 接收缓冲区 ，参考值 `4096	  87380	  16777216`
- `net.ipv4.tcp_wmem = 4096	16384	4194304` tcp 发送缓冲区 ，参考值  `4096	  87380	  16777216`


**其他**

- `net.ipv4.tcp_sack = 1` TCP通信时，如果发送序列中间某个数据包丢失，TCP会通过重传最后确认的包开始的后续包，这样原先已经正确传输的包也可能重复发送，急剧降低了TCP性能。为改善这种情况，发展出SACK(Selective Acknowledgment, 选择性确认)技术，使TCP只重新发送丢失的包，不用发送后续所有的包.
- `net.ipv4.tcp_fack = 0`