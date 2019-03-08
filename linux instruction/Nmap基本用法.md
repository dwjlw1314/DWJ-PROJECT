nmap 又叫做Network Mapper(网络映射器)用于浏览网络，执行安全扫描，网络审计以及在远程机器找到开放端口。它可以扫描在线主机，操作系统，滤包器和远程主机打开的端口

Linux安装包安装nmap
```
[root@dwj Desktop]# yum install nmap           [基于RedHat系统]
[root@dwj Desktop]$ sudo apt-get install nmap  [基于Debian系统]
```
Linux源码安装nmap
```
[root@dwj opt]# wget https://nmap.org/dist/nmap-7.70.tar.bz2
[root@dwj opt]# tar -jxvf nmap-7.70.tar.bz2
[root@dwj opt]# cd nmap-7.70
[root@dwj nmap-7.70]# ./configure
[root@dwj nmap-7.70]# make && make install
```
1.使用Hostname和IP地址来扫描系统
>[root@dwj nmap-7.70]# nmap www.baidu.com

```
Starting Nmap 5.51 ( http://nmap.org ) at 2018-06-06 17:26 CST
Warning: File ./nmap-services exists, but Nmap is using /usr/share/nmap/nmap-services for security and consistency reasons.  set NMAPDIR=. to give priority to files in your local directory (may affect the other data files too).
Nmap scan report for www.baidu.com (119.75.217.109)
Host is up (0.010s latency).
Not shown: 998 filtered ports
PORT    STATE SERVICE
80/tcp  open  http
443/tcp open  https

Nmap done: 1 IP address (1 host up) scanned in 4.99 seconds
```
>[root@dwj nmap-7.70]# nmap 192.168.8.45

```
Starting Nmap 5.51 ( http://nmap.org ) at 2018-06-06 17:28 CST
Warning: File ./nmap-services exists, but Nmap is using /usr/share/nmap/nmap-services for security and consistency reasons.  set NMAPDIR=. to give priority to files in your local directory (may affect the other data files too).
Nmap scan report for localhost (192.168.8.45)
Host is up (0.000042s latency).
Not shown: 995 filtered ports
PORT    STATE  SERVICE
21/tcp  closed ftp
22/tcp  open   ssh
23/tcp  open   telnet
80/tcp  closed http
443/tcp closed https

Nmap done: 1 IP address (1 host up) scanned in 4.64 seconds
```
>[root@dwj nmap-7.70]# nmap -v www.baidu.com

```
Starting Nmap 5.51 ( http://nmap.org ) at 2018-06-06 17:29 CST
Warning: File ./nmap-services exists, but Nmap is using /usr/share/nmap/nmap-services for security and consistency reasons.  set NMAPDIR=. to give priority to files in your local directory (may affect the other data files too).
Initiating Ping Scan at 17:29
Scanning www.baidu.com (119.75.217.109) [4 ports]
Completed Ping Scan at 17:29, 0.01s elapsed (1 total hosts)
Initiating SYN Stealth Scan at 17:29
Scanning www.baidu.com (119.75.217.109) [1000 ports]
Discovered open port 80/tcp on 119.75.217.109
Discovered open port 443/tcp on 119.75.217.109
SYN Stealth Scan Timing: About 39.60% done; ETC: 17:30 (0:00:47 remaining)
Completed SYN Stealth Scan at 17:29, 43.62s elapsed (1000 total ports)
Nmap scan report for www.baidu.com (119.75.217.109)
Host is up (0.058s latency).
Not shown: 998 filtered ports
PORT    STATE SERVICE
80/tcp  open  http
443/tcp open  https

Read data files from: /usr/share/nmap
Nmap done: 1 IP address (1 host up) scanned in 43.68 seconds
           Raw packets sent: 3026 (133.120KB) | Rcvd: 30 (1.304KB)
```
2.扫描多个主机只需要简单地以空格隔开输入他们IP地址或者主机名即可
>[root@dwj nmap-7.70]# nmap 192.168.8.45,192.168.8.54

3.通过使用通配符，你可以扫描整个子网或者IP段
>[root@dwj nmap-7.70]# nmap 192.168.8.*

