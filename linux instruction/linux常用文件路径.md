<font color=#FF0000 size=5>可以参考linux文件系统层次结构一书</font>

```
/etc/sysctl.conf                                  #系统内核配置文件

/etc/sysconfig/                                   #系统初始化环境配置文件位置
/etc/sysconfig/network-scripts/ifcfg-eth0         #网络ip配置文件
/etc/sysconfig/network                            #主机名配置
/etc/sysconfig/iptables-config                    #iptables配置文件
/etc/sysconfig/i18n	                              #设置linux系统语言和字符类型
/etc/crontab                                      #系统定义的任务计划
/etc/anacrontab                                   #实现检查过期和未完成的crontab的任务的配置文件
/etc/securetty                                    #确认某个安全终端允许root登录,一般只列出虚拟控制台
/etc/resolv.conf                                  #DNS配置文件，eg：nameserver 8.8.8.8
/etc/nsswitch.conf                                #地址服务器配置文件
/etc/motd                                         #ssh成功登录后自动输出自定义内容，经常用于通告信息
/etc/X11/*                                        #XFree86窗口配置文件
/etc/skel                                         #创建用户时自动创建文件的模版目录
/etc/inittab                                      #系统默认启动级别的配置文件
/etc/default/useradd                              #添加用户的默认信息的文件                                         
/etc/login.defs                                   #用户密码信息的默认属性文件
/etc/pam.d/su                                     #su切换用户的认证配置，注释有效的第一行，可实现所有用户切换需要输入密码

/etc/host.conf                                    #指定主机名查找方法，通常指先查找文件/etc/hosts,找不到时再向DNS服务器请求
/etc/xinetd.conf                                  #xinetd服务配置文件
/etc/xinetd.d/*                                   #该目录下面的每一个文件是就是一个用inetd方式启动的服务
/etc/services                                     #经过定义的有效服务名称(如telnet,echo等)，服务名字到端口的映射关系
/etc/protocols                                    #定义过的协议类型(如最常见的是tcp和udp)
/etc/hosts.allow                                  #设置允许使用xinetd服务的机器，如:All:210.38即允许所有来自210.38.x.x的请求
/etc/hosts.deny                                   #设置不允许使用xinetd服务的机器
/etc/rsyslog.conf                                 #rsyslog v5 configuration file
/etc/logrotate.conf                               #日志轮替配置文件
/etc/networks                                     #文件主要功能是路由表，其他的功能，如添加静态路由、删除路由等可参考man
/etc/mtab                                         #系统在启动时创建的信息文件，内容为已经mount的文件系统，可参考/proc/mounts

/etc/vimrc                                        #vim编辑器参数配置文件
/etc/selinux/config                               #系统策略配置文件
/etc/security/limits.conf                         #系统运行时参数配置文件
/etc/rc.d/init.d/functions                        #定义功能的配置文件
/etc/init.d                                       #描述如下，原因是inode相同
/etc/rc.d/init.d                                  #程序启动脚本存放位置(两者类似mount --bind olddir newdir命令)
/etc/rc.d/rc.local                                #系统启动后自启动脚本的配置文件
/etc/rc.d/rc.sysinit                              #系统启动设置配置文件
/etc/issue                                        #系统版本信息
/etc/system-release                               #系统版本信息
/etc/udev/rule.d/70-persistent-net.rules          #网络mac配置文件

/etc/inputrc                                      #输入设备配置文件
/etc/shells                                       #有效的登录shell的路径名称
/etc/dumpdates                                    #dump命令备份事件文件
/etc/hosts                                        #修改和添加主机信息
/etc/passwd                                       #用户信息表
/etc/shadow                                       #用户密码表
/etc/group                                        #组信息表
/etc/exports                                      #NFS共享的文件系统列表
/etc/yum.conf                                     #yum配置文件
/etc/inetd.conf                                   #inetd超级网络服务器
/etc/profile                                      #每个用户登录需要加载的环境变量
/etc/fstab                                        #开机启动自动挂载文件系统的设置文件
/etc/sudoers                                      #visudo命令实际修改的文件
/etc/logrotate.d                                  #rpm包安装方式的程序日志轮替配置文件

/etc/localtime
/etc/timezone                                     #时间和时区文件
/etc/services                                     #网络服务系统中对应端口关系表库
```

<font color=#FF0000 size=5> <p align="center">porc(系统内存文件，开机自动生成)</p></font>

```
/proc/cpuinfo                           #cpu信息
/proc/meminfo                           #memory信息
/proc/version                           #系统版本信息
/proc/mounts                            #文件系统挂载关系
/proc/dma                               #显示当前使用的 DMA 通道
/proc/filesystems                       #linux系统核心支持的文件系统
/proc/interrupts                        #显示使用的中断，and how many of each there have been.
/proc/ioports                           #当前使用的 I/O 端口
/proc/loadavg                           #系统"平均负载"
/proc/modules                           #当前系统加载了哪些核心模块
/proc/net/*                             #网络协议状态信息
/proc/mdstat                            #RAID设备的信息
/proc/cmdline                           #grub信息,eg: ro root=/dev/vol0/root ...
/proc/sys/kernel/hostname               #主机名的设定
/proc/partitions                        #所有已经分区的信息，如下图

[root@dwj opt]# cat /proc/partitions
major   minor   #blocks    name
    8      16   10485760   sdb
	  8      17    4194304   sdb1
  	8      18          1   sdb2
  	8      21    6289408   sdb5
  	8       0   52428800   sda
  	8       1   19214336   sda1
  	8       2   33213440   sda2
  	8       0    7587840   sr0
  253       0   29298688   dm-0
  253       1    3907584   dm-1
```

<font color=#FF0000 size=5> <p align="center">其他</p></font>

```
~/.local/share/Trash/files/                         #ubuntu系统回收站目录
/usr/src/kernels/3.10.0-327.el7.x86_64/fs           #centos系统支持的文件系统目录
/usr/share/vim/vim72/tutor/tutor.zh.utf-8           #vim基础学习文档
/usr/share/zoneinfo                                 #城市的timezone文件
/usr/share/dict                                     #linux系统单词列表
/usr/share/doc                                      #相关程序文档汇总目录
/usr/share/info                                     #GNU info系统的主文件夹
/usr/share/man                                      #系统手册页目录
/var/lib/xfsdump/inventory                          #xfsdump命令处理记录文件
/var/log/yum.log                                    #yum的日志文件
/var/log/cro：                                      #记录 crond 计划任务产生的事件信息
/var/log/dmesg                                      #记录 Linux 操作系统在引导过程中的各种事件信息
/var/log/maillog                                    #记录进入或发出系统的电子邮件活动
/var/log/lastlog                                    #记录每个用户最近的登录事件
/var/log/secure                                     #记录用户认证相关的安全事件信息
/var/log/wtmp                                       #记录每个用户登录、注销及系统启动和停机事件
/var/log/btmp                                       记录失败的、错误的登录尝试及验证事件。
/var/spool/mqueue                                   #已发送邮件内容
/var/                                               #详细目录信息如下图
```
文件夹  | 描述
:--|:--
cache  | 应用程序缓存数据
lib    | 可变状态信息
locale | /usr/local的可变数据
lock   | 锁文件
log    | 日志文件夹
run    | 进程正在运行的数据
