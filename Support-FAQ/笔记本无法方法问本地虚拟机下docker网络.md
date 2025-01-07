#### 笔记本使用VMware Workstation Pro搭建docker环境网络问题 

自己主机使用VMware Workstation Pro安装Centos下部署docker；安装Linux使用NET方式，导致docker网络和NET不是同一网段，主机无法直接访问docker网络

- 虚拟机Centos使用NET网络；192.168.194.0/16网段
- 虚拟机Centos IP：192.168.194.10
- docker网段：172.17.0.0/16

```shell
#查看网卡信息
[root@jackycheung ~]# ip a s
1: lo: <LOOPBACK,UP,LOWER_UP> mtu 65536 qdisc noqueue state UNKNOWN group default qlen 1000
    link/loopback 00:00:00:00:00:00 brd 00:00:00:00:00:00
    inet 127.0.0.1/8 scope host lo
       valid_lft forever preferred_lft forever
    inet6 ::1/128 scope host 
       valid_lft forever preferred_lft forever
2: ens33: <BROADCAST,MULTICAST,UP,LOWER_UP> mtu 1500 qdisc pfifo_fast master team1 state UP group default qlen 1000
    link/ether 00:0c:29:50:69:c4 brd ff:ff:ff:ff:ff:ff
3: ens34: <BROADCAST,MULTICAST,UP,LOWER_UP> mtu 1500 qdisc pfifo_fast master team1 state UP group default qlen 1000
    link/ether 00:0c:29:50:69:ce brd ff:ff:ff:ff:ff:ff
4: team1: <BROADCAST,MULTICAST,UP,LOWER_UP> mtu 1500 qdisc noqueue state UP group default qlen 1000
    link/ether 00:0c:29:50:69:ce brd ff:ff:ff:ff:ff:ff
    inet 192.168.194.10/24 brd 192.168.194.255 scope global noprefixroute team1
       valid_lft forever preferred_lft forever
    inet6 fe80::2d3:c7ef:d265:437/64 scope link noprefixroute 
       valid_lft forever preferred_lft forever
5: docker0: <BROADCAST,MULTICAST,UP,LOWER_UP> mtu 1500 qdisc noqueue state UP group default 
    link/ether 02:42:3f:eb:83:1b brd ff:ff:ff:ff:ff:ff
    inet 172.17.0.1/16 brd 172.17.255.255 scope global docker0
       valid_lft forever preferred_lft forever
    inet6 fe80::42:3fff:feeb:831b/64 scope link 
       valid_lft forever preferred_lft forever
15: vethd734ca9@if14: <BROADCAST,MULTICAST,UP,LOWER_UP> mtu 1500 qdisc noqueue master docker0 state UP group default 
    link/ether 4a:e8:6a:3b:a1:81 brd ff:ff:ff:ff:ff:ff link-netnsid 1
    inet6 fe80::48e8:6aff:fe3b:a181/64 scope link 
       valid_lft forever preferred_lft forever
25: veth3fddeed@if24: <BROADCAST,MULTICAST,UP,LOWER_UP> mtu 1500 qdisc noqueue master docker0 state UP group default 
    link/ether 8a:fd:c3:18:11:78 brd ff:ff:ff:ff:ff:ff link-netnsid 0
    inet6 fe80::88fd:c3ff:fe18:1178/64 scope link 
       valid_lft forever preferred_lft forever
27: veth3514552@if26: <BROADCAST,MULTICAST,UP,LOWER_UP> mtu 1500 qdisc noqueue master docker0 state UP group default 
    link/ether b2:07:13:72:76:8d brd ff:ff:ff:ff:ff:ff link-netnsid 2
    inet6 fe80::b007:13ff:fe72:768d/64 scope link 
       valid_lft forever preferred_lft forever
```

```shell
#启用IP转发
[root@jackycheung ~]# vim /etc/sysctl.conf 
[root@jackycheung ~]# echo 'net.ipv4.ip_forward = 1' >> /etc/sysctl.conf 
[root@jackycheung ~]# sysctl -p
net.ipv4.ip_forward = 1
```

```powershell
PS C:\Windows\system32> route ADD 172.17.0.0 MASK 255.255.0.0 192.168.194.10
 操作完成!
PS C:\Windows\system32> ping 172.17.0.1

正在 Ping 172.17.0.1 具有 32 字节的数据:
来自 172.17.0.1 的回复: 字节=32 时间<1ms TTL=64
来自 172.17.0.1 的回复: 字节=32 时间=2ms TTL=64

172.17.0.1 的 Ping 统计信息:
    数据包: 已发送 = 2，已接收 = 2，丢失 = 0 (0% 丢失)，
往返行程的估计时间(以毫秒为单位):
    最短 = 0ms，最长 = 2ms，平均 = 1ms
Control-C
PS C:\Windows\system32> ping 172.17.0.4

正在 Ping 172.17.0.4 具有 32 字节的数据:
来自 172.17.0.4 的回复: 字节=32 时间<1ms TTL=63

172.17.0.4 的 Ping 统计信息:
    数据包: 已发送 = 1，已接收 = 1，丢失 = 0 (0% 丢失)，
往返行程的估计时间(以毫秒为单位):
    最短 = 0ms，最长 = 0ms，平均 = 0ms
PS C:\Windows\system32> curl 172.17.0.4:8080


StatusCode        : 200
StatusDescription :
Content           : hello tomcat!

RawContent        : HTTP/1.1 200
                    Accept-Ranges: bytes
                    Content-Length: 14
                    Content-Type: text/html
                    Date: Tue, 07 Jan 2025 09:23:33 GMT
                    ETag: W/"14-1736238022000"
                    Last-Modified: Tue, 07 Jan 2025 08:20:22 GMT

                    hello...
Forms             : {}
Headers           : {[Accept-Ranges, bytes], [Content-Length, 14], [Content-Type, text/ht
                    ml], [Date, Tue, 07 Jan 2025 09:23:33 GMT]...}
Images            : {}
InputFields       : {}
Links             : {}
ParsedHtml        : mshtml.HTMLDocumentClass
RawContentLength  : 14    
```

