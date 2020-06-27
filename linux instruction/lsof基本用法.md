一、命令简介
```
lsof(list open files)是一个列出当前系统打开文件的工具。在linux环境下，任何事物都以文件的形式存在，通过文件不仅仅可以访问常规数据，还可以访问网络连接和硬件。所以如传输控制协议 (TCP) 和用户数据报协议 (UDP) 套接字等，系统在后台都为该应用程序分配了一个文件描述符，无论这个文件的本质如何，该文件描述符为应用程序与基础操作系统之间的交互提供了通用接口。因为应用程序打开文件的描述符列表提供了大量关于这个应用程序本身的信息，通过lsof工具能够查看这个列表对系统监测以及排错将是很有帮助的
```

lsof语法格式是： lsof ［options］ [filename]

二、输出信息含义

在终端下输入lsof即可显示系统打开的文件，因为需要访问核心内存和各种文件，必须以 root 用户的身份运行

<font color=#EEEE>直接输入lsof输出参数描述: </font>
```
COMMAND    PID   TID   USER   FD   TYPE   DEVICE    SIZE/OFF   NODE   NAME
```
每行显示一个打开的文件，若不指定条件默认将显示所有进程打开的所有文件
```
COMMAND：进程的名称
PID：进程标识符
USER：进程所有者
FD：文件描述符，应用程序通过文件描述符识别该文件。如cwd、txt等
TYPE：文件类型，如DIR、REG等
DEVICE：指定磁盘的名称
SIZE：文件的大小
NODE：索引节点（文件在磁盘上的标识）
NAME：打开文件的确切名称

FD 列中的文件描述符cwd 值表示应用程序的当前工作目录，这是该应用程序启动的目录，除非它本身对这个目录进行更改,txt 类型的文件是程序代码，如应用程序二进制文件本身或共享库
其次数值表示应用程序的文件描述符，这是打开该文件时返回的一个整数。
u 表示该文件被打开并处于读取/写入模式，而不是只读 r 或只写 w 模式。同时还有大写的 W 表示该应用程序具有对整个文件的写锁。该文件描述符用于确保每次只能打开一个应用程序实例

Type 列则比较直观。文件和目录分别称为 REG 和 DIR。而CHR 和 BLK，分别表示字符和块设备；或者 UNIX、FIFO 和 IPv4，分别表示 UNIX 域套接字、先进先出 (FIFO) 队列和网际协议 (IP) 套接字
```

三、lsof使用实例

<font color=#1199>查找谁在使用文件系统</font>

在卸载文件系统时，如果该文件系统中有任何打开的文件，操作通常将会失败。通过lsof可以找出那些进程在使用当前要卸载的文件系统
>[root@localhost dwj]# lsof /root

<font color=#1199>恢复删除的文件</font>

```
当Linux计算机受到入侵时，常见的情况是日志文件被删除，以掩盖攻击者的踪迹。管理错误也可能导致意外删除重要的文件，比如在清理旧日志时，意外地删除了数据库的活动事务日志
可以通过lsof来恢复这些文件。当进程打开了某个文件时，只要该进程保持打开该文件，即使将其删除，它依然存在于磁盘中。这时候进程并不知道文件已经被删除，它仍然可以向打开该文件时提供给它的文件描述符进行读取和写入。除了该进程之外，这个文件是不可见的，因为已经删除了其相应的目录索引节点
在 /proc 目录下，其中包含了反映内核和进程树的各种文件。该目录挂载的是在内存中所映射的一块区域，所以这些文件和目录并不存在于磁盘中，因此当我们对这些文件进行读取和写入时，实际上是在从内存中获取相关信息
大多数与 lsof 相关的信息都存储于以进程的 PID 命名的目录中，即 /proc/1234 中包含的是 PID 为 1234 的进程的信息。每个进程目录中存在着各种文件，它们可以使得应用程序简单地了解进程的内存空间、文件描述符列表、指向磁盘上的文件的符号链接和其他系统信息
lsof 程序使用该信息和其他关于内核内部状态的信息来产生其输出。所以lsof 可以显示进程的文件描述符和相关的文件名等信息。也就是我们通过访问进程的文件描述符可以找到该文件的相关信息
当系统中的某个文件被意外地删除了，只要这个时候系统中还有进程正在访问该文件，那么我们就可以通过lsof从/proc目录下恢复该文件的内容
 ```

