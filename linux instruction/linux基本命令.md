<font color=#FF0000 size=5>linux基本用法</font>

linux系统命令大全
>http://man.linuxde.net/

常用基本命令
```
strace -p 9586 -o strace.log                                         #对9586进程进行跟踪,结果写入strace.log
source /home/oracle/.bash_profile                                    #设置oracle用户环境变量
source /etc/profile                                                  #使修改后的环境变量生效
sysctl -p                                                            #使修改后的内核参数生效
service --status-all                                                 #查看系统所有进程启动状态信息
whoami / who am i                                                    #显示当前登录系统用户
who 和 w                                                             #显示当前登录用户详细信息
view abc.txt                                                         #vim只读版本，防止误操作
> :%!xxd                                                             #16进制显示文件内容
cat > gjsy.txt << end                                                #使用end作为文件结束输入标记
cat file1 file2 file3 > outfile                                      #文件合并输出到outfile
su - root -c "useradd  test"                                         #不切换root用户，一次执行root权限命令
touch {1..5}.txt                                                     #批量创建文件
yum -y install --installroot=/usr/local/apache  程序名               #指定安装路径
strings /lib64/libc.so.6 |grep GLIBC                                 #查看linux系统GLIBC库版本
mount -t iso9660                                                     #查看文件系统类型相关盘符权限详细信息
mtype [-st] [filename]                                               #是mtools工具集指令，模拟MS-DOS的指令显示文件的内容
> -s:去除8位字符码集的第一个位，使它兼容于7位的ASCII; -t:将MS-DOS文件中的"换行+光标移至行首"字符转换成Linux的换行字符
nm -a *.so                                                           #查看动态库文件中的函数名
free  -m / -h                                                        #已MB为单位显示当前内存使用情况
autoconf                                                             #生成编译configure文件
ldd  name                                                            #输出程序name使用的动态库和动态库的位置
diff -u file1 file2                                                  #比较file1和file2两个文件
vimdiff file1 file2                                                  #左右屏幕比较文件
hexdump -C filename                                                  #16进制方式查看filename内容
size pragma_name                                                     #获取可执行程序各个段的大小
nano                                                                 #vim相同的文本编辑命令
nload / "watch more /proc/net/dev"                                   #监控服务器网卡流量
ntsysv                                                               #自启动服务配置程序
id  username                                                         #查看username用户的uid，gid，groups
chattr +a / +i  filename/dir                                         #增加文件系统扩展属性
> +a 只能向文件追加内容  +i 文件不能被修改，如果目录具有这个属性，任何进程只能修改目录下的文件，不允许建立和删除文件
lsattr filename  / lsattr -d dirname                                 #查看文件系统扩展属性
du -sh / du -h -x --max-depth=1                                      #查看目录下每个文件夹占用大小
bc                                                                   #linux计算器
cal                                                                  #显示当前日历或指定日期的日历
iconv                                                                #文件编码转换命令
grep -rn "AC" ./                                                     #当前目录查找AC所在文件
dos2unix/unix2dos                                                    #将文件中的\r\n和\n互相转换
ethtool eth0                                                         #主要用于查询配置网卡参数
brctl show                                                           #查看网桥和端口连接信息
ip link set dev docker0 down                                         #关闭网桥docker0
brctl delbr docker0                                                  #删除网桥docker0
ifcfg eth0 add/del ip/mask                                           #eth0网卡添加删除ip
ifcfg eth0 up/down                                                   #eth0网卡启用和关闭
iwconfig                                                             #用于查看无线网络
getenforce                                                           #查看SElinux状态
setenforce                                                           #设置SElinux状态
setfacl/getfacl                                                      #设置和获取文件的ACL权限
nmap                                                                 #Linux下的网络安全扫描和嗅探工具包
nmapfe                                                               #nmap图形化界面
nmtui                                                                #NetworkManager界面，类似setup命令
nmcli con                                                            #查看网卡uuid和Type信息
nm-connection-editor                                                 #打开网络连接编辑界面
dmidecode                                                            #Linux下获取详细硬件信息的工具
htpasswd -c /opt/passwd user                                         #给user用户创建密码文件
nice -n 4 command / renice -n 1 command                              #设置和修改command或进程优先级
lscpu                                                                #cpu信息和/proc/cpuinfo一样
md5sum <filename>                                                    #获得文件的md5值
getent passwd root                                                   #显示指定数据库passwd的数据
支持的数据库：
> ahosts ahostsv4 ahostsv6 aliases ethers group gshadow hosts netgroup networks
  passwd protocols rpc services shadow
nslookup www.qq.com                                                  #查看域名对应的ip详细信息
dig www.baidu.com                                                    #域名查询工具,可以用来测试域名系统工作是否正常
ss -lnpt                                                             #命令用来显示处于活动状态的套接字信息
iftop                                                                #类似于top的实时网络流量监控工具
curl www.baidu.com                                                   #利用URL语法在命令行方式下工作的开源文件传输工具
watch iptables -vnxL                                                 #定期执行iptables命令，输出到屏幕
dump    (只适用于ext*文件系统)                                        #系统分区备份命令，分区可以增量备份，目录只能完全备份
restore   (只适用于ext*文件系统)                                      #系统分区恢复备份命令
xfsdump -I                                                           #查看xfs文件系统备份详情
xfsdump -f /opt/dwj -s grub2/grub.cfg /boot -L grub2 -M boot         #备份/boot/grub2/grub.cfg文件
xfsdump -f /opt/lgl /dev/sdb5 -L sdb5 -M sdb5                        #备份sdb5文件分区
xfsrestore -f /opt/dwj /boot                                         #恢复备份的文件系统
xdpyinfo                                                             #查看系统X11详细信息
dmesg                                                                #查看系统启动过程的所有信息
time 命令                                                            #显示命令执行的时间             
partprobe                                                            #重新读取分区表信息
pstree -p                                                            #查看进程树结构
parted -l                                                            #输出文件系统类型
lsblk / lscpu / lsscsi / lspci / lsusb                               #查看磁盘分区树状结构
blkid [-kU]                                                          #locate/print block device attributes
lsof                                                                 #列出系统或进程调用打开和使用了哪些文件和动态库
fuser -m -u -v                                                       #与lsof效果相似
find  ! -name '.*' -a ! -regex '.*/\.[^/]*/.*'                       #查找除隐藏文件以外的文件
lsmod |grep ftp                                                      #显示linux系统内核模块ftp是否加载
modprobe -l|grep ftp                                                 #查看系统内核模块名字*.ko文件
modprobe  nf_conntrack_ftp                                           #加载内核模块ftp
getconf LONG_BIT                                                     #查看CPU位数
umask -p                                                             #设置新创建目录或文件的默认权限
> umask 0022  目录：7- 掩码权限数字 ; 文件：目录权限去掉执行权限         #--案例--
users                                                                #显示当前登录系统的所有用户
uuidgen                                                              #随机生成UUID值
uname -r / -s / -a                                                   #查看内核版本信息(内核版本，内核名称，所有)
cat /etc/centos-release                                              #查看linux系统版本
whereis  nicstat                                                     #查找程序所在目录，更大的系统目录
which nicstat                                                        #查找nicstat程序所在目录
whatis                                                               #查看可执行命令的相关描述
whiptail                                                             #在shell脚本运行中显示对话框
> whiptail --msgbox showmessage 100 200                              #--案例--
locate stdio.h                                                       #从文件系统数据库索引中查找任何类型文件
ldconfig -v                                                          #显示所有软链接对应关系
updatedb                                                             #定期更新locate数据库，命令由cron运行
HISTTIMEFORMAT="%F %T "                                              #history历史查看显示时间格式环境变量设置
systemctl list-unit-files                                            #查看系统中所有服务启动状态
tune2fs -l /dev/sdb5 || dumpe2fs -h /dev/sdb5                        #查看分区文件系统信息
tune2fs -l /dev/sda1 | grep create || passwd -S root                 #查看操作系统的安装时间
e2label /dev/sdb5 mydisk                                             #设置分区卷标信息,默认是查看
mkfs -t ext4 /dev/sdb1 或者 mkfs.ext4 /dev/sdb1                      #新建分区后格式化普通文件系统
mkswap /dev/sdb6                                                     #新建swap分区后格式化交换文件系统
swapon/swapoff /dev/sdb6                                             #加入和取消新扩容的swap交换分区
swapon -a / swapon -s                                                #激活交换空间和查看交换空间命令
mount -a                                                             #重新读取</etc/fstab>文件进行挂载文件系统

mesg [y/n]                                                           #设置其他用户是否可以发信息到当前终端
write gjsy                                                           #给gjsy在线用户发送消息
wall helloword                                                       #给所有在线用户发送消息

logrotate -v /etc/logrotate.conf                                     #查看配置文件中需要日志轮替的文件
logrotate -f /etc/logrotate.conf                                     #强制运行配置文件中的日志轮替文件

[root@dwj ~]# echo '- - -' >/sys/class/scsi_host/host2/scan          #让系统识别新增加的磁盘

[root@dwj ~/Desktop]# zcat ntfs-3g.tgz                               #查看压缩包文件的内容
[root@dwj ~/Desktop]# tar -tvf ntfs-3g.tgz                           #不解压查看压缩包中的文件

[root@dwj ~/Desktop]# alias aliasname='command'                      #创建命令别名,如果是修改文件需要source生效

[root@dwj /usr/sbin]# sendmail diwenjie@gsafety.com                  #发送邮件命令
[root@dwj /usr/sbin]# mail                                           #该命令可以查看邮件发送状态，直接回车查看应答内容
[root@dwj /usr/sbin]# mailq                                          #该命令检测邮件发送Queue(/var/spool/mqueue)

[root@dwj ~/Desktop]# jobs                                           #查看后台运行程序，fg和bg把程序调到前台和后台执行
-l：显示进程号
-p：仅任务对应的显示进程号
-n：显示任务状态的变化
-r：仅输出运行状态（running）的任务
-s：仅输出停止状态（stoped）的任务
```
时间转换
```
把日期换算为时间戳
[root@dwj /opt]# echo $(($(date -d "2008/05/01" +%s)/86400))
把时间戳换算为日期
[root@dwj /opt]# date -d "1970/01/01 14000 days"
把秒数换算成日期
[root@dwj /opt]# date -d @1199116800
[root@dwj /opt]# date --date='@1199116800'
[root@dwj /opt]# date -d "10 days" #获取当前时间10天以后的日期
[root@dwj /opt]# date -d "164224 second ago" #获取164224秒之前的日期
```
Linux下查看系统启动时间和运行时间
```
[root@dwj /opt]# date -d "$(awk -F. '{print $1}' /proc/uptime) second ago" +"%Y-%m-%d %H:%M:%S"
[root@dwj /opt]# cat /proc/uptime| awk -F. '{run_days=$1 / 86400;run_hour=($1 % 86400)/3600;run_minute=($1 % 3600)/60;run_second=$1 % 60;printf("runtime: %d:%d:%d:%d",run_days,run_hour,run_minute,run_second)}'
[root@dwj /opt]# last reboot
[root@dwj /opt]# who -b
[root@dwj /opt]# w
```
Linux下查看进程相关信息
```
[root@dwj /opt]# pidof name                   #显示name进程pid
[root@dwj /opt]# ps -p pid -o parameter       #pid是进程号
```
parameter是以下内容之一:

