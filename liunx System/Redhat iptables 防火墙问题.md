RHEL 7.0默认使用的是firewall作为防火墙，这里改为iptables防火墙
```
关闭firewall：
1.systemctl stop firewalld.service                #停止firewall
2.systemctl disable firewalld.service             #禁止firewall开机启动
3.yum -y install iptables-services                #安装iptables
4.systemctl  start  iptables.service              #启动防火墙
5.systemctl  stop  iptables.service               #停止防火墙
6.systemctl  restart  iptables.service            #重启防火墙
7.systemctl  status  iptables.service             #查看防火墙状态
8.systemctl  enable  iptables.service             #设置开机启动
```
iptables只是Linux防火墙的管理工具而已，位于/sbin/iptables，由5表5链构成
真正实现防火墙功能的是netfilter，它是Linux内核中实现包过滤的内部结构

1.iptables传输数据包的过程
```
* 当一个数据包进入网卡时，它首先进入PREROUTING链，内核根据数据包目的IP判断是否需要转送出去
* 如果数据包就是进入本机的，它就会沿着图向下移动，到达INPUT链。数据包到了INPUT链后，任何进程都会收到它
* 本机上运行的程序可以发送数据包，这些数据包会经过OUTPUT链，然后到达POSTROUTING链输出
* 如果数据包是要转发出去的，且内核允许转发，数据包就会如图所示向右移动，经过FORWARD链，然后到达POSTROUTING链输出
```
![image](https://github.com/dwjlw1314/DWJ-PROJECT/raw/master/PictureSource/1.4.1.png)

2.iptables的规则表和链
```
表（tables）提供特定功能，内置了4个表，即filter表、nat表、mangle表和raw表，分别用于实现包过滤，网络地址转换、包重构(修改)和数据跟踪处理
链（chains）是数据包传播的路径，每一条链其实就是众多规则中的一个检查清单，每一条链中可以有一条或数条规则。当一个数据包到达一个链时，iptables就会从链中第一条规则开始检查，看该数据包是否满足规则所定义的条件。如果满足，系统就会根据该条规则所定义的方法处理该数据包；否则iptables将继续检查下一条规则，如果该数据包不符合链中任一条规则，iptables就会根据该链预先定 义的默认策略来处理数据包
```
![image](https://github.com/dwjlw1314/DWJ-PROJECT/raw/master/PictureSource/1.4.2.png)

规则表：

1.filter表——三个链：INPUT、FORWARD、OUTPUT

作用：过滤数据包；内核模块：iptables_filter

2.Nat表——三个链：PREROUTING、POSTROUTING、OUTPUT

作用：用于网络地址转换（IP、端口）；内核模块：iptable_nat

3.Mangle表——五个链：PREROUTING、POSTROUTING、INPUT、OUTPUT、FORWARD

作用：修改数据包的服务类型、TTL、并且可以配置路由实现QOS；内核模块：iptable_mangle

4.Raw表——两个链：OUTPUT、PREROUTING

作用：决定数据包是否被状态跟踪机制处理；内核模块：iptable_raw

5.security表——三个链：INPUT、FORWARD、OUTPUT

作用：主要用于强制网络访问的网络，如SElinux

规则链：
```
1.INPUT-------------------进来的数据包应用此规则链中的策略
2.OUTPUT ---------- ------外出的数据包应用此规则链中的策略
3.FORWARD ----------------转发数据包时应用此规则链中的策略
4.PREROUTING -------------对数据包作路由选择前应用此链中的规则（记住！所有的数据包进来的时侯都先由这个链处理
5.POSTROUTING ------------对数据包作路由选择后应用此链中的规则（所有的数据包出来的时侯都先由这个链处理）
```

3.规则表之间的优先顺序

raw—mangle—nat—filter

4.规则链之间的优先顺序(分三种)
```
第一种情况：入站数据流向
    从外界到达防火墙的数据包，先被PREROUTING规则链处理（是否修改数据包地址等），之后会进行路由选择（判断该数据包应该发往何处），如果数据包 的目标主机是防火墙本机（比如说Internet用户访问防火墙主机中的web服务器的数据包），那么内核将其传给INPUT链进行处理（决定是否允许通过等），通过以后再交给系统上层的应用程序（比如Apache服务器）进行响应
第二冲情况：转发数据流向
    来自外界的数据包到达防火墙后，首先被PREROUTING规则链处理，之后会进行路由选择，如果数据包的目标地址是其它外部地址（比如局域网用户通过网 关访问QQ站点的数据包），则内核将其传递给FORWARD链进行处理（是否转发或拦截），然后再交给POSTROUTING规则链（是否修改数据包的地 址等）进行处理
第三种情况：出站数据流向
    防火墙本机向外部地址发送的数据包（比如在防火墙主机中测试公网DNS服务器时），首先被OUTPUT规则链处理，之后进行路由选择，然后传递给POSTROUTING规则链（是否修改数据包的地址等）进行处理
```    

5.管理和设置iptables规则

![image](https://github.com/dwjlw1314/DWJ-PROJECT/raw/master/PictureSource/1.4.3.jpg)

![image](https://github.com/dwjlw1314/DWJ-PROJECT/raw/master/PictureSource/1.4.4.jpg)

6.iptables的基本语法格式

iptables [-t 表名] 命令选项 ［链名］ ［条件匹配］ ［-j 目标动作或跳转］

说明：表名、链名用于指定 iptables命令所操作的表和链，命令选项用于指定管理iptables规则的方式（比如：插入、增加、删除、查看等；条件匹配用于指定对符合什么样 条件的数据包进行处理；目标动作或跳转用于指定数据包的处理方式（比如允许通过、拒绝、丢弃、跳转（Jump）给其它链处理

7.iptables命令的管理控制选项
```
-A 在指定链的末尾添加（append）一条新的规则
-D  删除（delete）指定链中的某一条规则，可以按规则序号和内容删除
-I  在指定链中插入（insert）一条新的规则，默认在第一行添加
-R  修改、替换（replace）指定链中的某一条规则，可以按规则序号和内容替换
-L  列出（list）指定链中所有的规则进行查看
-E  重命名用户定义的链，不改变链本身
-F  清空（flush）
-N  新建（new-chain）一条用户自己定义的规则链
-X  删除指定表中用户自定义的规则链（delete-chain）
-P  设置指定链的默认策略（policy）
-Z 将所有表的所有链的字节和数据包计数器清零
-n  使用数字形式（numeric）显示输出结果
-v  查看规则表详细信息（verbose）的信息
-V  查看版本(version)
-h  获取帮助（help）
```
8.防火墙处理数据包的四种方式
```
ACCEPT 允许数据包通过
DROP 直接丢弃数据包，不给任何回应信息
REJECT 拒绝数据包通过，必要时会给数据发送端一个响应的信息
LOG在/var/log/messages文件中记录日志信息，然后将数据包传递给下一条规则
```
9.iptables防火墙规则的保存与恢复 (redhat系统有效)

iptables-save命令把规则保存到文件中，再由目录rc.d下的脚本（/etc/rc.d/init.d/iptables）自动装载

>[root@dwj ~]# iptables-save > /etc/sysconfig/iptables       #生成保存规则的文件 /etc/sysconfig/iptables

>[root@dwj ~]# service iptables save                         #它能把规则自动保存在/etc/sysconfig/iptables中

当计算机启动时，rc.d下的脚本将用命令iptables-restore调用这个文件，从而就自动恢复了规则

>[root@dwj ~]# iptables-restore /etc/sysconfig/iptables      #手动恢复iptables规则

>[root@dwj ~]# iptables -S                                   #查看当前系统iptables规则设置

>[root@dwj ~]# iptables -F                                   #清空iptables的设置规则

>[root@dwj ~]# iptables -Z                                   #清空watch iptables -nvL方式的数据包信息

10.基本范例

删除INPUT链的第一条规则
>[root@dwj ~]# iptables -D INPUT 1

11.iptables防火墙常用的策略

* 拒绝进入防火墙的所有ICMP协议数据包
>[root@dwj ~]# iptables -I INPUT -p icmp -j REJECT

* 允许防火墙转发除ICMP协议以外的所有数据包
>[root@dwj ~]# iptables -A FORWARD -p ! icmp -j ACCEPT         #使用“！”可以将条件取反

* 拒绝转发来自192.168.1.10主机的数据，允许转发来自192.168.0.0/24网段的数据

>[root@dwj ~]# iptables -A FORWARD -s 192.168.1.11 -j REJECT

>[root@dwj ~]# iptables -A FORWARD -s 192.168.0.0/24 -j ACCEPT    

注意要把拒绝的放在前面不然就不起作用了

* 丢弃从外网接口（eth1）进入防火墙本机的源地址为私网地址的数据包

>[root@dwj ~]# iptables -A INPUT -i eth1 -s 192.168.0.0/16 -j DROP

>[root@dwj ~]# iptables -A INPUT -i eth1 -s 172.16.0.0/12 -j DROP

>[root@dwj ~]# iptables -A INPUT -i eth1 -s 10.0.0.0/8 -j DROP

* 封堵网段（192.168.1.0/24），两小时后解封

>[root@dwj ~]# iptables -I INPUT -s 10.20.30.0/24 -j DROP

>[root@dwj ~]# iptables -I FORWARD -s 10.20.30.0/24 -j DROP

>[root@dwj ~]# at now 2 hours at> iptables -D INPUT 1 at> iptables -D FORWARD 1

* 只允许管理员从202.13.0.0/16网段使用SSH远程登录防火墙主机

>[root@dwj ~]# iptables -A INPUT -p tcp --dport 22 -s 202.13.0.0/16 -j ACCEPT

>[root@dwj ~]# iptables -A INPUT -p tcp --dport 22 -j DROP

这个用法比较适合对设备进行远程管理时使用，比如位于分公司中的SQL服务器需要被总公司管理时

* 允许本机开放从TCP端口20-1024提供的应用服务

>[root@dwj ~]# iptables -A INPUT -p tcp --dport 20:1024 -j ACCEPT

>[root@dwj ~]# iptables -A OUTPUT -p tcp --sport 20:1024 -j ACCEPT

* 允许转发来自192.168.0.0/24局域网段的DNS解析请求数据包

>[root@dwj ~]# iptables -A FORWARD -s 192.168.0.0/24 -p udp --dport 53 -j ACCEPT

>[root@dwj ~]# iptables -A FORWARD -d 192.168.0.0/24 -p udp --sport 53 -j ACCEPT

* 禁止其他主机ping防火墙主机，但是允许从防火墙上ping其他主机

>[root@dwj ~]# iptables -I INPUT -p icmp --icmp-type Echo-Request -j DROP

>[root@dwj ~]# iptables -I INPUT -p icmp --icmp-type Echo-Reply -j ACCEPT

>[root@dwj ~]# iptables -I INPUT -p icmp --icmp-type destination-Unreachable -j ACCEPT

* 禁止转发来自MAC地址为00：0C：29：27：55：3F的和主机的数据包

>[root@dwj ~]# iptables -A FORWARD -m mac --mac-source 00:0c:29:27:55:3F -j DROP <br>
iptables中使用“-m 模块关键字”的形式调用显示匹配。咱们这里用“-m mac –mac-source”来表示数据包的源MAC地址

* 允许防火墙本机对外开放TCP端口20、21、25、110以及被动模式FTP端口1250-1280

>[root@dwj ~]# iptables -A INPUT -p tcp -m multiport --dport 20,21,25,110,1250:1280 -j ACCEPT <br>
这里用“-m multiport –dport”来指定目的端口及范围

* 禁止转发源IP地址为192.168.1.20-192.168.1.99的TCP数据包

>[root@dwj ~]# iptables -A FORWARD -p tcp -m iprange --src-range 192.168.1.20-192.168.1.99 -j DROP <br>
此处用“-m –iprange –src-range”指定IP范围

* 禁止转发与正常TCP连接无关的非—syn请求数据包

>[root@dwj ~]# iptables -A FORWARD -m state --state NEW -p tcp ! --syn -j DROP <br>
"-m state" 表示数据包的连接状态，"NEW" 表示与任何连接无关的

* 拒绝访问防火墙的新数据包，但允许响应连接或与已有连接相关的数据包

>[root@dwj ~]# iptables -A INPUT -p tcp -m state --state NEW -j DROP <br>
>[root@dwj ~]# iptables -A INPUT -p tcp -m state --state ESTABLISHED,RELATED -j ACCEPT <br>
"ESTABLISHED" 表示已经响应请求或者已经建立连接的数据包，"RELATED" 表示与已建立的连接有相关性的，比如FTP数据连接等

* 只开放本机的web服务（80）、FTP(20、21、20450-20480)，放行外部主机发住服务器其它端口的应答数据包，将其他入站数据包均予以丢弃处理。

>[root@dwj ~]# iptables -I INPUT -p tcp -m multiport --dport 20,21,80 -j ACCEPT

>[root@dwj ~]# iptables -I INPUT -p tcp --dport 20450:20480 -j ACCEPT

>[root@dwj ~]# iptables -I INPUT -p tcp -m state --state ESTABLISHED -j ACCEPT

>[root@dwj ~]# iptables -P INPUT DROP