假如由于误操作将/var/log/messages文件删除掉了，那么这时要将/var/log/messages文件恢复的方法如下：

首先使用lsof来查看当前是否有进程打开/var/logmessages文件
>[root@localhost dwj]# lsof | grep /var/log/messages   #结果如下：<br>
>syslogd 1283 root 2w REG 3,3 5381017 1773647 /var/log/messages (deleted)
```
从上面的信息可以看到 PID 1283（syslogd）打开文件的文件描述符为 2
同时还可以看到/var/log/messages已经标记被删除了。因此我们可以在 /proc/1283/fd/2 (fd下的每个以数字命名的文件表示进程对应的文件描述符)中查看相应的信息
```
>[root@localhost dwj]# head -n 10 /proc/1283/fd/2

输出信息可以看出，查看 /proc/1283/fd/2 就可以得到所要恢复的数据。如果可以通过文件描述符查看相应的数据，那么就可以使用 I/O 重定向将其复制到文件中，如: cat /proc/1283/fd/2 > /var/log/messages 对于许多应用程序，尤其是日志文件和数据库，这种恢复删除文件的方法非常有用

<font color=#1199>列出进程可以打开的文件的信息</font>
```
被打开的文件可以是:
1.普通的文件
2.目录
3.网络文件系统的文件
4.字符设备文件
5.(函数)共享库
6.管道，命名管道
7.符号链接
8.底层的socket字流，网络socket，unix域名socket
9.在linux里面，大部分的东西都是被当做文件的
```

<font color=#1199>lsof 命令的使用</font>

查看谁正在使用某个文件
>[root@localhost dwj]# lsof /filepath/file

递归查看某个目录的文件信息(备注: 使用了+D，对应目录下的所有子目录和文件都会被列出)
>[root@localhost dwj]# lsof +D /opt/dwj

比使用+D选项，遍历查看某个目录的所有文件信息 的方法
>[root@localhost dwj]# lsof | grep '/opt/dwj'

列出某个用户打开的文件信息(备注: -u 选项其实是user的缩写)
>[root@localhost dwj]# lsof -u username

列出某个程序所打开的文件信息(备注: -c 选项将会列出所有以mysql开头的程序的文件)
>[root@localhost dwj]# lsof -c mysql

列出多个程序多打开的文件信息
>[root@localhost dwj]# lsof -c mysql -c apache

列出某个用户以及某个程序所打开的文件信息
>[root@localhost dwj]# lsof -u test -c mysql

列出除了某个用户外的被打开的文件信息(备注：^这个符号在用户名之前，进程显示非root信息)
>[root@localhost dwj]# lsof -u ^root

通过某个进程号显示进程打开的文件
>[root@localhost dwj]# lsof -p 1

列出多个进程号对应的文件信息
>[root@localhost dwj]# lsof -p 123,456,789

列出除了某个进程号，其他进程号所打开的文件信息
>[root@localhost dwj]# lsof -p ^1

<font color=#EE88>列出所有的网络连接</font>
>[root@localhost dwj]# lsof -i

列出所有tcp 网络连接信息
>[root@localhost dwj]# lsof -i tcp

列出所有udp网络连接信息
>[root@localhost dwj]# lsof -i udp

列出谁在使用某个端口
>[root@localhost dwj]# lsof -i :3306

列出谁在使用某个特定的udp端口
>[root@localhost dwj]# lsof -i udp:55

列出特定的tcp端口
>[root@localhost dwj]# lsof -i tcp:80

列出某个用户的所有活跃的网络端口
>[root@localhost dwj]# lsof -a -u test -i

列出所有网络文件系统
>[root@localhost dwj]# lsof -N

不断查看目前ftp连接的情况
>[root@localhost dwj]# lsof -i tcp@www.baidu.com -r

列出域名socket文件
>[root@localhost dwj]# lsof -u

某个用户组所打开的文件信息
>[root@localhost dwj]# lsof -g 1

根据文件描述列出对应的文件信息
>[root@localhost dwj]# lsof -d description(like 2)

根据文件描述范围列出文件信息
>[root@localhost dwj]# lsof -d 2-3