name | desc
---|---
pid | 进程ID
tty | 终端
user | 用户
comm | 进程名
lstart | 开始时间
etime | 运行时间

terminal终端常用组合键含义
```
[root@dwj /opt]# ctrl + e                                #跳转到输入命令末尾
[root@dwj /opt]# ctrl + w                                #删除当前光标到命令行首的内容
[root@dwj /opt]# ctrl + a                                #跳转到输入命令行首
[root@dwj /opt]# ctrl + k                                #删除当前光标到命令行末的内容
[root@dwj /opt]# ctrl + r                                #进入history关键字查找 #(reverse-i-search)`':
[root@dwj /opt]# ctrl + t                                #互换输入命令末尾两个单词
[root@dwj /opt]# ctrl + p                                #向上逐条调出history历史命令
[root@dwj /opt]# ctrl + n                                #向下逐条调出history历史命令
[root@dwj /opt]# ctrl + o                                #执行终端已输入的命令
[root@dwj /opt]# ctrl + shift + =                        #放大terminal终端
[root@dwj /opt]# ctrl  + -                               #缩小terminal终端
[root@dwj /opt]# ctrl  + 0                               #还原原始大小
[root@dwj /opt]# ctrl  + z                               #让进程暂停运行进入后台
```
terminal终端变量

变量	| 含义
:--|---
!!  |  执行上一条shell命令
!$  |  上一条命令的最后一个参数
$0	|  当前脚本的文件名
$n	|  传递给脚本或函数的参数。n 是一个数字，表示第几个参数。例如，第一个参数是$1，第二个参数是$2
$#	|  传递给脚本或函数的参数个数
$*	|  传递给脚本或函数的所有参数
$@	|  传递给脚本或函数的所有参数。被双引号(" ")包含时，与 $* 稍有不同
$?	|  上个命令的退出状态，或函数的返回值；windows获取命令：echo %ERRORLEVEL%
$$	|  当前Shell进程ID。对于 Shell 脚本，就是这些脚本所在的进程ID

route命令结果Flags标记含义如下
```
U — 路由是活动的
H — 目标是一个主机
G — 路由指向网关
R — 恢复动态路由产生的表项
D — 由路由的后台程序动态地安装
M — 由路由的后台程序修改
! — 拒绝路由
```

释放缓存区内存的方法
```
清理pagecache（页面缓存）
[root@dwj ~]# echo 1 > /proc/sys/vm/drop_caches 或者 # sysctl -w vm.drop_caches=1
清理dentries（目录缓存）和inodes
[root@dwj ~]# echo 2 > /proc/sys/vm/drop_caches 或者 # sysctl -w vm.drop_caches=2
清理所有缓存
[root@dwj ~]# echo 3 > /proc/sys/vm/drop_caches 或者 # sysctl -w vm.drop_caches=3

