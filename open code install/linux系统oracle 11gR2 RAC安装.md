前提准备 linux.x64_11gR2_database.tar.gz 和 linux.x64_11gR2_grid.zip 软件包

RAC是实时应用集群(real application clusters)，是Oracle高可用性的一种，也是Oracle数据库支持网格计算环境的核心技术，
对于RAC来说，只有一份数据文件，都是相同的存储上面放着oracle的文件，但是是由多个实例共用同一份数据文件,RAC至少有两套
物理上不同的网络，一个是实例之间的数据的传递，另外一个是公有网络，是对外提供服务的,RAC提供了在实例级别的冗余，同时RAC不能够解决数据的安全，尽管有多个实例，但是只要数据文件损坏了，那么整个数据库就损坏了

RAC中的特点是:
```
每一个节点的instance都有自己的SGA
每一个节点的instance都有自己的background process
每一个节点的instance都有自己的redo logs
每一个节点的instance都有自己的undo表空间
所有节点都共享一份datafiles和controlfiles
还有一个缓存融合的技术(Cache fusion),目的有两个
1.保证缓存的一致性; 2.减少共享磁盘IO的消耗
因此在RAC环境中多个节点保留了同一份的db cache
```
RAC相关的后台进程:
```
LMS(Gobal Cache Service Process) 全局缓存服务进程
处理数据库一个实例到另外一个实例数据的交换,并在不同实例的BUFFER CACHE中传输块镜像;
当在某个数据块上发生一致性读时，LMS负责回滚该数据块，并将它copy到请求的实例上;
每个RAC节点至少有2个LMS进程。 也称作 GCS (Global Cache Services) processes;

全局查询服务守护进程 LMD(Global Enqueue Service Daemon)
LMD进程主要管理对全局队列和资源的访问，并更新相应队列的状态，处理来自于其他实例的资源请求;

LMON(Global Enqueue Service Monitor) 全局查询服务监视进程
监控整个集群状况，维护GCS的内存结构;
处理非正常终止的进程和实例;
当实例离开和加入集群时，锁和资源的重新配置;
管理全局的锁和资源;
监控全局的锁资源，处理死锁和阻塞;

LCK0(Instance Enqueue Process) 实例查询进程
LCK进程用来管理实例间资源请求和跨实例调用操作，调用操作包括数据字典等对象的访问;
并处理非CACEH FUSION的CHACE资源请求(例如：dictionary cache或row cache的请求);
由于LMS进程负责主要的锁管理功能，所以每个实例只有一个LCK进程;

DIAG(Diagnostic Daemon) 诊断守护进程
对实例的健康情况进行监控，同时也监控实例是否挂起或者出现死锁;
收集实例和进程出错时的关键诊断信息,更新alert日志文件，写入一些重要告警信息;
```
RAC本身的服务进程:
```
RAC要对全局资源进行控制，所以实例不能直接访问存储，必须通过CRS层来协调两个实例之间访问存储，这个架构就是CRS
RAC指的是架构，具体是由CRS这套服务来实现的，这套服务里面有下面的四个服务组成

CRS(Cluster Ready Services) 集群资源服务
管理集群内高可用操作的基本程序,CRS管理的任何事物被称之为资源,数据库、实例、监听、虚拟IP(VIP)、应用进程等等
CRS是根据存储于OCR中的资源配置信息来管理这些资源,当一资源的状态改变时，CRS进程生成一个事件

CSS(Cluster Synchronization Service) 集群同步服务
管理集群节点的成员资格,控制哪个结点为集群的成员、结点在加入或离开集群时通知集群成员来控制集群的配置信息
此进程发生故障导致集群重启

EVMD(Event Management) 事件管理服务
事件管理守护进程,发布CRS创建事件的后台进程

ONS(Oracle Notification Service) 事件的发布及订阅服务
通信的快速应用通知事件的发布及订阅服务
```
RAC拓扑结构：
![image](https://github.com/dwjlw1314/DWJ-PROJECT/raw/master/PictureSource/oraclegR1/3.25.1.jpg)

共享存储的方式有很多种，自动存储管理(ASM)、Oracle集群文件系统(OCFS)、裸设备(RAW)、网络共享区域存储(NAS)

Oracle RAC架构延伸:
```
Oracle RAC和RMAN(基于数据库还原、备份的工具)：两者结合，可更高地保障数据库数据的安全性
Oracle RAC和Data Guard(基于日志复制技术的数据同步软件)：两者结合，解决共享存储出现故障的问题
Oracle RAC和Streams：两者结合，构成分布式系统，解决RAC系统负载过高的问题
Oracle RAC和Golden Gate：实现异地数据备份，包括单向和双向复制
```

软件包名 | 包含组件
---|---
database | Database、RMAN、DataGuard
grid     | RAC、ASM、ACFS

组件 | 功能
---|---
Database  | 数据存储、计算、事务处理
DataGuard | 远程复制
RMAN      | 备份恢复
RAC       | 高可用性集群
ACFS      | 可挂载的集群文件系统
ASM       | 集群卷管理，只能由数据库使用的简单文件系统

<font color=#FF0000 size=5><p align="center">Oracle RAC + ASM + Grid安装</p></font>

一、操作系统配置(实验环境包含2台服务器)

step1：主机名配置成node1和node2
```
[root@node1 ~]# vim /etc/sysconfig/network
NETWORKING=yes
HOSTNAME=node1
[root@node2 ~]# vim /etc/sysconfig/network
NETWORKING=yes
HOSTNAME=node2
```
根据网络设置，规划IP地址如下

ip分类 | 网卡名 | given-name | 主机node1 IP | given-name | 主机node2 IP
---|---|---|---
public  | eth0   | node1     | 192.168.0.111 | node2     | 192.168.0.112
virtual | eth0:0 | rac-vip1  | 192.168.0.121 | rac-vip2  | 192.168.0.122
scan    | eth0:1 | rac-scan  | 192.168.0.130 | rac-scan  | 192.168.0.130
private | eth1   | rac-priv1 | 192.168.0.131 | rac-priv2 | 192.168.0.132

实验环境需要为每台虚拟机新增加一块网卡(每个节点是双网卡)，将其配置成桥接模式模式

step2：配置IP地址(主机名：node1和node2)

>[root@node1 ~]# vim /etc/sysconfig/network-scripts/ifcfg-eth0    #修改如下内容

```
ONBOOT=yes
IPADDR=192.168.0.111
NETMASK=255.255.255.0
GATEWAY=192.168.0.1
```

从ifcfg-eth0拷贝一份ifcfg-eth1
>[root@node1 ~]# cp /etc/sysconfig/network-scripts/ifcfg-eth0 ifcfg-eth1

>[root@node1 ~]# vim /etc/sysconfig/network-scripts/ifcfg-eth1   #修改如下内容

```
DEVICE=eth1
HWADDR=00:0c:29:7b:42:3c
UUID=22183cb9-755f-4443-bddd-5a85838e930c
IPADDR=192.168.0.131
```

在2个节点上配置hosts文件，以node1为例添加以下内容
>[root@node1 ~]# vim /etc/hosts

```
#public
192.168.0.111 node1
192.168.0.112 node2
#virtual
192.168.0.121 rac-vip1
192.168.0.122 rac-vip2
#private
192.168.0.131 rac-priv1
192.168.0.132 rac-priv2
#scan
192.168.0.130 rac-scan
```

配置完成后，重启网卡
>[root@node1 ~]# service network restart

查看node1 IP地址，eth0和eth1的ip地址是否正确
>[root@node1 ~]# ip a

节点node2可以通过scp拷贝(快捷方式，不属于必要步骤)
>[root@node1 ~]# scp /etc/hosts node2:/etc/hosts

测试主机与虚拟机、虚拟机与虚拟机之间通讯是否正常
```
主机上ping节点node1
C:\Users\hsql> ping 192.168.0.111/131/112/132
节点node1上ping->其中192.168.0.100为主机ip
[root@node1 ~]# ping 192.168.0.110/112/132
节点node2上ping->其中192.168.0.101为主机ip
[root@node1 ~]# ping 192.168.0.99/111/131
```

二、配置共享存储

磁盘功能 | 个数 | 每个大小GB
---|---|---
ARCHIVE | 1 | 10
OCR     | 3 | 1
DATA    | 3 | 10

创建共享存储，以管理员方式打开DOS窗口，到vmware workstation安装目录下，需要提前磁盘创建路径，运行如下命令
```
vmware-vdiskmanager.exe -c -s 1000MB -a lsilogic -t 2 "F:\vmwareVirtueMachine\RAC\ShareDisk\votingdisk1.vmdk"
vmware-vdiskmanager.exe -c -s 1000Mb -a lsilogic -t 2 "F:\vmwareVirtueMachine\RAC\ShareDisk\votingdisk2.vmdk"
vmware-vdiskmanager.exe -c -s 1000Mb -a lsilogic -t 2 "F:\vmwareVirtueMachine\RAC\ShareDisk\votingdisk3.vmdk"
vmware-vdiskmanager.exe -c -s 10000Mb -a lsilogic -t 2 "F:\vmwareVirtueMachine\RAC\ShareDisk\datadisk1.vmdk"
vmware-vdiskmanager.exe -c -s 10000Mb -a lsilogic -t 2 "F:\vmwareVirtueMachine\RAC\ShareDisk\datadisk2.vmdk"
vmware-vdiskmanager.exe -c -s 10000Mb -a lsilogic -t 2 "F:\vmwareVirtueMachine\RAC\ShareDisk\datadisk3.vmdk"
vmware-vdiskmanager.exe -c -s 10000Mb -a lsilogic -t 2 "F:\vmwareVirtueMachine\RAC\ShareDisk\archdisk1.vmdk"
```
修改所有虚拟机的.vmx配置文件，文件最后加上以下信息：
```
scsi1.present = "TRUE"
scsi1.virtualDev = "lsilogic"
scsi1.sharedBus = "virtual"

scsi1:2.present = "TRUE"
scsi1:2.mode = "independent-persistent"
scsi1:2.filename = "F:\vmwareVirtueMachine\RAC\ShareDisk\votingdisk1.vmdk"
scsi1:2.deviceType = "disk"

scsi1:3.present = "TRUE"
scsi1:3.mode = "independent-persistent"
scsi1:3.filename = "F:\vmwareVirtueMachine\RAC\ShareDisk\votingdisk2.vmdk"
scsi1:3.deviceType = "disk"

scsi1:4.present = "TRUE"
scsi1:4.mode = "independent-persistent"
scsi1:4.filename = "F:\vmwareVirtueMachine\RAC\ShareDisk\votingdisk3.vmdk"
scsi1:4.deviceType = "disk"

scsi1:5.present = "TRUE"
scsi1:5.mode = "independent-persistent"
scsi1:5.filename = "F:\vmwareVirtueMachine\RAC\ShareDisk\datadisk1.vmdk"
scsi1:5.deviceType = "disk"

scsi1:6.present = "TRUE"
scsi1:6.mode = "independent-persistent"
scsi1:6.filename = "F:\vmwareVirtueMachine\RAC\ShareDisk\datadisk2.vmdk"
scsi1:6.deviceType = "disk"

scsi1:7.present = "TRUE"
scsi1:7.mode = "independent-persistent"
scsi1:7.filename = "F:\vmwareVirtueMachine\RAC\ShareDisk\datadisk3.vmdk"
scsi1:7.deviceType = "disk"

scsi1:8.present = "TRUE"
scsi1:8.mode = "independent-persistent"
scsi1:8.filename = "F:\vmwareVirtueMachine\RAC\ShareDisk\archdisk1.vmdk"
scsi1:8.deviceType = "disk"

disk.locking = "false"
diskLib.dataCacheMaxSize = "0"
diskLib.dataCacheMaxReadAheadSize = "0"
diskLib.DataCacheMinReadAheadSize = "0"
diskLib.dataCachePageSize = "4096"
diskLib.maxUnsyncedWrites = "0"
```
(如果启动提示某个scsi1:?已被使用，修改成其他数既可),配置好后重启虚拟机(node1和node2)
>[root@node1 ~]# reboot

最终我们看到的虚拟机的硬件配置大致是这样的

![image](https://github.com/dwjlw1314/DWJ-PROJECT/raw/master/PictureSource/oraclegR1/3.25.2.jpg)

查看添加上去的磁盘信息，磁盘从sdb到sdh
>[root@node1 ~]# fdisk -l

在node1上进行磁盘格式化，由于是共享磁盘，所以只需要在一个节点上执行,以/dev/sdb为例
>[root@node1 ~]# fdisk /dev/sdb    #选值如下：

```
Command (m for help): n
Command action
   e   extended
   p   primary partition (1-4)
p
Partition number (1-4): 1
First cylinder (1-1000, default 1):
Using default value 1
Last cylinder, +cylinders or +size{K,M,G} (1-1000, default 1000):
Using default value 1000

Command (m for help): w
The partition table has been altered!
Syncing disks.
```
添加裸设备，node1和node2都要执,添加以下内容
>[root@node1 ~]# vim /etc/udev/rules.d/60-raw.rules

```
ACTION=="add", KERNEL=="sdb1", RUN+="/bin/raw /dev/raw/raw1 %N"
ACTION=="add", KERNEL=="sdc1", RUN+="/bin/raw /dev/raw/raw2 %N"
ACTION=="add", KERNEL=="sdd1", RUN+="/bin/raw /dev/raw/raw3 %N"
ACTION=="add", KERNEL=="sde1", RUN+="/bin/raw /dev/raw/raw4 %N"
ACTION=="add", KERNEL=="sdf1", RUN+="/bin/raw /dev/raw/raw5 %N"
ACTION=="add", KERNEL=="sdg1", RUN+="/bin/raw /dev/raw/raw6 %N"
ACTION=="add", KERNEL=="sdh1", RUN+="/bin/raw /dev/raw/raw7 %N"
KERNEL=="raw[1]", MODE="0660", OWNER="grid", GROUP="asmadmin"
KERNEL=="raw[2]", MODE="0660", OWNER="grid", GROUP="asmadmin"
KERNEL=="raw[3]", MODE="0660", OWNER="grid", GROUP="asmadmin"
KERNEL=="raw[4]", MODE="0660", OWNER="grid", GROUP="asmadmin"
KERNEL=="raw[5]", MODE="0660", OWNER="grid", GROUP="asmadmin"
KERNEL=="raw[6]", MODE="0660", OWNER="grid", GROUP="asmadmin"
KERNEL=="raw[7]", MODE="0660", OWNER="grid", GROUP="asmadmin"
```
启动裸设备，node1和node2都要执都
>[root@node1 ~]# start_udev

查看裸设备，node1和node2都要查看,类似如下结果
>[root@node1 ~]# raw -qa

```
/dev/raw/raw1:	bound to major 8, minor 17
/dev/raw/raw2:	bound to major 8, minor 33
.....
```

三、Oracle和Grid预配置

step1：创建用户及安装目录(node1和node2)
```
[root@node1 ~]# groupadd -g 1000 oinstall
[root@node1 ~]# groupadd -g 1020 asmadmin
[root@node1 ~]# groupadd -g 1021 asmdba
[root@node1 ~]# groupadd -g 1022 asmoper
[root@node1 ~]# groupadd -g 1031 dba
[root@node1 ~]# groupadd -g 1032 oper

[root@node1 ~]# useradd -u 1100 -g oinstall -G asmadmin,asmdba,asmoper,oper,dba grid
[root@node1 ~]# useradd -u 1101 -g oinstall -G dba,asmdba,oper oracle
[root@node1 ~]# passwd oracle   #设置oracle用户密码oracle
[root@node1 ~]# passwd grid     #设置grid用户密码grid

[root@node1 ~]# mkdir -p /u01/app/11.2.0/grid
[root@node1 ~]# mkdir -p /u01/app/grid
[root@node1 ~]# mkdir /u01/app/oracle
[root@node1 ~]# chown -R grid:oinstall /u01
[root@node1 ~]# chown oracle:oinstall /u01/app/oracle
[root@node1 ~]# chmod -R 775 /u01/
```
安装目录结构如下：
>[root@node1 u01]# tree

```
.
└── app
    ├── 11.2.0
    │   └── grid
    ├── grid
    └── oracle
```
step2：切换到oracle用户，配置环境变量，ORACLE_SID需根据主机来填写,添加下面的部分
>[oracle@node1 ~]$ vim .bash_profile

```
export TMP=/tmp
export TMPDIR=$TMP
export ORACLE_SID=santdb1    #如果是节点2，则：export ORACLE_SID=santdb2
export ORACLE_BASE=/u01/app/oracle
export ORACLE_HOME=$ORACLE_BASE/product/11.2.0/db_1
export TNS_ADMIN=$ORACLE_HOME/network/admin
export PATH=/usr/sbin:$PATH
export PATH=$ORACLE_HOME/bin:$PATH
export LD_LIBRARY_PATH=$ORACLE_HOME/lib:/lib:/usr/lib
export CLASSPATH=$ORACLE_HOME/JRE:$ORACLE_HOME/jlib:$ORACLE_HOME/rdbms/jlib
umask 022
```
切换到grid用户，配置环境变量，ORACLE_SID需根据主机来填写,添加下面的部分
>[grid@node1 ~]$ vim .bash_profile

```
export TMP=/tmp
export TMPDIR=$TMP
export ORACLE_SID=+ASM1   #如果是节点2，则：export ORACLE_SID=+ASM2
export ORACLE_BASE=/u01/app/grid
export ORACLE_HOME=/u01/app/11.2.0/grid
export PATH=/usr/sbin:$PATH
export PATH=$ORACLE_HOME/bin:$PATH
export LD_LIBRARY_PATH=$ORACLE_HOME/lib:/lib:/usr/lib
export CLASSPATH=$ORACLE_HOME/JRE:$ORACLE_HOME/jlib:$ORACLE_HOME/rdbms/jlib
umask 022
```
step3：配置oracle用户的节点互信,首先创建ssh key文件
```
[oracle@node1 ~]$ mkdir ~/.ssh
[oracle@node1 ~]$ chmod 755 ~/.ssh
[oracle@node1 ~]$ /usr/bin/ssh-keygen -t rsa
[oracle@node1 ~]$ /usr/bin/ssh-keygen -t dsa
```
将所有的key文件汇总到一个总的认证文件中，只需在node1上执行
```
[oracle@node1 ~]$ cd ~/.ssh
[oracle@node1 .ssh]$ ssh node1 cat ~/.ssh/id_rsa.pub >> authorized_keys
[oracle@node1 .ssh]$ ssh node2 cat ~/.ssh/id_rsa.pub >> authorized_keys
[oracle@node1 .ssh]$ ssh node1 cat ~/.ssh/id_dsa.pub >> authorized_keys
[oracle@node1 .ssh]$ ssh node2 cat ~/.ssh/id_dsa.pub >> authorized_keys
```
将node1上authorized_keys拷贝一份完整的认证文件到node2
>[oracle@node1 .ssh]$ scp authorized_keys node2:~/.ssh/

在2个节点都执行以下命令，以node1为例
```
[oracle@node1 .ssh]$ ssh node1 date
[oracle@node1 .ssh]$ ssh node2 date
[oracle@node1 .ssh]$ ssh rac-priv1 date
[oracle@node1 .ssh]$ ssh rac-priv2 date
```
检验是否配置成功，在node1上不用输入密码就可以通过ssh连接node2,说明配置成功
>[oracle@node1 .ssh]$ ssh node2  <br>
Last login: Thu Nov  1 14:08:17 2015 from node1

step4：配置grid用户的节点互信,与Oracle用户执行方式相同，只是执行脚本的用户变成了grid

四、配置系统内核参数
>[root@node1 ~]# vim /etc/sysctl.conf     #在末尾添加

```
fs.aio-max-nr = 1048576
fs.file-max = 6815744
kernel.shmmni = 4096
kernel.sem = 250 32000 100 128
net.ipv4.ip_local_port_range = 9000 65500
net.core.rmem_default = 262144
net.core.rmem_max = 4194304
net.core.wmem_default = 262144
net.core.wmem_max = 1048586
net.ipv4.tcp_wmem = 262144 262144 262144
net.ipv4.tcp_rmem = 4194304 4194304 4194304
```
参数说明：
```
fs.file-max：系统中所允许的文件句柄最大数目
kernel.shmmni：整个系统共享内存段的最大数目
net.core.rmem_max：套接字接收缓冲区大小的最大值
net.core.wmem_max：套接字发送缓冲区大小的最大值
net.core.rmem_default：套接字接收缓冲区大小的缺省值
net.core.wmem_default：套接字发送缓冲区大小的缺省值
net.ipv4.ip_local_port_range：应用程序可使用的IPv4端口范围
```
内核参数修改生效
>[root@node1 ~]# sysctl -p

配置oracle、grid用户的shell限制,在末尾添加以下内容
>[root@node1 ~]# vim /etc/security/limits.conf   

```
grid soft nproc 2047
grid hard nproc 16384
grid soft nofile 1024
grid hard nofile 65536
oracle soft nproc 2047
oracle hard nproc 16384
oracle soft nofile 1024
oracle hard nofile 65536
```

五、安装依赖软件包

不同的操作系统安装不同的包，(以下所操作的系统是开发模式)；具体可看官方教程
>http://docs.oracle.com/cd/E11882_01/install.112/e41961/prelinux.htm#CWLIN

Linux x86-64 Oracle Grid Infrastructure and Oracle RAC Package Requirements
```
binutils-2.20.51.0.2-5.11.el6 (x86_64) or later
compat-libcap1-1.10-1 (x86_64) or later
compat-libstdc++-33-3.2.3-69.el6 (x86_64) or later
compat-libstdc++-33-3.2.3-69.el6.i686 or later
gcc-4.4.4-13.el6 (x86_64) or later
gcc-c++-4.4.4-13.el6 (x86_64) or later
glibc-2.12-1.7.el6 (i686) or later
glibc-2.12-1.7.el6 (x86_64) or later
glibc-devel-2.12-1.7.el6 (x86_64) or later
glibc-devel-2.12-1.7.el6.i686 or later
ksh or later
libgcc-4.4.4-13.el6 (i686) or later
libgcc-4.4.4-13.el6 (x86_64) or later
libstdc++-4.4.4-13.el6 (x86_64) or later
libstdc++-4.4.4-13.el6.i686 or later
libstdc++-devel-4.4.4-13.el6 (x86_64) or later
libstdc++-devel-4.4.4-13.el6.i686 or later
libaio-0.3.107-10.el6 (x86_64) or later
libaio-0.3.107-10.el6.i686 or later
libaio-devel-0.3.107-10.el6 (x86_64) or later
libaio-devel-0.3.107-10.el6.i686 or later
make-3.81-19.el6 or later
sysstat-9.0.4-11.el6 (x86_64) or later
```
Linux x86-64 Oracle Database Features Package Requirements
```
unixODBC-2.2.14-11.el6 (x86_64) or later
unixODBC-2.2.14-11.el6.i686 or later
unixODBC-devel-2.2.14-11.el6 (x86_64) or later
unixODBC-devel-2.2.14-11.el6.i686 or later
```

六、安装Grid

step1：解压grid安装包
>[root@node1 ~]# unzip linux.x64_11gR2_grid.zip

step2：安装cvuqdisk包，用于检查RAC环境配置
```
[root@node1 ~]# cd grid/rpm
[root@node1 rpm]# rpm -ivh cvuqdisk-1.0.7-1.rpm
```
step3：检查环境配置，这里需要一项一项的去核对，确保没有不通过的选项
```
[root@node1 rpm]# su - grid
[grid@node1 ~]$ cd ~/grid
[grid@node1 grid]$ ./runcluvfy.sh stage -pre crsinst -n node1,node2 -fixup -verbose
```
step4：开启安装，安装只需要在一个节点上进行即可，会自动同步到另一个节点，下面以node1为例
>[grid@node1 grid]$ ./runInstaller

![image](https://github.com/dwjlw1314/DWJ-PROJECT/raw/master/PictureSource/oraclegR1/3.25.3.jpg)
选择：Install and Configure Grid Infrastructure for a Cluster

![image](https://github.com/dwjlw1314/DWJ-PROJECT/raw/master/PictureSource/oraclegR1/3.25.4.jpg)
选择：Advanced Installation，语言选择默认即可

![image](https://github.com/dwjlw1314/DWJ-PROJECT/raw/master/PictureSource/oraclegR1/3.25.5.jpg)
这一步需要注意SCAN Name，需要与/etc/hosts文件里的值相同，取消Configure GNS

![image](https://github.com/dwjlw1314/DWJ-PROJECT/raw/master/PictureSource/oraclegR1/3.25.6.jpg)
添加另一个节点，Virtual IP Name与/etc/hosts文件里#virtual名字相同

![image](https://github.com/dwjlw1314/DWJ-PROJECT/raw/master/PictureSource/oraclegR1/3.25.7.jpg)
添加节点之后，点击SSH Connectivity，填写grid用户的密码，点击setup

![image](https://github.com/dwjlw1314/DWJ-PROJECT/raw/master/PictureSource/oraclegR1/3.25.8.jpg)
配置网络，这里至少需要两个网络，根据前面的IP规划表选择即可

![image](https://github.com/dwjlw1314/DWJ-PROJECT/raw/master/PictureSource/oraclegR1/3.25.9.jpg)
选择：Automatic Storage Management(ASM)

![image](https://github.com/dwjlw1314/DWJ-PROJECT/raw/master/PictureSource/oraclegR1/3.25.10.jpg)
选择3块1GB的磁盘，组成一个磁盘组OCR

![image](https://github.com/dwjlw1314/DWJ-PROJECT/raw/master/PictureSource/oraclegR1/3.25.11.jpg)
输入密码，这里有警告信息，不符合Oracle的密码规定，忽略不管

![image](https://github.com/dwjlw1314/DWJ-PROJECT/raw/master/PictureSource/oraclegR1/3.25.12.jpg)
选择：Do not use Interlligent Platform Management Interface(IPMI)

![image](https://github.com/dwjlw1314/DWJ-PROJECT/raw/master/PictureSource/oraclegR1/3.25.13.jpg)
选择默认即可，3种用户组是之前创建好的

![image](https://github.com/dwjlw1314/DWJ-PROJECT/raw/master/PictureSource/oraclegR1/3.25.14.jpg)
设置Oracle Base和software Location路径,界面默认加载之前创建好的路径,采用默认既可

![image](https://github.com/dwjlw1314/DWJ-PROJECT/raw/master/PictureSource/oraclegR1/3.25.15.jpg)
设置orainverntory Directory,默认即可

![image](https://github.com/dwjlw1314/DWJ-PROJECT/raw/master/PictureSource/oraclegR1/3.25.16.jpg)
界面自动进行 Perform Prerequisite Checks，NTP检查失败，忽略，其他需要安装依赖解决

![image](https://github.com/dwjlw1314/DWJ-PROJECT/raw/master/PictureSource/oraclegR1/3.25.17.jpg)
开始进行安装…

![image](https://github.com/dwjlw1314/DWJ-PROJECT/raw/master/PictureSource/oraclegR1/3.25.18.jpg)
安装过程中，会弹出如下窗口：按照其要求，一个节点一个节点的执行，千万不要2个节点一起执行

在node1上打开一个shell终端，以root用户登录
>[root@node1 ~]# /u01/app/oraInventory/orainstRoot.sh

待上面语句执行完成，再执行下面语句,如果遇到CRS-4124错误，使用下面方法进行处理
>[root@node1 ~]# /u01/app/11.2.0/grid/root.sh

```
重新执行root.sh之前先删除配置
[root@node1 ~]# /u01/app/11.2.0/grid/crs/install/roothas.pl -deconfig -force -verbose
继续执行脚本，同时执行下一行的dd命令
[root@node1 ~]# /u01/app/11.2.0/grid/root.sh
在执行root.sh之前，在新的命令行窗口执行以下命令
[root@node1 ~]# /bin/dd if=/var/tmp/.oracle/npohasd of=/dev/null bs=1024 count=1
如果出现 /bin/dd: opening '/var/tmp/.oracle/npohasd': No such file or directory
该命令可能直到出现<Adding daemon to inittab>信息才可执行
```
在node2上打开一个shell终端，以root用户权限执行上面同样的指令，指令执行完成。点击OK

![image](https://github.com/dwjlw1314/DWJ-PROJECT/raw/master/PictureSource/oraclegR1/3.25.19.jpg)
由于配置了/etc/hosts来解析SCAN，导致未走DNS来进行SCAN的解析，爆出此错误，如果可以ping通rac-scan 则忽略掉

七、配置磁盘组

只需在node1上执行
>[grid@node1 grid]$ asmca

![image](https://github.com/dwjlw1314/DWJ-PROJECT/raw/master/PictureSource/oraclegR1/3.25.20.jpg)
点击create

![image](https://github.com/dwjlw1314/DWJ-PROJECT/raw/master/PictureSource/oraclegR1/3.25.21.jpg)
创建磁盘组DATA

![image](https://github.com/dwjlw1314/DWJ-PROJECT/raw/master/PictureSource/oraclegR1/3.25.22.jpg)
创建磁盘组ARCH

![image](https://github.com/dwjlw1314/DWJ-PROJECT/raw/master/PictureSource/oraclegR1/3.25.23.jpg)
最终创建结果

八、Grid Infrastructure 中提供被称作 Oracle Restart 的功能。代理会检查数据库、监听器、ASM 等重要组件的可用性，在这些组件关闭的情况下自动将其启动。用于检查可用性并重新启动故障组件的组件被称作 HAS(高可用性服务)

检查 HAS 自身可用性的方法(/u01/app/11.2.0/grid/bin)
>[grid@node1 ~]$ crsctl check has  <br>
CRS-4638: Oracle High Availability Services is online

该服务必须联机才能生效。可通过以下命令来检查版本
>[grid@node1 ~]$ crsctl query crs releaseversion  <br>
Oracle High Availability Services release version on the local node is [11.2.0.1.0]

>[grid@node1 ~]$ crsctl query crs softwareversion  <br>
Oracle High Availability Services version on the local node is [11.2.0.1.0]

该服务可设置为自动启动。检查该服务是否已配置为自动启动
>[root@node1 ~]# /u01/app/11.2.0/grid/bin/crsctl config has  <br>
CRS-4622: Oracle High Availability Services autostart is enabled.

设置为自动启动，可以通过以下方式进行设置，将enable替换为disable取消自动启动属性
>[root@node1 ~]# /u01/app/11.2.0/grid/bin/crsctl enable has  <br>
CRS-4622: Oracle High Availability Services autostart is enabled.

如果未启动，可通过如下方式启动，如果遇到CRS-4124错误，用以下方法处理
>[root@node1 ~]# /u01/app/11.2.0/grid/bin/crsctl start has

```
在启动CRS之前，先在node1和node2运行指定dd命令
[root@node1 ~]# /bin/dd if=/var/tmp/.oracle/npohasd of=/dev/null bs=1024 count=1
然后在node1和node2启动has
[root@node1 ~]# /u01/app/11.2.0/grid/bin/crsctl start has
```

九、安装Oracle软件(node1执行会同步到node2)

解压安装包，拷贝程序到oracle用户目录下，调出图形界面
>[oracle@node1 database]$ ./runInstaller

![image](https://github.com/dwjlw1314/DWJ-PROJECT/raw/master/PictureSource/oraclegR1/3.25.24.jpg)
取消<I wish to receive security updates via My Oracle Support>,Next选择第二个，只安装数据库软件

![image](https://github.com/dwjlw1314/DWJ-PROJECT/raw/master/PictureSource/oraclegR1/3.25.25.jpg)
选择Next,如果节点互信总是出问题，重新手动配置一下就ok

![image](https://github.com/dwjlw1314/DWJ-PROJECT/raw/master/PictureSource/oraclegR1/3.25.26.jpg)
路径使用默认既可

![image](https://github.com/dwjlw1314/DWJ-PROJECT/raw/master/PictureSource/oraclegR1/3.25.27.jpg)
时钟同步问题，忽略；其他依赖需要解决

![image](https://github.com/dwjlw1314/DWJ-PROJECT/raw/master/PictureSource/oraclegR1/3.25.28.jpg)
点击Finish等待安装完成,要拷贝几乎4G点数据，会很耗时间

![image](https://github.com/dwjlw1314/DWJ-PROJECT/raw/master/PictureSource/oraclegR1/3.25.29.jpg)
在node1和node2运行指定脚本，安装完成
>[root@node1 ~]# /u01/app/oracle/product/11.2.0/db_1/root.sh

十、在oracle用户下创建数据库
>[oracle@node1 database]$ dbca

```
终端启动图形化程序界面时报错：No protocol specified
新开shell终端切换到root用户
[root@node1 ~]# xhost +
提示access control disabled, clients can connect from any host即可
```

![image](https://github.com/dwjlw1314/DWJ-PROJECT/raw/master/PictureSource/oraclegR1/3.25.30.jpg)
选择：Oracle Real Application Clusters database

![image](https://github.com/dwjlw1314/DWJ-PROJECT/raw/master/PictureSource/oraclegR1/3.25.31.jpg)
选择Next

![image](https://github.com/dwjlw1314/DWJ-PROJECT/raw/master/PictureSource/oraclegR1/3.25.32.jpg)
Global Database Name输入santdb，node1和node2全部选择,需要与环境变量ORACLE_SID对应

![image](https://github.com/dwjlw1314/DWJ-PROJECT/raw/master/PictureSource/oraclegR1/3.25.33.jpg)
设置用户密码，与用户名相同

![image](https://github.com/dwjlw1314/DWJ-PROJECT/raw/master/PictureSource/oraclegR1/3.25.34.jpg)
默认既可，会提示输入asmsnmp密码到asm

![image](https://github.com/dwjlw1314/DWJ-PROJECT/raw/master/PictureSource/oraclegR1/3.14.29.jpg)
保留Fast Recovery Area以及Fast Recovery Area Size的默认数值

![image](https://github.com/dwjlw1314/DWJ-PROJECT/raw/master/PictureSource/oraclegR1/3.14.30.jpg)
选择Sample Schemas

![image](https://github.com/dwjlw1314/DWJ-PROJECT/raw/master/PictureSource/oraclegR1/3.14.31.jpg)
选择Use Unicode(AL32UTF8), 在“Memory”页签中修改“Memory Size”大小为内存的40％

![image](https://github.com/dwjlw1314/DWJ-PROJECT/raw/master/PictureSource/oraclegR1/3.14.32.jpg)
选择Next

![image](https://github.com/dwjlw1314/DWJ-PROJECT/raw/master/PictureSource/oraclegR1/3.14.33.jpg)
勾选Create Database,选择Finish

![image](https://github.com/dwjlw1314/DWJ-PROJECT/raw/master/PictureSource/oraclegR1/3.14.34.jpg)

确认所创建的数据库，选择ok，等待数据库创建完成

十一、检查安装结果,如果有OFFLINE状态，则手动启动
>[grid@node1 ~]$ crsctl status res -t

```
--------------------------------------------------------------------------------
NAME           TARGET  STATE        SERVER                   STATE_DETAILS       
--------------------------------------------------------------------------------
Local Resources
--------------------------------------------------------------------------------
ora.ARCH.dg
               ONLINE  ONLINE       node1                                        
               ONLINE  ONLINE       node2                                        
ora.DATA.dg
               ONLINE  ONLINE       node1                                        
               ONLINE  ONLINE       node2                                        
ora.LISTENER.lsnr
               ONLINE  ONLINE       node1                                        
               ONLINE  ONLINE       node2                                        
ora.OCR.dg
               ONLINE  ONLINE       node1                                        
               ONLINE  ONLINE       node2                                        
ora.asm
               ONLINE  ONLINE       node1                    Started             
               ONLINE  ONLINE       node2                    Started             
ora.eons
               ONLINE  ONLINE       node1                                        
               ONLINE  ONLINE       node2                                        
ora.gsd
               OFFLINE OFFLINE      node1                                        
               OFFLINE OFFLINE      node2                                        
ora.net1.network
               ONLINE  ONLINE       node1                                        
               ONLINE  ONLINE       node2                                        
ora.ons
               ONLINE  ONLINE       node1                                        
               ONLINE  ONLINE       node2                                        
--------------------------------------------------------------------------------
Cluster Resources
--------------------------------------------------------------------------------
ora.LISTENER_SCAN1.lsnr
      1        ONLINE  ONLINE       node1                                        
ora.node1.vip
      1        ONLINE  ONLINE       node1                                        
ora.node2.vip
      1        ONLINE  ONLINE       node2                                        
ora.oc4j
      1        OFFLINE OFFLINE                                                   
ora.santdb.db
      1        ONLINE  ONLINE       node1                    Open                
      2        ONLINE  ONLINE       node2                    Open                
ora.scan1.vip
      1        ONLINE  ONLINE       node1
```

手动启动arch
>[grid@node1 ~]$ srvctl start diskgroup -g arch

手动启动data
>[grid@node1 ~]$ srvctl start diskgroup -g data

手动启动数据库
>[grid@node1 ~]$ srvctl start database -d santdb

获取数据库安装的详细参数，使用config选项
>[grid@node1 ~]$ srvctl config database -d santdb -a

ORA-12545: Connect failed because target host or object does not exist修改方法如下：
```
---------node1
SQL> alter system set local_listener='(DESCRIPTION=(ADDRESS_LIST=(ADDRESS=(PROTOCOL=TCP)(HOST=192.168.0.121)(PORT=1521))))' scope=both sid='santdb1';

---------node2
SQL> alter system set local_listener='(DESCRIPTION=(ADDRESS_LIST=(ADDRESS=(PROTOCOL=TCP)(HOST=192.168.0.122)(PORT=1521))))' scope=both sid='santdb2';

SQL> alter system register;
```
修改前:
```
----------node1和node2配置均如下
SQL> show parameter listener
local_listener           string      (DESCRIPTION=(ADDRESS_LIST=(AD
                                     DRESS=(PROTOCOL=TCP)(HOST=rac-vip1/2)(PORT=1521))))
remote_listener          string      rac-scan:1521
```
修改后：
```
-------------node1
SQL> show parameter listener

NAME                      TYPE         VALUE
------------------------  -----------  ------------------------------
listener_networks         string
local_listener            string     (DESCRIPTION=(ADDRESS_LIST=(AD
                                      DRESS=(PROTOCOL=TCP)(HOST=192.168.0.121)(PORT=1521))))
-------------node2
SQL> show parameter listener
local_listener           string     (DESCRIPTION=(ADDRESS_LIST=(AD
                                     DRESS=(PROTOCOL=TCP)(HOST=192.168.0.122)(PORT=1521))))
```

<font color=#FF0000 size=5><p align="center">oracle表空间创建</p></font>

查询数据库ASM磁盘组信息
>SQL> select name,total_mb,free_mb,USABLE_FILE_MB from v$asm_diskgroup;

查看当前数据库已有数据表空间信息,获取存储路径
>SQL> select tablespace_name,file_name,bytes/1024/1024/1024 gb from dba_data_files;

查看当前数据库已有临时表空间信息,获取存储路径
>SQL> select tablespace_name,file_name,bytes/1024/1024/1024 gb from dba_temp_files;

后面的步骤可以参考 <linux系统oracle 11gR1安装>

<font color=#FF0000 size=5><p align="center">oracle rac的启动和关闭顺序</p></font>

1.使用crs_stat 命令查询RAC节点的服务状态是否正常
>[root@node1 ~]# crs_stat -t -v

```
Name           Type           R/RA   F/FT   Target    State     Host        
----------------------------------------------------------------------
ora.ARCH.dg    ora....up.type 0/5    0/     ONLINE    ONLINE    node1       
ora.DATA.dg    ora....up.type 0/5    0/     ONLINE    ONLINE    node1       
ora....ER.lsnr ora....er.type 0/5    0/     ONLINE    ONLINE    node1       
ora....N1.lsnr ora....er.type 0/5    0/0    ONLINE    ONLINE    node1       
ora.OCR.dg     ora....up.type 0/5    0/     ONLINE    ONLINE    node1       
ora.asm        ora.asm.type   0/5    0/     ONLINE    ONLINE    node1       
ora.eons       ora.eons.type  0/3    0/     ONLINE    ONLINE    node1       
ora.gsd        ora.gsd.type   0/5    0/     OFFLINE   OFFLINE               
ora....network ora....rk.type 0/5    0/     ONLINE    ONLINE    node1       
ora....SM1.asm application    0/5    0/0    ONLINE    ONLINE    node1       
ora....E1.lsnr application    0/5    0/0    ONLINE    ONLINE    node1       
ora.node1.gsd  application    0/5    0/0    OFFLINE   OFFLINE               
ora.node1.ons  application    0/3    0/0    ONLINE    ONLINE    node1       
ora.node1.vip  ora....t1.type 0/0    0/0    ONLINE    ONLINE    node1       
ora....SM2.asm application    0/5    0/0    ONLINE    ONLINE    node2       
ora....E2.lsnr application    0/5    0/0    ONLINE    ONLINE    node2       
ora.node2.gsd  application    0/5    0/0    OFFLINE   OFFLINE               
ora.node2.ons  application    0/3    0/0    ONLINE    ONLINE    node2       
ora.node2.vip  ora....t1.type 0/0    0/0    ONLINE    ONLINE    node2       
ora.oc4j       ora.oc4j.type  0/5    0/0    OFFLINE   OFFLINE               
ora.ons        ora.ons.type   0/3    0/     ONLINE    ONLINE    node1       
ora.santdb.db  ora....se.type 0/2    0/1    ONLINE    ONLINE    node1       
ora.scan1.vip  ora....ip.type 0/0    0/0    ONLINE    ONLINE    node1  
```
结果状态中关于ora.gsd和ora.oc4j处于OFFLINE，RAC可以不用理会，想启动可以采用下面方式
>[grid@node1 ~]$ srvctl enable oc4j   <br>
>[grid@node1 ~]$ srvctl start oc4j -v

2.使用srvctl命令按照关闭顺序：关闭数据库(实例)-->关闭ASM实例-->关闭节点服务 关闭集群服务
```
关闭数据库
[oracle@node1 ~]$ srvctl stop database -d santdb
关闭各节点的ASM实例
[oracle@node1 ~]$ srvctl stop asm -n node2
[oracle@node1 ~]$ srvctl stop asm -n node1
关闭各节点的服务包括：listener、gsd、ons、vip
[oracle@node1 ~]$ srvctl stop nodeapps -n node2
[oracle@node1 ~]$ srvctl stop nodeapps -n node1
```
3.使用srvctl命令按照启动顺序：启动节点服务-->启动ASM实例-->启动数据库实例 启动集群服务
```
启动各节点的服务包括：listener、gsd、ons、vip
[oracle@node1 ~]$ srvctl start nodeapps -n node2
[oracle@node1 ~]$ srvctl start nodeapps -n node1
启动各节点的ASM实例
[oracle@node1 ~]$ srvctl start asm -n node2
[oracle@node1 ~]$ srvctl start asm -n node1
启动数据库
[oracle@node1 ~]$ srvctl start database -d santdb
```
