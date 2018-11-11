NTP是网络时间协议(Network Time Protocol)，它是用来同步网络中各个计算机的时间的协议

```
ntpd与ntpdate在更新时间时有什么区别? ntpd不仅仅是时间同步服务器，它还可以做客户端与标准时间服务器进行同步时间，而且是平滑同步，并非ntpdate立即同步，在生产环境中慎用ntpdate，也正如此两者不可同时运行

时钟的跃变，对于某些程序会导致很严重的问题。许多应用程序依赖连续的时钟——毕竟，这是一项常见的假定，即，取得的时间是线性的，一些操作，例如数据库事务，通常会地依赖这样的事实：时间不会往回跳跃。不幸的是，ntpdate调整时间的方式就是我们所说的”跃变“：在获得一个时间之后，ntpdate使用settimeofday(2)设置系统时间，这有几个非常明显的问题：

第一，这样做不安全。ntpdate的设置依赖于ntp服务器的安全性，攻击者可以利用一些软件设计上的缺陷，拿下ntp服务器并令与其同步的服务器执行某些消耗性的任务。由于ntpdate采用的方式是跳变，跟随它的服务器无法知道是否发生了异常（时间不一样的时候，唯一的办法是以服务器为准）

第二，这样做不精确。一旦ntp服务器宕机，跟随它的服务器也就会无法同步时间。与此不同，ntpd不仅能够校准计算机的时间，而且能够校准计算机的时钟

第三，这样做不够优雅。由于是跳变，而不是使时间变快或变慢，依赖时序的程序会出错（例如，如果ntpdate发现你的时间快了，则可能会经历两个相同的时刻，对某些应用而言，这是致命的）。因而，唯一一个可以令时间发生跳变的点，是计算机刚刚启动，但还没有启动很多服务的那个时候。其余的时候，理想的做法是使用ntpd来校准时钟，而不是调整计算机时钟上的时间

NTPD 在和时间服务器的同步过程中，会把 BIOS 计时器的振荡频率偏差——或者说 Local Clock 的自然漂移(drift)记录下来。这样即使网络有问题，本机仍然能维持一个相当精确的走时
```
1.检查是否安装了NTP的rpm包，本次已经安装
```
[root@dwj ~]# rpm -qa|grep ntp
ntpdate-4.2.4p8-3.e16.x86_64
fontpackages-filesystem-1.41-1.1.e16.noarch
ntp-4.2.4p8-3.e16.x86_64
```
2.在NTP的官方网站http://www.pool.ntp.org查找最近的2个NTP Server, 格式都是：number.country.pool.ntp.org
![image](https://github.com/dwjlw1314/DWJ-PROJECT/raw/master/PictureSource/4.24.1.png)

3.打开NTP服务器之前先和这些服务器做一个同步，用ntpdate命令手动更新时间
>[root@dwj ~]# ntpdate 202.120.2.101       #ip可以通过域名解析获取

那么为什么在打开NTP服务之前先要手动运行同步呢?
>因为根据NTP的设置,如果你的系统时间比正确时间要快的话那么NTP是不会帮你调整的,所以要么你把时间设置回去,要么先做一个手动同步,当你的时间设置和NTP服务器的时间相差很大的时候,NTP会花上较长一段时间进行调整.所以手动同步可以减少这段时间

4.修改配置文件/etc/ntp.conf
>[root@dwj ~]# vim /etc/ntp.conf

```
系统时间与BIOS事件的偏差记录,driftfile字段后指定文件会被ntpd自动更新，所以权限是ntpd才行，ntpd的owner是ntp
driftfile /opt/drift

确保localhost(这个常用的IP地址用来指Linux服务器本身)有足够权限.使用没有任何限制关键词的语法：
restrict 127.0.0.1
restrict -6 ::1

如果集群是在一个封闭的局域网内，可以屏蔽掉默认的server
#server 0.rhel.pool.ntp.org
#server 1.rhel.pool.ntp.org
#server 2.rhel.pool.ntp.org

如果集群是在广域网中，可以设置指定时间服务器的ip或者域名
server 202.120.2.101 prefer    #prefer是优先级
server 120.25.108.11
server south-america.pool.ntp.org iburst

如果将本地的硬件时间也作为同步的时间源之一，不能联网的时候可以把本机时间作为同步时间源，在内网可以删除掉其他的server
server 127.127.1.0  #local clock
设置本地时钟源的层次为10，如果NTPD服务是从本地时钟源获取时间的话，NTPD对外宣布的时间层次为11
fudge  127.127.1.0  stratum 10   #0是顶级的，只能提供同步服务

配置客户端的授权,给指定的机器设置访问NTP Server的权限，这是通过restrict配置项实现的(下面是两条不同设置方式)
restrict 192.168.0.100 mask 255.255.255.0 nomodify notrap noquery
restrict -6 default kod nomodify notrap nopeer noquery
```
语法: restrict IP地址 mask 子网掩码 参数 (default就是指所有的IP)

参数名 | 含义
---|---
ignore   | 关闭所有的 NTP 联机服务
nomodify | 客户端不能更改服务端的时间参数，但是客户端可以通过服务端进行网络校时
notrust  | 拒绝没有认证的客户端
noquery  | 客户端不能使用ntpq、ntpc等指令来查询服务器时间，等于不提供ntp的网络校时
notrap   | 不提供trap这个远程时间登录的功能
nopeer   | 不与其他同一层的ntp服务器进行时间同步
kod      | 访问违规时发送 KoD 包
-6       | 表示IPV6地址的权限设置

通过一个例子来解释以下语句：restrict 10.221.18.112 mask 255.255.255.240 nomodify notrap
```
该含义是授权10.221.18.112网段上的所有机器可以从这台机器上查询和同步时间，对于第一个参数[address] 它可能是一个IP，也可能是一个网段，这取决于后面给出的子网掩码。如果这里的子网掩码是255.255.255.255，那么配置就变成了只授权给IP是10.221.18.112的那一台机器连接。但是这里子网掩码是255.255.255.240，则此时的10.221.18.112就是一个网络标识了，成为指定网段了
```

ntp服务默认只会同步系统时间。如果想要让ntp同时同步硬件时间，可以设置/etc/sysconfig/ntpd文件,添加以下内容
>[root@dwj ~]# vim /etc/sysconfig/ntpd

```
OPTIONS="-x -u ntp:ntp -p /var/run/ntpd.pid"
SYNC_HWCLOCK=yes
```
允许BIOS与系统时间同步
>[root@dwj Desktop]# hwclock -w

5.启动NTP Server,并且设置其在开机后自动运行
>[root@dwj ~]# /etc/init.d/ntpd start  <br>
>[root@dwj ~]# chkconfig --level 35 ntpd on

6.查看NTP服务的运行状况
>[root@dwj ~]# watch ntpq -p

7.查看ntp服务器有无和上层ntp连通
>[root@dwj ~]# ntpstat

8.查看与时间同步服务器的时间偏差
>[root@dwj ~]# ntpdc -c loopinfo