要想永久释放缓存，需要在/etc/sysctl.conf文件中配置：vm.drop_caches=1/2/3
[root@dwj /opt]# sysctl -p  #该命令使设置生效
另外，可以使用sync命令来清理文件系统缓存，还会清理僵尸(zombie)对象和它们占用的内存
[root@dwj /opt]# sync
```

<font color=#FF0000 size=5> <p align="center">用文件动态增加swap分区大小</p></font>

首先查看磁盘分区使用情况,从空闲空间大的磁盘抽离部分作为swap分区
>[root@node1 ~]# df -h

例如抽取/目录下的磁盘空间，在该目录下创建swapimage文件夹
>[root@node1 ~]# mkdir /swapimage

进入swapimage文件夹
>[root@node1 ~]# cd /swapimage/

添加交换文件并设置大小为1G bs表示每次读写1M，count定义读写次数为1024
>[root@node1 swapimage]# dd if=/dev/zero of=/swapimage/swap bs=1M count=1024

查看磁盘空间可以发现根目录下少了1G,使用mkswap将/swapimage/swap文件格式化为虚拟内存文件格式
>[root@node1 swapimage]# mkswap /swapimage/swap

使用swapon命令，启用新增的1G交换空间
>[root@node1 swapimage]# swapon /swapimage/swap

查看内存情况，确认新增swap大小,如果成功执行设置系统自动加载
>[root@node1 swapimage]# free -m

修改/etc/rc.local文件，添加以下内容，使新增1G交换空间在系统启动生效
>[root@node1 swapimage]# vim /etc/rc.local  <br>
swapon /swapimage/swap

或者修改/etc/fstab文件，实现交换分区随系统启动生效
>[root@node1 swapimage]# vim /etc/fstab  <br>
/swapimage/swap swap  swap defaults 0 0

<font color=#FF0000 size=5> <p align="center">Linux 命令提示符显示全路径设置</p></font>

>[root@dwj ~]# vim etc/bashrc  #或者  <br>
>[root@dwj ~]# vim ~/.bash_profile
```
在最后加上下面语句，其中\w 是小写
export PS1='[\u@\h \w]\$'
```
最后执行
>[root@dwj ~]# source ~/.bash_profile

<font color=#FF0000 size=5> <p align="center">chmod</p></font>

一个文件都有一个所有者, 表示该文件是谁创建的. 同时, 该文件还有一个组编号, 表示该文件所属的组, 一般为文件所有者所属的组
如果是一个可执行文件, 那么在执行时, 一般该文件只拥有调用该文件的用户具有的权限. 而setuid, setgid可以来改变这种设置
```
setuid: 设置使文件在执行阶段具有文件所有者的权限，/usr/bin/passwd是典型，一般用户执行该文件, 执行过程中可以获得root权
setgid: 该权限只对目录有效，目录被设置该位后, 任何用户在此目录下创建的文件都具有和该目录相同的所属组
sticky bit: 该位是文件粘滞位，一个文件是否可以被某用户删除, 主要取决于该文件所属的组是否对该用户具有写权限
```
如果没有写权限, 则这个目录下的所有文件都不能被删除, 同时也不能添加新的文件。如果希望用户能够添加文件，但同时不能删除文件
则可以对文件使用sticky bit位，设置该位后，就算用户对目录具有写权限，也不能删除该文件

操作标志命令与操作文件权限的命令是一样的，都是chmod，第一种操作方法
```
chmod u+s temp           为temp文件加上setuid标志. (setuid 只对文件有效)
chmod g+s tempdir        为tempdir目录加上setgid标志 (setgid 只对目录有效)
chmod o+t temp           为temp文件加上sticky标志 (sticky只对文件有效)
```
第二种是采用八进制方式，对一般文件通过三组八进制数字来置标志，如666等，如果设置这些特殊标志
则在这组数字之外外加一组八进制数字，如4666等，如果有这些标志,，则会在原来的执行标志位置上显示
```
setuid位, 如果该位为1, 则表示设置setuid
setgid位, 如果该位为1, 则表示设置setgid
sticky位, 如果该位为1, 则表示设置sticky
rwsrw-r–      表示有setuid标志
rwxrwsrw-     表示有setgid标志
rwxrw-rwt     表示有sticky标志
```
那么原来的执行标志x到哪里去了呢? 系统是这样规定的, 如果本来在该位上有x, 则这些特殊标志显示为小写字母，否则, 显示为大写字母

常用操作：找出所有危险的目录（设置目录所有人可读写却没有设置sticky位的目录）
>[root@dwj opt]# find / -perm -0007 -type d

找出所有设置了suid的文件
>[root@dwj opt]# find / -perm -4000 -type f

设置目录或文件粘滞位，给目录设置粘滞位可以阻止一个用户随意删除可写目录下面其他人的文件
>[root@dwj opt]# chmod +t  filename

<font color=#FF0000 size=5> <p align="center">内核参数和环境变量修改</p></font>

>[root@dwj ~]# ulimit -a              #显示系统部分内核参数配置信息  <br>
>[root@dwj ~]# ulimit -SHn 102400     #修改当前session有效文件描述符的限制
>[root@dwj ~]# ulimit -c unlimited    #开启program core dump的支持

长期开启core dump功能需要修改/etc/profile，在末尾加上命令：
>ulimit -c unlimited >/dev/null 2>&1

core dump文件的生成方式可以修改 /etc/sysctl.conf 文件，加入以下内容：
```
kernel.core_uses_pid = 1     //Appends the coring processes PID to the core file name.
kernel.core_pattern = /tmp/core-%e-%s-%u-%g-%p-%t
/*
When the application terminates abnormally, a core file should appear in the /tmp. The kernel.core_pattern sysctl controls exact location of core file. You can define the core file name with the following template whih can contain % specifiers which are substituted by the following values when a core file is created:
%% - A single % character
%p - PID of dumped process
%u - real UID of dumped process
%g - real GID of dumped process
%s - number of signal causing dump
%t - time of dump (seconds since 0:00h, 1 Jan 1970)
%h - hostname (same as ’nodename’ returned by uname(2))
%e - executable filename
*/
fs.suid_dumpable = 2         // Make sure you get core dumps for setuid programs.
```

永久变更需要修改/etc/security/limits.conf 文件，如下：
```
vi /etc/security/limits.conf
* hard nofile 65535
* soft nofile 65535
```
保存退出后重新登录，其最大文件描述符就被永久更改了

参数名 | 参数含义
--- | ---
-a | 列出所有当前资源极限
-c | 设置core文件的最大值.单位:blocks
-d | 设置一个进程的数据段的最大值.单位:kbytes
-f | Shell 创建文件的文件大小的最大值，单位：blocks
-h | 指定设置某个给定资源的硬极限。如果用户拥有 root 用户权限，可以增大硬极限。任何用户均可减少硬极限
-l | 可以锁住的物理内存的最大值
-m | 可以使用的常驻内存的最大值,单位：kbytes
-n | 每个进程可以同时打开的最大文件数
-p | 设置管道的最大值，单位为block，1block=512bytes
-s | 指定堆栈的最大值：单位：kbytes
-S | 指定为给定的资源设置软极限。软极限可增大到硬极限的值。如果 -H 和 -S 标志均未指定，极限适用于以上二者
-t | 指定每个进程所使用的秒数,单位：seconds
-u | 可以运行的最大并发进程数
-v | Shell可使用的最大的虚拟内存，单位：kbytes

<font color=#FF0000 size=5> <p align="center">dd命令</p></font>

```
linux中特殊的设备(/dev/zero,/dev/null,/dev/unrandom,/dev/random)
dev/null看作"黑洞"。 它等价于一个只写文件。所有写入它的内容都会永远丢失。 而尝试从它那儿读取内容则什么也读不到。所以在dd命令中当为了测试某个磁盘的读性能时候，就可以将of指定为/dev/null

dev/zero文件产生连续不断的null的流(二进制的零流，而不是ASCII型的)。写入它的输出会丢失不见，主要是用来创建一个指定长度用于初始化的空文件.另一个应用是用零去填充一个指定大小的文件，用dd命令为了测试磁盘写性能时候，可以将if指定为/dev/zero这样，就相当源源不断的向测试设备中写入数据

dev/random和/dev/urandom是unix系统提供的产生随机数的设备，区别在于/dev/urandom生成的速度比/dev/random快。如果不能立即生成随机串，/dev/random会一直阻塞，有时会非常耗费CPU;/dev/urandom则会根据其他值立即生成一个随机串，不会阻塞。/dev/urandom生成的随机值没有/dev/random随机。大多数情况下，选用/dev/urandom

常见的DD命令，这四条DD命令区别在于内存中写缓存的处理方式(执行DD命令测试硬盘IO性能，对硬盘的损害很大)

[root@dwj ~]# dd bs=64k count=4k if=/dev/zero of=test
dd默认的方式不包括“同步(sync)”命令。也就是说，dd命令完成前并没有让系统真正把文件写到磁盘上。所以以上命令只是单纯地把这128MB的数据读到内存缓冲当中(写缓存[write cache])。所以会得到一个超级快的速度。因为dd执行的只是读取速度，直到dd完成后系统才开始真正往磁盘上写数据，但这个速度是看不到了

[root@dwj ~]# dd bs=64k count=4k if=/dev/zero of=test; sync
分号隔开的只是先后两个独立的命令。当sync命令准备开始往磁盘上真正写入数据的时候，前面dd命令已经把错误的“写入速度”值显示在屏幕上了。所以还是得不到真正的写入速度

[root@dwj ~]# dd bs=64k count=4k if=/dev/zero of=test conv=fdatasync
dd命令执行到最后会真正执行一次“同步(sync)”操作，所以这时候得到的是读取这128M数据到内存并写入到磁盘上所需的时间，符合实际使用结果