```
Starting Nmap 5.51 ( http://nmap.org ) at 2018-06-06 17:33 CST
Warning: File ./nmap-services exists, but Nmap is using /usr/share/nmap/nmap-services for security and consistency reasons.  set NMAPDIR=. to give priority to files in your local directory (may affect the other data files too).
Nmap scan report for localhost (192.168.8.1)
Host is up (0.017s latency).
Not shown: 989 closed ports
PORT      STATE SERVICE
53/tcp    open  domain
135/tcp   open  msrpc
139/tcp   open  netbios-ssn
445/tcp   open  microsoft-ds
3300/tcp  open  unknown
3389/tcp  open  ms-term-serv
49152/tcp open  unknown
49153/tcp open  unknown
49154/tcp open  unknown
49155/tcp open  unknown
49157/tcp open  unknown
MAC Address: 00:14:5E:DD:6C:39 (IBM)

Nmap scan report for 192.168.8.10
Host is up (0.041s latency).
Not shown: 996 filtered ports
PORT     STATE SERVICE
999/tcp  open  garcon
1688/tcp open  nsjtp-data
8081/tcp open  blackice-icecap
9000/tcp open  cslistener
MAC Address: 60:D8:19:D3:4D:4D (Unknown)

Nmap scan report for 192.168.8.11
Host is up (0.056s latency).
All 1000 scanned ports on 192.168.8.11 are filtered
MAC Address: F8:59:71:C6:A8:54 (Unknown)

Nmap scan report for localhost (192.168.8.17)
Host is up (0.030s latency).
Not shown: 993 filtered ports
PORT     STATE SERVICE
135/tcp  open  msrpc
139/tcp  open  netbios-ssn
443/tcp  open  https
445/tcp  open  microsoft-ds
902/tcp  open  iss-realsecure
912/tcp  open  apex-mesh
5357/tcp open  wsdapi
MAC Address: E8:B1:FC:A3:E4:A4 (Unknown)
...
```
在上面的输出你可以看见nmap扫描整个子网并且提供了那些主机在这个网络是上线状态的信息

4.可以通过简单的使用IP地址的最后8字节，执行扫描多个IP地址
>[root@dwj nmap-7.70]# nmap 192.168.8.54,55,56

5.扫描来自文件的主机列表,创建一个文本文件叫“nmaptest.txt”并且规定所有需要做扫描的IP地址和服务器的主机名
>[root@dwj nmap-7.70]# cat > nmaptest.txt << end

    > www.baidu.com
    > 192.168.8.45
    > 172.1.1.1
    > end

使用“iL”选项的nmap命令去扫描所有在文件列出的IP地址[root@dwj nmap-7.70]# nmap -iL nmaptest.txt
>[root@dwj nmap-7.70]# nmap -iL nmaptest.txt

6.扫描IP段 你可以用Nmap执行扫描指定的IP段
>[root@dwj nmap-7.70]# nmap 192.168.8.45-50

7.使用Nmap的通配符扫描整个网络的时候想要排除某几个IP地址
>[root@dwj nmap-7.70]# nmap 192.168.8.* --exclude 192.168.8.40

8.扫描系统信息和路由追踪,使用“-A”选项
>[root@dwj nmap-7.70]# nmap -A 192.168.8.45

```
Warning: File ./nmap-os-db exists, but Nmap is using /usr/share/nmap/nmap-os-db for security and consistency reasons.  set NMAPDIR=. to give priority to files in your local directory (may affect the other data files too).

Starting Nmap 5.51 ( http://nmap.org ) at 2018-06-06 17:49 CST
Nmap scan report for localhost (192.168.8.45)
Host is up (0.00010s latency).
Not shown: 995 filtered ports
PORT    STATE  SERVICE VERSION
21/tcp  closed ftp
22/tcp  open   ssh     OpenSSH 5.3 (protocol 2.0)
| ssh-hostkey: 1024 63:81:66:33:b3:6e:78:23:4d:8e:9a:dd:19:c3:6f:c6 (DSA)
|\_2048 c7:38:85:ce:de:43:33:25:11:e1:34:1d:0b:12:eb:7d (RSA)
23/tcp  open   telnet  Linux telnetd
80/tcp  closed http
443/tcp closed https
Device type: general purpose|WAP|specialized|firewall|webcam|PBX
Running (JUST GUESSING): Linux 2.6.X|2.4.X (95%), Netgear embedded (94%), Crestron 2-Series (91%), Check Point embedded (88%), AXIS Linux 2.6.X (88%), Vodavi embedded (88%), Linksys embedded (85%)
Aggressive OS guesses: Linux 2.6.19 - 2.6.36 (95%), Netgear DG834G WAP (94%), Linux 2.6.31 - 2.6.35 (91%), Crestron XPanel control system (91%), Linux 2.6.15 - 2.6.31 (91%), Linux 2.6.22 (Ubuntu 7.10, x86_64) (91%), Linux 2.6.28 (91%), Linux 2.6.22 (90%), Linux 2.6.18 - 2.6.25 (90%), Linux 2.6.20 - 2.6.25 (90%)
No exact OS matches for host (test conditions non-ideal).
Network Distance: 0 hops
Service Info: OS: Linux

OS and Service detection performed. Please report any incorrect results at http://nmap.org/submit/ .
Nmap done: 1 IP address (1 host up) scanned in 9.17 seconds
```
在上面的输出，你可以看见NMAP提供了远程主机正在运行的操作系统的TCP/IP指纹信息、更多的端口细节信息和运行在远程主机的服务

