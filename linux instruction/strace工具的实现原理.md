<font size=5>一、strace简介</font>

strace是Linux系统下的一个用来跟踪进程执行时系统调用和所接收的信号的工具，它的实现基础是ptrace系统调用。在Linux世界，
进程不能直接访问硬件设备，当进程需要访问硬件设备(比如读取磁盘文件，接收网络数据等等)时，必须由用户态模式切换至内核态模式，
通过系统调用访问硬件设备。strace可以跟踪到一个进程产生的系统调用,包括参数，返回值，执行消耗的时间，安装strace之后，就可以使用了

<font size=5>二、strace使用</font>

1.strace PROG，PROG是要执行的程序。strace命令执行的结果就是按照调用顺序打印出所有的系统调用，包括函数名、参数列表以及返回值

2.使用strace跟踪一个进程的系统调用的基本流程如图

![image](https://github.com/dwjlw1314/DWJ-PROJECT/raw/master/PictureSource/4.18.1.png)

<p align="center">图1 strace实现流程</p>

从图中可以看出strace做了以下几件事情：
```
1.设置SIGCHLD 信号的处理函数，这个处理函数只要不是SIG_IGN即可。由于子进程停止后是通过SIGCHLD信号通知父进程的，所以这里要防止SIGCHLD信号被忽略
2.创建子进程，在子进程中调用ptrace(PTRACE_TRACEME,0L, 0L, 0L)使其被父进程跟踪，并通过execv函数执行被跟踪的程序
3.通过wait()等待子进程停止，并获得子进程停止时的状态status
4.通过子进程的状态查看子进程是否已正常退出，如果是，则不再跟踪，随后调用ptrace发送PTRACE_DETACH请求解除跟踪关系
5.子进程停止后，打印系统调用的函数名、参数和返回值。具体流程见图2
6.通过PTRACE_SYSCALL让子进程继续运行，由于这个请求会让子进程在系统调用的入口处和系统调用完成时都会停止并通知父进程，这样，父进程就可以在系统调用开始之前获得参数，结束之后获得返回值
```
在系统调用的入口和结束时子进程停止运行时，这时父进程认为子进程是因为收到SIGTRAP信号而停止的。所以父进程在wait()后可以通过SIGTRAP来与其他信号区分开，上面已经提到，子进程会在系统调用前后各停止一次，所以打印系统调用信息时分为两个阶段：在系统调用开始时可以获取系统调用号和参数，在系统调用结束时可以获取系统调用的返回结果。通过给tcb结构的flags字段清除和添加TCB_INSYSCALL标志位来区分系统调用的开始和结束

![image](https://github.com/dwjlw1314/DWJ-PROJECT/raw/master/PictureSource/4.18.2.png)

<p align="center">图2 strace中打印系统调用的实现流程</p>

跟踪一个正在运行的进程，使用-p选项加上进程的pid

跟踪某个特定的系统调用，使用-e选项加上系统调用名

例如，跟踪进程8080的epoll_wait系统调用：
>[root@dwj ~] strace -e epoll_wait -p 8080

通用的完整用法：
>[root@dwj ~] strace -o output.txt -T -tt -e trace=all -p 28979

跟踪28979进程的所有系统调用，并统计系统调用的花费时间，以及开始时间，最后将结果存在output.txt文件里面

<font size=5>三、strace参数</font>

参数名 | 描述
---|---
-c  | 统计和报告每个系统调用所执行的时间、调用次数和出错次数等
-d  | 输出strace关于标准错误的调试信息
-D  | 将跟踪程序进程以分离的孙子进程运行而不是父进程模式
-f  | 跟踪当前进程及其通过fork系统调用所创建的子进程
-ff | 常与-o选项联合使用，不同进程(子进程)的跟踪结果分别输出到相应的filename.pid文件中
-F  | 尝试跟踪vfork系统调用。否则即使打开-f选项，vfork也不会被跟踪
-h  | 输出简要的帮助信息
-i  | 显示发生系统调用时的指令指针(IP)寄存器值
-q  | 抑制(禁止输出)关于结合(attaching)、脱离(detaching)的消息。当输出重定向到一个文件时，自动抑制此类消息
-r  | 显示每个系统调用发生时的相对时间戳，即连续的系统调用起点之间的时间差
-t  | 在输出的每一行前加上时间信息，秒级
-tt | 在输出的每一行前加上时间信息，微秒级
-ttt| 在每行输出前添加相对时间信息，格式为”自纪元时间起经历的秒数.微秒数”
-T  | 显示每个系统调用所耗费的时间，其时间开销在输出行最右侧的尖括号内
-v  | 输出所有的系统调用,一些调用关于环境变量,状态,输入输出等调用由于使用频繁,默认不输出
-V  | 输出strace的版本信息
-x  | 以十六进制形式输出非标准字符串
-xx | 以十六进制形式输出所有字符串
-o  | 将输出信息写入文件file中，例如：strace -c -o test.txt ./test
-u  | 用指定用户的UID和/或GID身份运行待跟踪程序
-E  | 将var参数放入命令的环境变量列表，-E var=value
-E  | 从命令的环境变量列表中移除var参数，-E var
-a column | 设置返回值的输出位置.默认为40
-e expr | 指定一个表达式,用来控制如何跟踪.格式如下: [qualifier=][!]value1[,value2]... qualifier只能是trace、abbrev、verbose、raw、signal、read、write其中之一，value是用来限定的符号或数字.默认的qualifier是trace.感叹号是否定符号，例如:-eopen等价于 -e trace=open,表示只跟踪open调用.而-etrace!=open表示跟踪除了open以外的其他调用。有两个特殊的符号all和none

注意有些shell使用!来执行历史记录里的命令,所以要使用\\

-e控制参数 | 描述
---|---
-e trace=set |只跟踪指定的系统调用 例如-e trace=open,close表示只跟踪这2个系统调用.默认set=all
-e trace=file |只跟踪有关文件操作的系统调用
-e trace=process |只跟踪有关进程控制的系统调用
-e trace=network |跟踪与网络有关的所有系统调用
-e strace=signal |跟踪所有与系统信号有关的 系统调用
-e trace=ipc  |跟踪所有与进程通讯有关的系统调用
-e abbrev=set  |设定strace输出的系统调用的结果集 -v等同abbrev=none.默认为abbrev=all
-e raw=set |将指定的系统调用的参数以十六进制显示
-e signal=set  |指定跟踪的系统信号.默认为all.如 signal=!SIGIO(或者signal=!io),表示不跟踪SIGIO信号
-e read=set  |输出从指定文件中读出的数据.例如: -e read=3,5
-e write=set |输出写入到指定文件中的数据
-o filename  |将strace的输出写入文件filename
-p pid  |跟踪指定的进程pid
-s strsize  |指定输出的字符串的最大长度.默认为32文件名一直全部输出
-u username |username的UID和GID执行被跟踪的命令

<font size=5>四、strace案例</font>

1.用strace调试程序，在理想世界里，每当一个程序不能正常执行一个功能时，它就会给出一个有用的错误提示，同时指出足够改正错误的线索。但遗憾的是，有时候一个程序出现了问题，无法找到原因，这就是调试程序出现的原因。strace是一个必不可少的调试工具，用来监视系统调用。不仅可以调试一个新开始的程序，也可以调试一个已经在运行的程序(把strace绑定到一个已有的PID上面)，首先看一个真实的例子：启动KDE时出现问题
![image](https://github.com/dwjlw1314/DWJ-PROJECT/raw/master/PictureSource/4.18.3.png)

这个错误信息只是一个对KDE来说至关重要的负责进程间通信的程序无法启动，和ICE协议(Inter Client Exchange)有关

采用strace看一下在启动 dcopserver时到底程序做了什么
>[root@dwj ~] strace -f -F -o ~/dcop.txt dcopserver       #程序运行出错前的有关记录如下：

这里-f -F选项告诉strace同时跟踪fork和vfork出来的进程，-o选项把输出写到~/dcop.txt里面，dcopserver是要启动和调试的程序
```
mkdir("/tmp/.ICE-unix", 0777) = -1 EEXIST (File exists)
lstat64("/tmp/.ICE-unix", {st_mode=S_IFDIR|S_ISVTX|0755, st_size=4096, ...}) = 0
unlink("/tmp/.ICE-unix/dcop27207-1066844596") = -1 ENOENT (No such file or directory)
bind(3, {sin_family=AF_UNIX, path="/tmp/.ICE-unix/dcop27207-1066844596"}, 38) = -1 EACCES (Permission denied)
write(2, "_KDE_IceTrans", 13) = 13
write(2, "SocketCreateListener: failed to "..., 46) = 46
close(3) = 0 27207 write(2, "_KDE_IceTrans", 13) = 13
write(2, "SocketUNIXCreateListener: ...Soc"..., 59) = 59
umask(0) = 0 27207 write(2, "_KDE_IceTrans", 13) = 13
write(2, "MakeAllCOTSServerListeners: fail"..., 64) = 64
write(2, "Cannot establish any listening s"..., 39) = 39
```
其中第一行显示程序试图创建/tmp/.ICE-unix目录，权限为0777，这个操作因为目录已经存在而失败了。第二个系统调用（lstat64）检查了目录状态，并显示这个目录的权限是0755，这里出现了第一个程序运行错误的线索：程序试图创建属性为0777的目录，但是已经存在了一个属性为 0755的目录。第三个系统调用(unlink)试图删除一个文件，但是这个文件并不存在。这并不奇怪，因为这个操作只是试图删掉可能存在的老文件。但是，第四行确认了错误所在。他试图绑定到/tmp/.ICE-unix/dcop27207-1066844596，但是出现了拒绝访问错误。ICE_unix目录的用户和组都是root，并且只有所有者具有写权限。一个非root用户无法在这个目录下面建立文件，如果把目录属性改成0777， 则前面的操作有可能可以执行，而这正是第一步错误出现时进行过的操作。所以运行了chmod 0777 /tmp/.ICE-unix之后KDE就可以正常启动了，用strace进行跟踪调试只需要花很短的几分钟时间跟踪程序运行，然后检查并分析输出文件。说明：运行chmod 0777只是一个测试，一般不要把一个目录设置成所有用户可读写，同时不设置粘滞位(sticky bit)。给目录设置粘滞位可以阻止一个用户随意删除可写目录下面其他人的文件。一般你会发现/tmp目录因为这个原因设置了粘滞位。KDE可以正常启动之后，运行chmod +t /tmp/.ICE-unix给.ICE_unix设置粘滞位

2.解决库依赖问题

starce可以解决和动态库相关的问题。当对一个可执行文件运行ldd时，它会告诉你程序使用的动态库和找到动态库的位置，nss(name server switch)尽管这样，ldd并不能把所有程序依赖的动态库列出来，系统调用dlopen可以在需要的时候自动调入需要的动态库，而这些库可能不会被ldd列出来。作为glibc的一部分的NSS库就是一个典型的例子，NSS的一个作用就是告诉应用程序到哪里去寻找系统帐号数据库。应用程序不会直接连接到NSS库，glibc则会通过dlopen自动调入NSS库。如果这样的库偶然丢失，你不会被告知存在库依赖问题，但这样的程序就无法通过用户名解析得到用户ID了。举一个例子：whoami程序会给出当前用户名，在一些需要知道运行程序的真正用户的脚本程序里面非常有用，whoami的一个示例 输出如下：

>[root@dwj ~]# whoami  <br>
    root

假设因为某种原因在升级glibc的过程中负责用户名和用户ID转换的库NSS丢失，可以通过把nss库改名来模拟这个环境：

>[root@dwj ~] mv /lib64/libnss_files.so.2 /lib64/libnss_files.so.2.backup

>[root@dwj ~] whoami  <br>
whoami: cannot find username for UID 0

这里可以看到，运行whoami时出现了错误，ldd程序的输出不会提供有用的帮助

>[root@dwj ~] ldd /usr/bin/whoami

![image](https://github.com/dwjlw1314/DWJ-PROJECT/raw/master/PictureSource/4.18.4.png)
只会看到whoami依赖Libc.so.6和ld-linux.x86-64.so.2，它没有给出运行whoami所必须的其他库

>[root@dwj ~] strace -o whoami-strace.txt whoami        这里时用strace跟踪 whoami时的部分输出
```
open("/lib64/libnss_files.so.2", O_RDONLY) = -1 ENOENT (No such file or directory)
open("/lib64/tls/x86_64/libnss_files.so.2", O_RDONLY) = -1 ENOENT (No such file or directory)
stat("/lib64/tls/x86_64", 0x7fff167175d0) = -1 ENOENT (No such file or directory)
open("/lib64/tls/libnss_files.so.2", O_RDONLY) = -1 ENOENT (No such file or directory)
stat("/lib64/tls", {st_mode=S_IFDIR|0555, st_size=4096, ...}) = 0
open("/lib64/x86_64/libnss_files.so.2", O_RDONLY) = -1 ENOENT (No such file or directory)
stat("/lib64/x86_64", 0x7fff167175d0)   = -1 ENOENT (No such file or directory)
open("/lib64/libnss_files.so.2", O_RDONLY) = -1 ENOENT (No such file or directory)
stat("/lib64", {st_mode=S_IFDIR|0555, st_size=12288, ...}) = 0
open("/usr/lib64/tls/x86_64/libnss_files.so.2", O_RDONLY) = -1 ENOENT (No such file or directory)
stat("/usr/lib64/tls/x86_64", 0x7fff167175d0) = -1 ENOENT (No such file or directory)
open("/usr/lib64/tls/libnss_files.so.2", O_RDONLY) = -1 ENOENT (No such file or directory)
stat("/usr/lib64/tls", {st_mode=S_IFDIR|0555, st_size=4096, ...}) = 0
open("/usr/lib64/x86_64/libnss_files.so.2", O_RDONLY) = -1 ENOENT (No such file or directory)
stat("/usr/lib64/x86_64", 0x7fff167175d0) = -1 ENOENT (No such file or directory)
open("/usr/lib64/libnss_files.so.2", O_RDONLY) = -1 ENOENT (No such file or directory)
```