[root@dwj ~]# dd bs=64k count=4k if=/dev/zero of=test oflag=dsync
dd在执行时每次都会进行同步写入操作。也就是说，这条命令每次读取64k后就要先把这64k写入磁盘，然后再读取下面这64k，一共重复128次。这可能是最慢的一种方式了，因为基本上没有用到写缓存(write cache)
```

二、dd应用实例:
1.将本地的/dev/hdb整盘备份到/dev/hdd
>[root@dwj ~]# dd if=/dev/hdb of=/dev/hdd

2.将/dev/hdb全盘数据备份到指定路径的image文件
>[root@dwj ~]# dd if=/dev/hdb of=/root/image

3.将备份文件恢复到指定盘
>[root@dwj ~]# dd if=/root/image of=/dev/hdb

4.备份/dev/hdb全盘数据，并利用gzip工具进行压缩，保存到指定路径
>[root@dwj ~]# dd if=/dev/hdb | gzip > /root/image.gz

5.将压缩的备份文件恢复到指定盘
>[root@dwj ~]# gzip -dc /root/image.gz | dd of=/dev/hdb

6.备份与恢复MBR   <br>
备份磁盘开始的512个字节大小的MBR信息到指定文件：
>[root@dwj ~]# dd if=/dev/hda of=/root/image count=1 bs=512
   count=1指仅拷贝一个块；bs=512指块大小为512个字节。

备份恢复(将备份的MBR信息写到磁盘开始部分)：
>[root@dwj ~]# dd if=/root/image of=/dev/had

备份软盘
>[root@dwj ~]# dd if=/dev/fd0 of=disk.img count=1 bs=1440k (即块大小为1.44M)

8.拷贝内存内容到硬盘
>[root@dwj ~]# dd if=/dev/mem of=/root/mem.bin bs=1024 (指定块大小为1k)

9.拷贝光盘内容到指定文件夹，并保存为cd.iso文件
>[root@dwj ~]# dd if=/dev/cdrom(hdc) of=/root/cd.iso

10.销毁磁盘数据
>[root@dwj ~]# dd if=/dev/urandom of=/dev/hda1
注意：利用随机的数据填充硬盘，在某些必要的场合可以用来销毁数据

12.测试硬盘的读写速度
>[root@dwj ~]# dd if=/dev/zero bs=1024 count=1000000 of=/root/1Gb.file
>[root@dwj ~]# dd if=/root/1Gb.file bs=64k | dd of=/dev/null
通过以上两个命令输出的命令执行时间，可以计算出硬盘的读、写速度

13.确定硬盘的最佳块大小：
```
dd if=/dev/zero bs=1024 count=1000000 of=/root/1Gb.file
dd if=/dev/zero bs=2048 count=500000 of=/root/1Gb.file
dd if=/dev/zero bs=4096 count=250000 of=/root/1Gb.file
dd if=/dev/zero bs=8192 count=125000 of=/root/1Gb.file
通过比较以上命令输出中所显示的命令执行时间，即可确定系统最佳的块大小
```

14.修复硬盘：
>[root@dwj ~]# dd if=/dev/sda of=/dev/sda
当硬盘较长时间(一年以上)放置不使用后，磁盘上会产生magnetic flux point，当磁头读到这些区域时会遇到困难，并可能导致I/O错误。当这种情况影响到硬盘的第一个扇区时，可能导致硬盘报废。上边的命令有可能使这些数据起死回生。并且这个过程是安全、高效的

15.利用netcat远程备份
>[root@dwj ~]# dd if=/dev/hda bs=16065b | netcat < targethost-IP > 1234

在源主机上执行此命令备份/dev/hda
>[root@dwj ~]# netcat -l -p 1234 | dd of=/dev/hdc bs=16065b

在目的主机上执行此命令来接收数据并写入/dev/hdc
>[root@dwj ~]# netcat -l -p 1234 | bzip2 > partition.img <br>
>[root@dwj ~]# netcat -l -p 1234 | gzip > partition.img
以上两条指令是目的主机指令的变化分别采用bzip2、gzip对数据进行压缩，并将备份文件保存在当前目录。

将一个很大的视频文件中的第i个字节的值改成0x41（也就是大写字母A的ASCII值）
>[root@dwj ~]# echo A | dd of=bigfile seek=$i bs=1 count=1 conv=notrunc

三、/dev/null和/dev/zero的区别
/dev/null，外号叫无底洞，你可以向它输出任何数据，它通吃，并且不会撑着
/dev/zero，是一个输入设备，你可你用它来初始化文件。该设备无穷尽地提供0
/dev/null, 它是空设备，任何写入它的输出都会被抛弃。如果不想让消息以标准输出显示或写入文件，那么可以将消息重定向到位桶

把/dev/null看作"黑洞"， 它等价于一个只写文件，所有写入它的内容都会永远丢失.，而尝试从它那儿读取内容则什么也读不到。然而， /dev/null对命令行和脚本都非常的有用

l.禁止标准输出
>[root@dwj ~]# cat $filename >/dev/null

2.隐藏cookie而不再使用
特别适合处理这些由商业Web站点发送的讨厌的"cookies"
```
if [ -f ~/.netscape/cookies ] then  #如果存在则删除
rm -f ~/.netscape/cookies
fi
ln -s /dev/null ~/.netscape/cookies
现在所有的cookies都会丢入黑洞而不会保存在磁盘上了
```
3.使用/dev/zero
像/dev/null一样， /dev/zero也是一个伪文件，但它实际上产生连续不断的null的流（二进制的零流，而不是ASCII型的）。 写入它的输出会丢失不见，而从/dev/zero读出一连串的null也比较困难， 虽然这也能通过od或一个十六进制编辑器来做到。 /dev/zero主要的用处是用来创建一个指定长度用于初始化的空文件，就像临时交换文件。
用/dev/zero创建一个交换临时文件

<font color=#FF0000 size=5> <p align="center">系统时间</p></font>

```
date -R                                    #查看时区信息
date -s 时间字符串                          #修改操作系统时间
date -s 2012-08-02                         #只修改系统的日期，不修改时间（时分秒）
date -s 10:08:00                           #只修改时间不修改日期
date -s "2012-05-18 04:53:00"              #同时修改日期和时间
```
上述只是修改了linux的系统时间，CMOS中的时间可能还没有改变，需要使用 clock -w 把当前系统时间写入到CMOS中
系统时间和CMOS时间的关系。系统时间是由linux操作系统来维护的；CMOS时间是CMOS芯片保存的时间。
系统启动时，操作系统将从CMOS读出时间记录为系统时间，同时操作系统也会自动每隔一段时间将系统时间写入CMOS中。
如果使用date命令修改系统时间后马上重启电脑，操作系统还没有将系统时间同步到CMOS，这样开机后就还是没有修改前的时间了，
所以为了保险起见，最还还是手动使用命令 clock 将系统时间同步到CMOS中

/usr/share/zoneinfo目录下基本涵盖了大部分的国家和城市的时区

查看每个 timezone 当前的时间可以用zdump命令
```
[root@dwj zoneinfo]# zdump Hongkong
结果：Hongkong  Thu Mar 30 16:47:31 2017 HKT
[root@dwj zoneinfo]# zdump /etc/localtime
[root@dwj zoneinfo]# zdump /etc/timezone
```
修改当前系统的时区方法，修改后的值保存在“/etc/sysconfig/clock”文件中

1.修改/etc/localtime文件
>[root@dwj zoneinfo]# vim /etc/localtime

2.可以在/usr/share/zoneinfo下找到我们的time zone文件然后做个symbolic link
>[root@dwj zoneinfo]# ln -sf /usr/share/zoneinfo/posix/Asia/Shanghai /etc/localtime

3.使用tzselect命令进行时区选择，Redhat没有修改成功
>[root@dwj zoneinfo]# tzselect

Real Time Clock(RTC) and System Clock（硬件时间时钟和系统时钟）
```
[root@dwj zoneinfo]# hwclock --show              #查看机器上的硬件时间
[root@dwj zoneinfo]# hwclock --hctosys           #硬件时间设置成系统时间
[root@dwj zoneinfo]# hwclock --systohc           #系统时间设置成硬件时间
[root@dwj zoneinfo]# hwclock -w                  #同步硬件时间
[root@dwj zoneinfo]# cat /etc/adjtime            #hwclock -w命令后秒数存放的文件
```

<font color=#FF0000 size=5> <p align="center">邮箱默认端口</p></font>

```
SMTP,  简单的邮件传输协议，TCP 25端口，加密时使用TCP 465端口
POP3， 第三版邮局协议，TCP 110端口，加密时使用995端口
IMAP4，第四版互联网消息访问协议，TCP 143端口，加密时使用993端口
MUA（邮件用户代理），MTA（邮件传输代理）， MDA（又见分发代理），MRA（邮件检索代理）
```

<font color=#FF0000 size=5> <p align="center">/etc/issue文件参数</p></font>

当前系统启动后，登录前的提示信息为配置参数如下：

参数名 | 含义
---|---
\d | 显示当前日期
\l | 显示虚拟控制台号
\m | 显示机器类型，即 CPU 架构，如 i386 或 x86_64 等(相当于 uname -m)
\n | 显示主机的网络名(相当于 uname -n)
\o | 显示域名
\r | 显示 Kernel 内核版本号(相当于 uname -r)
\t | 显示当前时间
\s | 显示当前操作系统名称
\u | 显示当前登录用户的编号
\U | 显示当前登录用户的编号和用户
\v | 显示当前操作系统的版本日期

注意：只会在普通登录时才会显示，远程 ssh 连接的时候并不会显示此信息

<font color=#FF0000 size=5> <p align="center">ssh登录欢迎信息设置</p></font>

当输入用户后，在弹出输入密码提示之前显示欢迎信息
>[root@dwj ~]# /etc/ssh/sshd_config     #在其中添加对应的Banner文件路径  <br>
Banner /etc/ssh/banner

然后再创建 /etc/ssh/banner 文件，文件内容即为输入用户名后的欢迎信息

修改完后，执行如下命令重新加载,再次登录即可显示欢迎信息
>[root@dwj ~]# service sshd reload

注意：此信息只在 ssh 输入用户名后显示，在普通登录输入用户名后不显示

<font color=#FF0000 size=5> <p align="center">NFS参数</p></font>

```
exportfs #命令用来管理当前NFS共享的文件系统列表
-a 打开或取消所有目录共享
-o options,...指定一列共享选项，与exports(5)中讲到的类似
-i 忽略/etc/exports文件，从而只使用默认的和命令行指定的选项
-r 重新共享所有目录。它使/var/lib/nfs/xtab和/etc/exports同步。它将/etc/exports中已删除的条目从/var/lib/nfs/xtab中删除，将内核共享表中任何不再有效的条目移除
-u 取消一个或多个目录的共享
-f 在“新”模式下，刷新内核共享表之外的任何东西。任何活动的客户程序将在它们的下次请求中得到mountd添加的新的共享条目
-v 输出详细信息。当共享或者取消共享时，显示在做什么。显示当前共享列表的时候，同时显示共享的选项
```

<font color=#FF0000 size=5> <p align="center">用户添加-删除</p></font>

>[root@dwj ~]# useradd -d /usr/dwj -m dwj
```
-d 　指定用户根目录
-s 　指定用户登陆shell
-u　 指定用户uid
-g 　指定用户所属主组
-G 　指定用户所属附属组
-m   创建指定用户的根目录
```

命令usermod修改一个用户的各项信息；格式：usermod <参数> <用户名>
```
-l 　 修改用户名，对应的是/etc/passwd的第一栏
-c    更改/etc/passwd第5栏用户信息说明的部分，后面接描述信息，可以使用chfn命令替代
-e    更改/etc/shadow的第8栏账号的失效日期，后面接日期参数格式为 MM/DD/YY 或 YYYY-MM-DD
-f    更改/etc/shadow的第7栏账号过期宽限时间部分，当后面接的值为0时，账号立即失效，为-1时关闭此功能
-u    修改uid，对应的是/etc/passwd的和3栏，此 UID不能与目前系统中已经存在的UID相同
-s    后面接shell的实际文件，即 /bin/bash,/bin/csh之类，可以使用 chsh 命令替代
-g    修改用户组，id对应/etc/passwd的第4栏
-G    修改用户附属组，修改的是/etc/group
-L    锁定用户，暂将用户的密码冻结，即更改/etc/shadow的密码栏，在其前面加上!
-U    解锁用户，即去掉其/etc/shadow密码栏前面的!
-d    更改/etc/passwd第6栏用户的home目录部分，如果再加上-m参数（只与-d配合）
      则会将现有home目录的地址重命名为新的地址，如原来没有指定，则为账号新建一个指定的home目录地址
