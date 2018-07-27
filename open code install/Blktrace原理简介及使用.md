<font color=#FF0000 size=4>一、Blktrace简介</font>

Blktrace是一个用户态的工具，针对Linux内核块设备I/O层的跟踪，用来收集磁盘IO信息中当IO进行到块设备层（block层，所以叫blktrace）时的详细信息（如IO请求提交，入队，合并，完成，进行读写的进程名称、进程号、执行时间、读写的物理块号、块大小等等一些列的信息）

块设备层处于下图中的“block layer”

![image](https://github.com/dwjlw1314/DWJ-PROJECT/raw/master/PictureSource/3.13.1.jpg)

Blktrace工作原理

(1)blktrace测试的时候，会分配物理机上逻辑cpu个数个线程，并且每一个线程绑定一个逻辑cpu来收集数据

(2)blktrace在debugfs挂载的路径（默认是/sys/kernel/debug）下每个线程产生一个文件（就有了对应的文件描述符），然后调用ioctl函数（携带文件描述符， \_IOWR(0x12,115,struct blk_user_trace_setup)，& blk_user_trace_setup三个参数），产生系统调用将这些东西给内核去调用相应函数来处理，由内核经由debugfs文件系统往此文件描述符写入数据

(3)blktrace需要结合blkparse来使用，由blkparse来解析blktrace产生的特定格式的二进制数据

(4)blkparse仅打开blktrace产生的文件，从文件里面取数据做展示以及最后做per cpu的统计输出，但blkparse中展示的数据状态（如 A，U，Q，详细见下表）是blkparse在t->action & 0xffff之后自己把数值转换为“A，Q，U之类的状态”来展示的

<p align="center">Blktrace主要选项</p>

<style>
table th:first-of-type {
    width: 150px;
}
</style>

选项    |  使用说明
:---|:---
-b size      | 指定抽取事件的buffer大小，为1024的倍数，单位为KB。默认大小为512KB
-d dev       | 增加需要追踪的设备
-I file      | 增加多个需要追踪的设备，在file中指定。格式为：设备1 设备2   <br>【注】：设备为全路径，设备之间用空格分离
-n num-stub  | 指定使用多少的buffer，默认为四个子buffer
-l           | 使用网络侦听模式blktraceserver即此时运行blktrace的设备作为服务器，侦听来自客户机端的网络信息。网络相关
-h hostname  | 运行网络客户端模式，与由hostname指定的服务器端链接。网络相关
-p number    | 指定使用的网络端口（默认为8462）。网络相关
-o basename  | 从输入文件指定基础名字，默认为: device.blktrace.cpu。若使用命令：-o 则当前与blkparse一起运行在实时模式，发送到标准输出端
-D dir       | 输出到指定的文件夹dir
-w seconds   | 设置运行的时间。单位为秒

Blkparse用于将块设备的事件流解析成有格式的输出
<p align="center">Blkparse主要选项</p>

选项|使用说明
:---|:---
-b batch | 指定标准输入读取的批量
-i file  | 指定输入文件的基本名字，默认为device.blktrace.cpu <br>若使用命令:-i 则与blktrace一起运行在实时模式，从标准输入读取
-F type, <br>fmt -f fmt | 设置输出格式。其中：-f 指定了所有的事件的格式；-F type类型的事件的格式
-o file  | 输出到文件file
-w span  | 显示有span指定的时间间隔的trace　<br>1.end_time: 显示0-end_time时间间隔之间的trace　<br>2.start:end-time: 显示start_time-end_time时间间隔之间的trace

<p align="center">Blkparse主要动作</p>

缩写  | 解释说明
:---|:---
A   |  栈设备的重映射--- Remap动作详细说明了从哪个设备映射到哪个设备
C   |  之前的A发起完成的动作。输出将会详细显示请求的sector和大小，以及成功或者失败
Q   |  将节点加入到给定地址的I/O队列中
I   |  插入的A请求被发送到I/O调度器，此时请求已经完成成型

<p align="center">Blkparse动作相关输出 </p>

缩写   |  解释说明
:---|:---
A    |  输出：起始地址+长度，并伴随着原设备和sector偏移量<br>例如：2531656 + 8 <- (251,2) 72 起始地址+长度 <-（原设备主设备号/此设备号）偏移量
C    |  如果当前有一个payload，则其表示头之后的填充，跟随者出错值。<br>若果当前没有payload，则显示 起始地址+长度（单位为sector）。若果-t选项指定了，则会显示运行时间。无论是以上哪种形式，都会跟着一个完成的出错值; <br>例如：1024 + 1024 [0] 起始地址 + 长度 [出错码]
I/Q  |  若当前有一个payload，则输出payload的字节数目，并跟着payload的16进制表示(圆括号之间)<br>若当先没有p。ayload，则显示起始地址+长度（单位为sector）<br>若果-t选项指定了，则会显示运行时间<br>无论是以上哪种形式，都会跟着与事件相关的命令<br> 例如：20480 + 1024 [dd] 起始地址+长度[命令]

<font color=#FF0000 size=4>二、  Blktrace使用</font>

1.Blktrace安装

[root@dwj ~]# yum -y install blktrace                        //安装

blktrace工作原理需要借助内核经由debugfs文件系统（debugfs文件系统在内存中）来输出信息，首先挂载debugfs文件系统

[root@dwj ~]# mount -t debugfs debugfs /sys/kernel/debug           //设定debug文件系统

或者在/etc/fstab中添加下面一行以便在开机启动的时候自动挂载

debug   /sys/kernel/debug   debugfs   default    0    0

2.Blktrace常用命令

[root@dwj ~]# blktrace -d /dev/sda -o - |blkparse -i -             //blktrace和blkparse的”-“代表终端，截图部分显示

说明：追踪设备/dev/sda的I/O信息，将结果输出到屏幕，并将输出的信息重定向为blkparse的输入。经过blkparse解析之后，将结果输出到屏幕
![image](https://github.com/dwjlw1314/DWJ-PROJECT/raw/master/PictureSource/3.13.2.png)

[root@dwj opt]# blktrace -d /dev/sda -o trace | blkparse -i -

说明：blktrace的结果输出到指定的文件trace中，运行后会产生新的文件名为trace. blktrace.[0-cpu数]

[root@dwj opt]# blkparse -i trace

说明：blkparse将trace文件作为blkparse的输入，blkparse的结果依然输出到屏幕

[root@dwj opt]# blkparse -i trace -o ./trace.txt      //将分析结果输出到文件trace.txt   <br>
![image](https://github.com/dwjlw1314/DWJ-PROJECT/raw/master/PictureSource/3.13.3.png)
<p align="center">上图blkparse输出结果分析</p>

数值  | 解释说明
:---|:---
8,48对应%D    |  major/minor。即：主设备号为8，次设备号为48
0对应%2c      |  CPU ID（2字节宽度），即当前动作发生在CPU 0
1对应%8s      |  序列号
0.000000000 <br>对应%5T.%9t   | 当前事件发生的时间戳。%5T.%9t, 秒级为5个字节的区域，纳秒为9个字节的区域
25413对应%5p  |  进程ID号，5字节的区域
Q对应%2a      |  动作Action的标识，2字节区域。Q表示加入队列的动作
WS对应%3d     |  RWBS，3字节的区域。其中：R：读；W：写；D：块扔除操作；B：barrier操作；S：同步操，WS表示当前为写同步操作
0 + 1024 [dd] |  起始地址为0，长度为1024，命令为dd

3.Blktrace示例

终端1：[root@dwj opt]# blktrace -d /dev/sda -o - |blkparse -i -

终端2：[root@dwj ~]# dd if=/dev/zero of=/root/a1 bs=4k count=2000

终端1显示：默认输出格式 –f "%D %2c %8s %5T.%9t %5p %2a %3d" 其中数字代表占几个字符

8,2    0        1     0.000000000   495  A  WS 25670528 + 8 <- (253,0) 25668480

8,0    0        2     0.000001685   495  A  WS 26696576 + 8 <- (8,2) 25670528

8,0    0        3     0.000003054   495  Q  WS 26696576 + 8 [jbd2/dm-0-8]

8,0    0        4     0.000013339   495  G  WS 26696576 + 8 [jbd2/dm-0-8]

8,0    0        5     0.000017300   495  P   N [jbd2/dm-0-8]

8,0    0        6     0.000019095   495  I  WS 26696576 + 8 [jbd2/dm-0-8]

Action[0]='Q',后面的 26696576 是（8，0即sda）相对8:0的扇区起始号，+8，为后面连续的8个扇区（默认一个扇区512byte，所以8个扇区就是4K）,
后面的[jbd2/dm-0-8]是程序的名字。
Action[0]='A',  后面的 25670528 是相对8:0（即sda）的起始扇区号，（8,2）是相对/dev/sda2分区的扇区号为25668480

(由于/dev/sda2分区时sda磁盘上面的一个分区，故sda2上面的起始位置要先映射到sda磁盘上面去)。

由于扇区号在磁盘上面是连续的，磁盘又被格式化成很多块，一个块里包含多个扇区，所以，扇区号/块大小=块号

根据块号你就可以找到对应的inode，（debugfs -R 'icheck  块号'  具体磁盘或分区）

如你的扇区号是相对sda2上面算出来的块号，那（debugfs –R‘icheck 块号’/dev/sda2）就可以找到对应的inode

根据inode你就可以找到对应的文件是什么了 （ find / -inum your_inode）

<font color=#FF0000 size=5> <p align="center">附录：action含义</p></font>
附录：action含义
C - 完成以前发出的请求已经完成。输出请求的扇区和大小详细说明，以及是否成功
D - 确保先前驻留在块层队列中的请求已发送，io调度程序已经发送到驱动程序
I – inserted A request is being sent to the io scheduler for addition to the
internal queue and later service by the driver. The request is fully formed
at this time.

Q – queued This notes intent to queue io at the given location. No real requests
exists yet.

B – bounced The data pages attached to this bio are not reachable by the
hardware and must be bounced to a lower memory location. This causes
a big slowdown in io performance, since the data must be copied to/from
kernel buffers. Usually this can be fixed with using better hardware -
either a better io controller, or a platform with an IOMMU.

m – message Text message generated via kernel call to blk add trace msg.

M – back merge A previously inserted request exists that ends on the boundary
of where this io begins, so the io scheduler can merge them together.

F – front merge Same as the back merge, except this io ends where a previously
inserted requests starts.

G – get request To send any type of request to a block device, a struct request
Container must be allocated first.

S – sleep No available request structures were available, so the issuer has to
wait for one to be freed.

P – plug When io is queued to a previously empty block device queue, Linux
will plug the queue in anticipation of future iOS being added before this
data is needed.

U – unplug Some request data already queued in the device, start sending
requests to the driver. This may happen automatically if a timeout period
has passed (see next entry) or if a number of requests have been added to
the queue.

T – unplug due to timer If nobody requests the io that was queued after
plugging the queue, Linux will automatically unplug it after a defined
period has passed.

X – split On raid or device mapper setups, an incoming io may straddle a
device or internal zone and needs to be chopped up into smaller pieces
for service. This may indicate a performance problem due to a bad setup
of that raid/dm device, but may also just be part of normal boundary
conditions. dm is notably bad at this and will clone lots of io.

A – remap For stacked devices, incoming io is remapped to device below it in
the io stack. The remap action details what exactly is being remapped to
what.
