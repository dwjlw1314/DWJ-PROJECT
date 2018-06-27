netstat 命令用于显示各种网络相关信息，如网络连接，路由表，接口状态等等，从整体上看，netstat的输出结果可以分为两个部分：

一个是Active Internet connections，称为有源TCP连接，其中"Recv-Q"和"Send-Q"的是接收队列和发送队列。这些数字一般都应该是0。如果不是则表示软件包正在队列中堆积

一个是Active UNIX domain sockets，称为有源Unix域套接口(和网络套接字一样，但是只能用于本机通信，性能可以提高一倍)。Proto显示连接使用的协议,RefCnt表示连接到本套接口上的进程号,Types显示套接口的类型,State显示套接口当前的状态,Path表示连接到套接口的其它进程使用的路径名

常见参数: 提示：LISTEN和LISTENING的状态只有用-a或者-l才能看到

参数名 | 描述
---|---
-a (all)| 显示所有选项，默认不显示LISTEN相关
-t (tcp)| 仅显示tcp相关选项
-u (udp)| 仅显示udp相关选项
-n | 拒绝显示别名，能显示数字的全部转化成数字。
-l | 仅列出有在 Listen (监听) 的服务状态
-p | 显示建立相关链接的程序名
-r | 显示路由信息，路由表
-e | 显示扩展信息，例如uid等
-s | 按各个协议进行统计
-c | 每隔一个固定时间，执行该netstat命令。

```
1. [root@dwj Desktop]# netstat -a   #列出所有端口 (包括监听和未监听的)  
2. [root@dwj Desktop]# netstat -at  #列出所有tcp 端口  
3. [root@dwj Desktop]# netstat -au  #列出所有udp端口   
4. [root@dwj Desktop]# netstat -l   #只显示监听端口  
5. [root@dwj Desktop]# netstat -lt  #只列出所有监听tcp端口  
6. [root@dwj Desktop]# netstat -lu  #只列出所有监听udp端口  
7. [root@dwj Desktop]# netstat -lx  #只列出所有监听UNIX端口  
8. [root@dwj Desktop]# netstat -s   #显示所有端口的统计信息  
9. [root@dwj Desktop]# netstat -stu #显示TCP和UDP端口的统计信息  
10.[root@dwj Desktop]# netstat -p   #输出中显示PID和进程名称  
11.[root@dwj Desktop]# netstat -n   #在netstat输出中不显示主机，端口和用户名;
12.[root@dwj Desktop]# netstat -an  #当你想让主机，端口和用户名使用数字代替那些名称
如果只是不想让这三个名称中的一个被显示，使用以下命令
[root@dwj Desktop]# netsat -a --numeric-ports
[root@dwj Desktop]# netsat -a --numeric-hosts
[root@dwj Desktop]# netsat -a --numeric-users
13.[root@dwj Desktop]# netstat --verbose #显示系统不支持的地址族，在输出的末尾，会有如下的信息
netstat: no support for `AF IPX' on this system.
netstat: no support for `AF AX25' on this system.
netstat: no support for `AF X25' on this system.
netstat: no support for `AF NETROM' on this system.
14.[root@dwj Desktop]# netstat -r   #显示核心路由信息  
15.[root@dwj Desktop]# netstat -i   #显示网络接口列表  
16.[root@dwj Desktop]# netstat -ie  #显示详细信息
```
#查看连接某服务端口最多的的IP地址
>[root@dwj dwj]# netstat -nat | grep "192.168.1.15:22" |awk '{print $5}'|awk -F: '{print $1}'|sort|uniq -c|sort -nr|head -20