```
>[root@dwj ~]# userdel dwj        #删除用户  <br>
>[root@dwj ~]# passwd dwj         #修改用户密码

<font color=#FF0000 size=5> <p align="center">vim注释和删除注释</p></font>

1.多行注释：
```
a. 按下Ctrl + v，进入列模式;
b. 在行首选择需要注释的行;
c. 按下shift+i进入插入模式；
d. 然后输入注释符#或者//
e. 按下Esc键
```
2.删除多行注释：
```
a. 按下Ctrl + v, 进入列模式;
b. 选定要取消的注释符;
c. 按下x或者d
d. 按下Esc键
```
3.在命令管理行下注释
```
添加注释 :起始行号,结束行号s/^/注释符/g
取消注释：:起始行号,结束行号s/^注释符//g
例子：
在10-20行添加注释 :10,50s#^#//#g
在10-20行删除注释 :10,20s#^//##g
```
4.代码注释
```
if false ; then
 ....被注释的多行内容
fi
```
5.其他
```
:<< ！
....被注释的多行内容
!
```
其他常用技巧
```
ctrl+g   #显示当前编辑的文件名
d+G      #删除光标到末尾全部行
:noh     #取消高亮显示
:e!      #还原编辑文件到最初状态
set ls=2 #总是显示状态栏 ls : laststatus
~/.viminfo   #vim操作日志
vim -O /etc/passwd /opt/gjsy.txt    #左右分屏显示,使用ctrl+w两次进行左右光标切换
```

<font color=#FF0000 size=5> <p align="center">SELinux设置</p></font>

查看SELinux状态：
>[root@dwj ~]# /usr/sbin/sestatus -v    #SELinux status: enabled即为开启状态  <br>
>[root@dwj ~]# getenforce               #也可以用这个命令检查

关闭SELinux：
```
--临时关闭(不用重启机器)
[root@dwj ~]# setenforce 0   #设置SELinux 成为permissive模式
[root@dwj ~]# setenforce 1   #设置SELinux 成为enforcing模式