9.使用Nmap启动操作系统检测，使用“-O”选项和“-osscan-guess”都可以帮助发现操作系统
>[root@dwj nmap-7.70]# nmap -O 192.168.8.45

10.扫描主机来检测防火墙
>[root@dwj nmap-7.70]# nmap -sA 192.168.8.45

11.扫描主机是否受到任何的滤包器和防火墙的保护
>[root@dwj nmap-7.70]# nmap -PN 192.168.8.45

12.找出网络中在线的主机,“-sP”选项;有这个选项支持的nmap跳过端口探测和其他检测
>[root@dwj nmap-7.70]# nmap -sP 192.168.8.*

```
Nmap scan report for 192.168.8.253
Host is up (0.019s latency).
MAC Address: 9C:06:1B:8C:72:20 (Unknown)
Nmap scan report for localhost (192.168.8.254)
Host is up (0.031s latency).
MAC Address: 0C:DA:41:18:6F:BC (Unknown)
Nmap done: 256 IP addresses (88 hosts up) scanned in 6.43 seconds
```
13.执行快速扫描,使用“-F”选项去扫描nmap-services文件列出的端口，但不会扫描其他的端口
>[root@dwj nmap-7.70]# nmap -F 192.168.8.45   <br>
  Not shown: 95 filtered ports

14.连续地扫描端口,使用“-r”标记替代随机扫描
>[root@dwj nmap-7.70]# ./nmap -r 192.168.8.45   <br>
  Not shown: 995 filtered ports

14.打印主机接口和路由
>[root@dwj nmap-7.70]# ./nmap --iflist

```
Starting Nmap 5.51 ( http://nmap.org ) at 2018-06-07 10:44 CST
************************INTERFACES************************
DEV    (SHORT)  IP/MASK          TYPE     UP MTU   MAC
lo     (lo)     127.0.0.1/8      loopback up 16436
eth0   (eth0)   192.168.8.45/24  ethernet up 1500  00:0C:29:95:06:5B
eth1   (eth1)   192.168.8.123/24 ethernet up 1500  00:0C:29:95:06:65
virbr0 (virbr0) 192.168.122.1/24 ethernet up 1500  52:54:00:B6:67:15

**************************ROUTES**************************
DST/MASK         DEV    GATEWAY
192.168.122.0/24 virbr0
192.168.8.0/24   eth1
192.168.8.0/24   eth0
0.0.0.0/0        eth0   192.168.8.254
```
在上面的输出列出了你的系统的接口和他们各自的路由

15.扫描特定的端口,使用“-p”选项，默认情况下Nmap扫描只扫描TCP端口
>[root@dwj nmap-7.70]# nmap -p 80,111 192.168.8.45

16.扫描一段的端口
>[root@dwj nmap-7.70]# nmap -p 80-111 192.168.8.45

17.指定特别的端口类型和标号来扫描,扫描一个TCP端口
>[root@dwj nmap-7.70]# nmap -p T:8888,80 192.168.8.45

18.指定特别的端口类型和标号来扫描,扫描一个UDP端口
>[root@dwj nmap-7.70]# nmap -sU 53 192.168.8.45

19.使用“-sV”选项，我们可以查询出远程服务器的服务版本
>[root@dwj nmap-7.70]# nmap -sV 192.168.8.45

20.滤包器防火墙阻止ICMP的ping请求，这种情况下，我们可以使用TCP ACK和TCP Syn方法来扫描远程主机
>[root@dwj nmap-7.70]# nmap -PS 192.168.8.45

21.用TCP ACK扫描远程主机扫描特定端口
>[root@dwj nmap-7.70]# nmap -PA -p 22,80 192.168.8.45

22.用TCP Syn扫描远程主机扫描特定端口
>[root@dwj nmap-7.70]# nmap -PS -p 22,80 192.168.8.45

23.执行一个秘密的扫描
>[root@dwj nmap-7.70]# nmap -sS 192.168.8.45

24.执行一个TCP空扫描来欺骗防火墙
>[root@dwj nmap-7.70]# nmap -sN 192.168.8.45
