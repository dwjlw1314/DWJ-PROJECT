------ rhel-server-6.4-x86_64 系统 --------

<font color=#8B0000 size=5>一、NFS服务简介</font>

NFS 在文件传送或信息传送过程中依赖于RPC协议。NFS本身是没有提供信息传输的协议和功能的，NFS也是一个RPC SERVER,
所以只要用到NFS的地方都要启动RPC服务，NFS是一个文件系统，而RPC是负责信息的传输。

<font color=#8B0000 size=5>二、安装NFS服务</font>

nfs-utils-* ：包括基本的NFS命令与监控程序

portmap-* ：支持安全NFS RPC服务的连接

本次安装过程如果没有portmap则可以忽略过去

1.查看系统是否已安装nfs
```
[root@dwj mnt/Packages]# rpm -q *nfs*
package nfs4-acl-tools-0.3.3-6.el6.x86_64.rpm is not installed
package nfs-utils-1.2.3-39.el6.x86_64.rpm is not installed
package nfs-utils-lib-1.1.5-6.el6.x86_64.rpm is not installed
package sblim-cmpi-nfsv3-1.1.1-1.el6.x86_64.rpm is not installed
package sblim-cmpi-nfsv4-1.1.0-1.el6.x86_64.rpm is not installed
```
2.如果系统中已安装NFS所需的软件包，跳过即可，否则进行安装

首先挂载光盘，首先需要先配置yum源，然后进行安装
```
[root@dwj ~]# yum -y install *nfs*
[root@dwj ~]# rpm -q *nfs*
```

<font color=#8B0000 size=5>三、NFS系统守护进程</font>

```
nfsd：它是基本的NFS守护进程，主要功能是管理客户端是否能够登录服务器
mountd：它是RPC安装守护进程，主要功能是管理NFS的文件系统。当客户端顺利通过nfsd登录NFS服务器后，在使用NFS服务所提供的文件前，还必须通过文件使用权限的验证,它会读取NFS的配置文件/etc/exports来对比客户端权限
portmap：主要功能是进行端口映射工作
```

<font color=#8B0000 size=5>四、NFS服务器的配置</font>

NFS服务器的配置相对比较简单，只需要在配置文件中进行设置，然后启动NFS服务器即可

NFS的常用目录：
```
/etc/exports                             NFS服务的主要配置文件
/usr/sbin/exportfs                       NFS服务的管理命令
/usr/sbin/showmount                      客户端的查看命令
/var/lib/nfs/etab                        记录NFS分享出来的目录的完整权限设定值
/var/lib/nfs/xtab                        记录曾经登录过的客户端信息
NFS服务的配置文件为 /etc/exports，这是NFS的主要配置文件，如果没有需要手动创建
```
/etc/exports文件内容格式：

<输出目录> [客户端1 选项（访问权限,用户映射,其他）] [客户端2 选项（访问权限,用户映射,其他）]
```
a. 输出目录：输出目录是指NFS系统中需要共享给客户机使用的目录
b. 客户端：客户端是指网络中可以访问这个NFS输出目录的计算机，客户端常用的指定方式：
指定ip地址的主机：192.168.0.200
指定子网中的所有主机：192.168.0.0/24 192.168.0.0/255.255.255.0
指定域名的主机：david.bsmart.cn
指定域中的所有主机：*.bsmart.cn
所有主机：*
c. 选项：用来设置输出目录的访问权限、用户映射等
```
访问权限选项：
```
设置输出目录只读：ro
设置输出目录读写：rw
```
用户映射选项：
```
all_squash：将远程访问的所有普通用户及所属组都映射为匿名用户或用户组（nfsnobody）
no_all_squash：与all_squash取反（默认设置）
root_squash：将root用户及所属组都映射为匿名用户或用户组（默认设置）
no_root_squash：与rootsquash取反
anonuid=xxx：将远程访问的所有用户都映射为匿名用户，并指定该用户为本地用户（UID=xxx）
anongid=xxx：将远程访问的所有用户组都映射为匿名用户组账户，并指定该匿名用户组账户为本地用户组账户（GID=xxx）
```
其它选项
```
secure：限制客户端只能从小于1024的tcp/ip端口连接nfs服务器（默认设置）
insecure：允许客户端从大于1024的tcp/ip端口连接服务器
sync：将数据同步写入内存缓冲区与磁盘中，效率低，但可以保证数据的一致性
async：将数据先保存在内存缓冲区中，必要时才写入磁盘
wdelay：检查是否有相关的写操作，如果有则将这些写操作一起执行，这样可以提高效率（默认设置）
no_wdelay：若有写操作则立即执行，应与sync配合使用
subtree：若输出目录是一个子目录，则nfs服务器将检查其父目录的权限(默认设置)
no_subtree：即使输出目录是一个子目录，nfs服务器也不检查其父目录的权限，这样可以提高效率
```

<font color=#8B0000 size=5>五、NFS服务器的启动与停止</font>

#启动nfs服务
>[root@dwj ~]# service nfs start  

#客户端命令查看nfs服务端有那些共享目录
>[root@dwj ~]# showmount -e 192.168.8.45

#修改nfs服务端配置文件，添加目录
```
[root@dwj ~]# vim /etc/exports
/media   *(rw)
[root@dwj ~]# service nfs restart
```
#客户端挂载服务目录
>[root@dwj ~]# mount 192.168.8.45:/media /opt

/etc/exports部分参数含义：
```
sync/async             #同步和异步
no_root_squash         #非压制root
root_squash            #压制root，如果用户是root登录，修改为nfsnobody
all_squash             #用户登录nfs时，指定身份UID/GID用户
```
使用autofs可以实现自动挂载，有两个配置文件 /etc/auto.misc , /etc/auto.master