--修改配置文件需要重启机器
修改/etc/selinux/config 文件
将SELINUX=enforcing改为SELINUX=disabled
重启机器即可
```

<font color=#FF0000 size=5> <p align="center">/usr/share/man手册页</p></font>

```
/usr/share/<mandir>/<locale> 细述整个系统中手册页的组织：
Man1：用户程序手册。本节中包含了描述公共访问的命令的手册页面。用户需要使用的绝大多数程序文档位于这里
Man2：系统调用。这一节描述了所有的系统调用（对内核执行操作的请求）
Man3：库函数和子例程
Man4：特殊文件，包括/dev下能找到的设备文件和内核对网络协议的支持接口
Man5：文件格式，这包括各种头文件、程序输出文件和系统文件
Man6：游戏。这一节是游戏、演示和通常很琐碎的程序的文档
Man7：各种难以归类的手册页，Troff和其他文本处理宏包（的手册）放在这里
Man8：系统管理员用来进行系统操作和维护的系统管理程序的文档放在这里。有些程序有时也对普通用户有用
```

<font color=#FF0000 size=5> <p align="center">Top</p></font>

>[root@dwj ~]# top   #运行命令

![image](https://github.com/dwjlw1314/DWJ-PROJECT/raw/master/PictureSource/4.1.2.png)

<font color=#FF0000>一. top前五行统计信息</font>

第1行是任务队列信息，同 uptime 命令的执行结果一样，其内容如下:

12:38:33 | 当前时间
:--|:--
up 50days  |  系统运行时间，格式为时:分
1 user     |  当前登录用户数
load average: 0.6, 0.6, 0.4  |  系统负载，即任务队列的平均长度。 三个数值分别为 <br>1分钟、5分钟、15分钟前到现在的平均值

第2、3行为进程和CPU的信息，当有多个CPU时，这些内容可能会超过两行,内容如下:

Tasks: 29 total | 进程总数
:--|:--
1 running       | 正在运行的进程数
28 sleeping     | 睡眠的进程数
0 stopped       | 停止的进程数
0 zombie        | 僵尸进程数
cpu(s): 0.3% us | 用户空间占用CPU百分比
1.0% sy         | 内核空间占用CPU百分比
0.0% ni         | 用户进程空间内改变过优先级的进程占用CPU百分比
98.7% id        | 空闲CPU百分比
0.0% wa         | 等待输入输出的CPU时间百分比
hi,si,st        | 硬件中断,软件中断,实时

第4、5行为内存信息，内容如下:

Mem: 1912k total    | 物理内存总量
:--|:--
173656k used        | 使用的物理内存总量
17616k free         | 空闲内存总量
22052k buffers      | 用作内核缓存的内存量
Swap: 1927k total   | 交换区总量
0k used             | 使用的交换区总量
192772k free        | 空闲交换区总量
123988k cached      | 缓冲的交换区总量。 内存中的内容被换出到交换区，而后又被换入到内存，但使用过的交换区尚未被覆盖， <br>该数值即为这些内容已存在于内存中的交换区的大小。相应的内存再次被换出时可不必再对交换区写入

<font color=#FF0000>二. 进程信息</font>

列名|含义
:--|:--
PID      | 进程id
PPID     | 父进程id
RUSER    | Real user name
UID      | 进程所有者的用户id
USER     | 进程所有者的用户名
GROUP    | 进程所有者的组名
TTY      | 启动进程的终端名。不是从终端启动的进程则显示为 ?
PR       | 优先级
NI       | nice值。负值表示高优先级，正值表示低优先级
P        | 最后使用的CPU，仅在多CPU环境下有意义
%CPU     | 上次更新到现在的CPU时间占用百分比
TIME     | 进程使用的CPU时间总计，单位秒
TIME+    | 进程使用的CPU时间总计，单位1/100秒
%MEM     | 进程使用的物理内存百分比
VIRT     | 进程使用的虚拟内存总量，单位kb。VIRT=SWAP+RES
SWAP     | 进程使用的虚拟内存中，被换出的大小，单位kb。
RES      | 进程使用的、未被换出的物理内存大小，单位kb。RES=CODE+DATA
CODE     | 可执行代码占用的物理内存大小，单位kb
DATA     | 可执行代码以外的部分(数据段+栈)占用的物理内存大小，单位kb
SHR      | 共享内存大小，单位kb
nFLT     | 页面错误次数
nDRT     | 最后一次写入到现在，被修改过的页面数
S        | 进程状态: D=不可中断的睡眠状态; R=运行; S=睡眠; T=跟踪/停止; Z=僵尸进程
COMMAND  | 命令名/命令行
WCHAN    | 若该进程在睡眠，则显示睡眠中的系统函数名
Flags    | 任务标志，参考 sched.h

<font color=#FF0000>三. 快捷键更改显示内容</font>

1.更改显示内容通过f键可以选择显示的内容,按a-z即可显示或隐藏对应的列，最后按回车键确定

2.按o键可以改变列的显示顺序   <br>
按小写的a-z可以将相应的列向右移动，而大写的A-Z可以将相应的列向左移动。最后按回车键确定  <br>
按大写的F或O键，然后按a-z可以将进程按照相应的列进行排序。而大写的R键可以将当前的排序倒转。设置完按回车返回界面

3.命令使用；命令格式：top [-] [d] [p] [q] [c] [C] [S] [n]
```
参数说明：
d：指定每两次屏幕信息刷新之间的时间间隔。当然用户可以使用s交互命令来改变之
p：通过指定监控进程ID来仅仅监控某个进程的状态
q：该选项将使top没有任何延迟的进行刷新
S：指定累计模式
s：使top命令在安全模式中运行。这将去除交互命令所带来的潜在危险
i：使top不显示任何闲置或者僵死进程
c：显示整个命令行而不只是显示命令名
```
4.top命令的显示窗口交互命令
```
k：终止一个进程。系统将提示用户输入需要终止的进程PID，以及需要发送给该进程什么样的信号。一般的终止进程可以使用15信号；如果不能正常结束那就使用信号9强制结束该进程。默认值是信号15。在安全模式中此命令被屏蔽
i：忽略闲置和僵死进程。这是一个开关式命令
q：退出程序
r：重新安排一个进程的优先级别。系统提示用户输入需要改变的进程PID以及需要设置的进程优先级值。输入一个正值将使优先级降低，反之则可以使该进程拥有更高的优先权。默认值是10
S：切换到累计模式
s: 改变两次刷新之间的延迟时间。系统将提示用户输入新的时间，单位为s。如果有小数，就换算成ms。输入0值则系统将不断刷新，默认值是5s。需要注意的是如果设置太小的时间，很可能会引起不断刷新，从而根本来不及看清显示的情况，而且系统负载也会大大增加
f或者F: 从当前显示中添加或者删除项目
o或者O: 改变显示项目的顺序
l: 切换显示平均负载和启动时间信息。即显示影藏第一行
m：切换显示内存信息。即显示影藏内存行
t：切换显示进程和CPU状态信息。即显示影藏CPU行
c：切换显示命令名称和完整命令行。 显示完整的命令。 这个功能很有用
M: 根据驻留内存大小进行排序
P：根据CPU使用百分比大小进行排序
T：根据时间/累计时间进行排序
W：将当前设置写入~/.toprc文件中。这是写top配置文件的推荐方法
```

<font color=#FF0000 size=5> <p align="center">vmstat</p></font>

vmstat命令是Linux监控工具，可以展现给定时间间隔的服务器的状态值,包括服务器的CPU使用率，内存使用，虚拟内存交换，IO读写情况
相比top，可以看到整个机器的CPU,内存,IO的使用情况，而不是单单看到各个进程的CPU使用率和内存使用率(使用场景不一样)
一般vmstat工具的使用是通过两个数字参数来完成的，第一个参数是采样的时间间隔数，单位是秒，第二个参数是采样的次数

>[root@dwj ~]# vmstat 2 1   #运行命令，参数2表示每个两秒采集一次服务器状态，1表示只采集一次
```
procs ----------------memory---------------swap------------io----------system----------cpu-------
 r     b    swpd    free     buff     cache    si   so    bi    bo   in   cs  us  sy   id  wa  st
 0     0     0     836944   142284    472868    0    0    17     5   40   71   0  0   99  0   0
参数含义：
r 表示运行队列(就是说多少个进程真的分配到CPU)，当这个值超过了CPU数目，就会出现CPU瓶颈了。这个也和top的负载有关系，一般负载超过了3就比较高，超过了5就高，超过了10就不正常了，服务器的状态很危险。top的负载类似每秒的运行队列。如果运行队列过大，表示你的CPU很繁忙，一般会造成CPU使用率很高
b 表示阻塞的进程
swpd 虚拟内存已使用的大小，如果大于0，表示你的机器物理内存不足了，如果不是程序内存泄露的原因，那么你该升级内存了或者把耗内存的任务迁移到其他机器
free 空闲的物理内存的大小
buff Linux/Unix系统是用来存储，目录里面有什么内容，权限等的缓存
cache 直接用来记忆我们打开的文件,给文件做缓冲，把空闲的物理内存的一部分拿来做文件和目录的缓存，是为了提高程序执行的性能，当程序使用内存时，buffer/cached会很快地被使用
si 每秒从磁盘读入虚拟内存的大小，如果这个值大于0，表示物理内存不够用或者内存泄露了，要查找耗内存进程解决掉
so 每秒虚拟内存写入磁盘的大小，同上
bi 块设备每秒接收的块数量，这里的块设备是指系统上所有的磁盘和其他块设备，默认块大小是1024byte
bo 块设备每秒发送的块数量，例如我们读取文件，bo就要大于0。bi和bo一般都要接近0，不然就是IO过于频繁，需要调整
in 每秒CPU的中断次数，包括时间中断
cs 每秒上下文切换次数，例如我们调用系统函数，就要进行上下文切换，线程的切换，也要进程上下文切换，这个值要越小越好，太大了，要考虑调低线程或者进程的数目，例如在apache和nginx这种web服务器中，我们一般做性能测试时会进行几千并发甚至几万并发的测试，选择web服务器的进程可以由进程或者线程的峰值一直下调，压测，直到cs到一个比较小的值，这个进程和线程数就是比较合适的值了。系统调用也是，每次调用系统函数，我们的代码就会进入内核空间，导致上下文切换，这个是很耗资源，也要尽量避免频繁调用系统函数。上下文切换次数过多表示你的CPU大部分浪费在上下文切换，导致CPU干正经事的时间少了，CPU没有充分利用，是不可取的
us 用户CPU时间，我曾经在一个做加密解密很频繁的服务器上，可以看到us接近100,r运行队列达到80
sy 系统CPU时间，如果太高，表示系统调用时间长，例如是IO操作频繁
id 空闲CPU时间，一般来说，id + us + sy = 100,一般我认为id是空闲CPU使用率，us是用户CPU使用率，sy是系统CPU使用率
wa IO等待时间百分比 wa的值高时，说明IO等待比较严重，这可能由于磁盘大量作随机访问造成，也有可能磁盘出现瓶颈(块操作)
```

<font color=#FF0000 size=5> <p align="center">mpstat</p></font>

mpstat是Multiprocessor Statistics的缩写，是实时系统监控工具。其报告与CPU的一些统计信息，这些信息存放在/proc/stat文件中。在多CPUs系统里，其不但能查看所有CPU的平均状况信息，而且能够查看特定CPU的信息。mpstat最大的特点是：可以查看多核心cpu中每个计算核心的统计数据；而类似工具vmstat只能查看系统整体cpu情况

mpstat [-P {|ALL}] [internal [count]]      #参数解释
```
-P {|ALL} 表示监控哪个CPU， cpu在[0,cpu个数-1]中取值
internal 相邻的两次采样的间隔时间
count 采样的次数，count只能和delay一起使用
当没有参数时，mpstat则显示系统启动以后所有信息的平均值。
有interval时，第一行的信息自系统启动以来的平均信息，从第二行开始，输出为前一个interval时间段的平均信息
```
[root@dwj ~]# mpstat -P ALL 2
![image](https://github.com/dwjlw1314/DWJ-PROJECT/raw/master/PictureSource/4.1.3.png)
```
%user       在internal时间段里，用户态的CPU时间(%)，不包含nice值为负进程  (usr/total)*100
%nice       在internal时间段里，nice值为负进程的CPU时间(%)   (nice/total)*100
%sys        在internal时间段里，内核时间(%)       (system/total)*100
%iowait     在internal时间段里，硬盘IO等待时间(%) (iowait/total)*100
%irq        在internal时间段里，硬中断时间(%)     (irq/total)*100
%soft       在internal时间段里，软中断时间(%)     (softirq/total)*100
%idle       在internal时间段里，CPU除去等待磁盘IO操作外的因为任何原因而空闲的时间闲置时间(%) (idle/total)*100
```

<font color=#FF0000 size=5> <p align="center">pidstat</p></font>

pidstat主要用于监控全部或指定进程占用系统资源的情况，如CPU，内存、设备IO、任务切换、线程等。
执行pidstat，将输出系统启动后所有活动进程的cpu统计信息

[root@dwj ~]# pidstat -r -p 3512
![image](https://github.com/dwjlw1314/DWJ-PROJECT/raw/master/PictureSource/4.1.4.png)
```
以上各列输出的含义如下：
minflt/s: 每秒次缺页错误次数(minor page faults)，次缺页错误次数意即虚拟内存地址映射成物理内存地址产生的page fault次数
majflt/s: 每秒主缺页错误次数(major page faults)，当虚拟内存地址映射成物理内存地址时，相应的page在swap中，内存紧张时产生
VSZ:      该进程使用的虚拟内存(以kB为单位)
RSS:      该进程使用的物理内存(以kB为单位)
%MEM:     该进程使用内存的百分比
Command:  拉起进程对应的命令
```
[root@dwj ~]# pidstat -d -p 3512 1 2
![image](https://github.com/dwjlw1314/DWJ-PROJECT/raw/master/PictureSource/4.1.5.png)
```
输出信息含义：
kB_rd/s: 每秒进程从磁盘读取的数据量(以kB为单位)
kB_wr/s: 每秒进程向磁盘写的数据量(以kB为单位)
Command: 拉起进程对应的命令
```

<font color=#FF0000 size=5> <p align="center">iostat</p></font>

iostat命令查看IO请求下发情况、系统IO处理能力，以及命令执行结果中各字段的含义详解

1.不加选项执行iostat，我们先来看直接执行iostat的输出结果
![image](https://github.com/dwjlw1314/DWJ-PROJECT/raw/master/PictureSource/4.1.6.png)
```
最上面指示系统版本、主机名和日期的一行外
avg-cpu: 总体cpu使用情况统计信息，对于多核cpu，这里为所有cpu的平均值，主要看iowait的值，它指示cpu用于等待io请求完成的时间
Device: 各磁盘设备的IO统计信息，各列含义如下：
    Device:     以sdX形式显示的设备名称
    tps:        每秒进程下发的IO读、写请求数量
    Blk_read/s: 每秒读扇区数量(一扇区为512bytes)
    Blk_wrtn/s: 每秒写扇区数量
    Blk_read:   取样时间间隔内读扇区总数量
    Blk_wrtn:   取样时间间隔内写扇区总数量
可以使用-c选项单独显示avg-cpu部分的结果，使用-d选项单独显示Device部分的信息
```

2.指定采样时间间隔与采样次数,iostat interval [count] 形式指定iostat命令的采样间隔和采样次数
>[root@dwj nicstat-1.95]# iostat -d 1 3

3.更详细的io统计信息可以使用-x选项，在分析io瓶颈时，一般都会开启-x选项

![image](https://github.com/dwjlw1314/DWJ-PROJECT/raw/master/PictureSource/4.1.7.png)
```
以上各列的含义如下：
rrqm/s: 每秒对该设备的读请求被合并次数，文件系统会对读取同块(block)的请求进行合并
wrqm/s: 每秒对该设备的写请求被合并次数
r/s: 每秒完成的读次数
w/s: 每秒完成的写次数
rkB/s: 每秒读数据量(kB为单位)
wkB/s: 每秒写数据量(kB为单位)
avgrq-sz:平均每次IO操作的数据量(扇区数为单位)
avgqu-sz: 平均等待处理的IO请求队列长度
await: 平均每次IO请求等待时间(包括等待时间和处理时间，毫秒为单位)
svctm: 平均每次IO请求的处理时间(毫秒为单位)
%util: 采用周期内用于IO操作的时间比率，即IO队列非空的时间比率
```

<font color=#FF0000 size=5> <p align="center">ifstat</p></font>

ifstat命令是一个统计网络接口活动状态的工具，选项含义如下：
```
[root@dwj ~]# ifstat -tT
-l 监测环路网络接口（lo）。缺省情况下，ifstat监测活动的所有非环路网络接口。经使用发现，加上-l参数能监测所有的网络接口的信息，而不是只监测 lo 的接口信息，也就是说，加上-l参数比不加-l参数会多一个lo接口的状态信息。
-a 监测能检测到的所有网络接口的状态信息。使用发现，比加上-l参数还多一个plip0的接口信息，搜索一下发现这是并口（网络设备中有一个叫PLIP(Parallel Line Internet Protocol). 它提供了并口...）
-z 隐藏流量是无的接口，例如那些接口虽然启动了但是未用的
-i 指定要监测的接口,后面跟网络接口名
-s 等于加-d snmp:[comm@][#]host[/nn]] 参数，通过SNMP查询一个远程主机
-h 显示简短的帮助信息
-n 关闭显示周期性出现的头部信息（也就是说，不加-n参数运行ifstat时最顶部会出现网络接口的名称，当一屏显示不下时，会再一次出现接口的名称,提示我们显示的流量信息具体是哪个网络接口的。加上-n参数把周期性的显示接口名称关闭，只显示一次）
-t 在每一行的开头加一个时间 戳（能告诉我们具体的时间）
-T 报告所有监测接口的全部带宽（最后一列有个total，显示所有的接口的in流量和所有接口的out流量，简单的把所有接口的in流量相加,out流量相加）
-w 用指定的列宽，而不是为了适应接口名称的长度而去自动放大列宽
-W 如果内容比终端窗口的宽度还要宽就自动换行
-S 在同一行保持状态更新（不滚动不换行）注：如果不喜欢屏幕滚动则此项非常方便，与bmon的显示方式类似
-b 用kbits/s显示带宽而不是kbytes/s
-q 安静模式，警告信息不出现
-v 显示版本信息
-d 指定一个驱动来收集状态信息
```

<font color=#FF0000 size=5> <p align="center">dstat</p></font>

dstat可以取代vmstat、iostat、netstat、ifstat的程序,可以实时的监控cpu、磁盘、网络、IO、内存等使用情况，功能描述：
```
1.结合了vmstat，iostat，ifstat，netstat以及更多的信息
2.实时显示统计情况
3.在分析和排障时可以通过启用监控项并排序
4.模块化设计
5.使用python编写的，更方便扩展现有的工作任务
6.容易扩展和添加你的计数器（请为此做出贡献）
7.包含的许多扩展插件充分说明了增加新的监控项目是很方便的
8.可以分组统计块设备/网络设备，并给出总数
9.可以显示每台设备的当前状态
10.极准确的时间精度，即便是系统负荷较高也不会延迟显示
11.显示准确地单位和和限制转换误差范围
12.用不同的颜色显示不同的单位
13.显示中间结果延时小于1秒
14.支持输出CSV格式报表，并能导入到Gnumeric和Excel以生成图形
```
语法：dstat [-afv] [options] [delay [count]]
```
参数详解：
  -c：显示CPU系统占用，用户占用，空闲，等待，中断，软件中断等信息
	-C：当有多个CPU时候，此参数可按需分别显示cpu状态，例：-C 0,1 是显示cpu0和cpu1的信息
	-d：显示磁盘读写数据大小
	-D hda,total：include hda and total
	-n：显示网络状态
	-N eth1,total：有多块网卡时，指定要显示的网卡
	-l：显示系统负载情况
	-m：显示内存使用情况
	-g：显示页面使用情况
	-p：显示进程状态
	-s：显示交换分区使用情况
	-S：类似D/N
	-r：I/O请求情况
	-y：系统状态
	--ipc：显示ipc消息队列，信号等信息
	--socket：用来显示tcp udp端口状态
	-a：此为默认选项，等同于-cdngy
	-v：等同于 -pmgdsc -D total
  -t：显示时间信息
	--output 文件：此选项也比较有用，可以把状态信息以csv的格式重定向到指定的文件中，例：dstat --output /root/dstat.csv & 此时让程序默默的在后台运行并把结果输出到/root/dstat.csv文件中
```
监控swap，process，sockets，filesystem并显示监控的时间,若要将结果输出到文件可以加 --output filename
>[root@dwj ~]# dstat -tsp --socket --fs

这样生成的csv文件可以用excel打开，然后生成图表
>[root@dwj ~]# dstat -tsp --socket --fs --output /opt/ds.csv

通过dstat --list可以查看能使用的所有参数，/usr/share/dstat是插件目录，这些插件可以监控电源、mysql等相关系统数据

<font color=#FF0000 size=5> <p align="center">traceroute</p></font>

traceroute 是用来发出数据包从主机到目标主机之间所经过的网关的工具。traceroute 的原理是试图以最小的TTL发出探测包来跟踪数据包到达目标主机所经过的网关，然后监听一个来自网关ICMP的应答。发送数据包的大小默认为38个字节

使用格式：traceroute [参数选项] 域名、 hostname、IP地址
```
[root@dwj ~]# traceroute localhost
参数选项：
	-i 指定网络接口，对于多个网络接口有用。比如 -i eth1 或-i ppp1等；
	-m 把在外发探测试包中所用的最大生存期设置为max-ttl次转发，默认值为30次；
	-n 显示IP地址，不查主机名。当DNS不起作用时常用到这个参数；

	-p port 探测包使用的基本UDP端口设置为port ，默认值是33434
	-q n 在每次设置生存期时，把探测包的个数设置为值n，默认时为3；
	-r  绕过正常的路由表，直接发送到网络相连的主机；
	-w n 把对外发探测包的等待响应时间设置为n秒，默认值为3秒；
```
序列号从1开始，每个纪录就是一跳，每跳表示一个网关，我们看到每行有三个时间，单位是ms，其实就是-q的默认参数。探测数据包向每个网关发送三个数据包后，网关响应后返回的时间

常用的一些参数的用法示例
```
[root@localhost ~]# traceroute -m 10 linuxsir.org         #把跳数设置为10次
[root@localhost ~]# traceroute -n linuxsir.org            #显示IP地址，不查主机名
[root@localhost ~]# traceroute -p 6888 linuxsir.org       #探测包使用的基本UDP端口设置6888
[root@localhost ~]# traceroute -q 4 linuxsir.org          #把探测包的个数设置为值4
[root@localhost ~]# traceroute -r linuxsir.org            #绕过正常的路由表，直接发送到网络相连的主机
[root@localhost ~]# traceroute -w 3 linuxsir.org          #把对外发探测包的等待响应时间设置为5秒
在windows系统中，用tracert来跟踪路由
```

<font color=#FF0000 size=5> <p align="center">iotop</p></font>

iotop命令是一个用来监视磁盘I/O使用状况的top类工具。具有与top相似的UI，其中包括PID、用户、I/O、进程等相关信息。Linux下的IO统计工具如iostat，nmon等大多数是只能统计到per设备的读写情况，如果想知道每个进程是如何使用IO的就可以使用iotop命令

```
[root@dwj ~]# iotop
选项详解：
-o：只显示有io操作的进程
-b：批量显示，无交互，主要用作记录到文件
-n NUM：显示NUM次，主要用于非交互式模式
-d SEC：间隔SEC秒显示一次
-p PID：监控的进程pid
-u USER：监控的进程用户
```
常用快捷键：
```
-> <-左右箭头：改变排序方式，默认是按IO排序
r：改变排序顺序
o：只显示有IO输出的进程
p：进程/线程的显示方式的切换
a：显示累积使用量
q：退出
```

<font color=#FF0000 size=5> <p align="center">slabtop</p></font>

Linux slabtop命令显示内核片缓存信息，Linux内核需要为临时对象如任务或者设备结构和节点分配内存，缓存分配器管理着这些类型对象的缓存。现代Linux内核部署了该缓存分配器以持有缓存，称之为片。不同类型的片缓存由片分配器维护，linux系统透过/proc/slabinfo来向用户暴露slab的信息

参数使用详解：
1.显示间隔----默认情况下，slabtop每隔3秒刷新一次。可以使用-d或者--delay=N选项来调整刷新间隔，以秒为单位
>[root@dwj ~]# slabtop --delay=1  或者  [root@dwj ~]# slabtop -d 1

2.输出一次----使用-o或--once选项不会刷新输出，它仅仅将一次输出结果丢给STDOUT，然后退出
>[root@dwj ~]# slabtop --once  或者  [root@dwj ~]# slabtop -o

3.排序标准----参数-s或--sort=S选项可以根据指定的排序标准对这些字段排序，确定哪个片缓存显示在顶部
```
The following are valid sort criteria:
a: 对活跃对象编号进行排序
b: 每分片对象数
c: 缓存大小
l: 分片数量
v: 活跃分片数量（注意：这不同于上面讲得活跃对象数量）
n: 缓存名称
o: 对象数量
p: 每分片页面数
s: 对象大小
u: 缓存使用量排序
```
