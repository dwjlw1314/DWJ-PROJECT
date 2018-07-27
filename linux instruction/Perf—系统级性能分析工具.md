性能相关的处理器硬件特性，硬件特性之 cache，内存读写是很快的，但还是无法和处理器的指令执行速度相比。为了从内存中读取指令和数据，处理器需要等待，用处理器的时间来衡量，这种等待非常漫长。Cache 是一种 SRAM，它的读写速率非常快，能和处理器处理速度相匹配。因此将常用的数据保存在 cache 中，处理器便无须等待，从而提高性能。Cache 的尺寸一般都很小，充分利用 cache 是软件调优非常重要的部分

硬件特性之流水线，超标量体系结构，乱序执行，提高性能最有效的方式之一就是并行。处理器在硬件设计时也尽可能地并行，比如流水线，超标量体系结构以及乱序执行。处理器处理一条指令需要分多个步骤完成，比如先取指令，然后完成运算，最后将计算结果输出到总线上。在处理器内部，这可以看作一个三级流水线，处理器流水线如下图所示： <br>
![image](https://github.com/dwjlw1314/DWJ-PROJECT/raw/master/PictureSource/4.16.1.jpg)

指令从左边进入处理器，上图中的流水线有三级，一个时钟周期内可以同时处理三条指令，分别被流水线的不同部分处理
超标量（superscalar）指一个时钟周期发射多条指令的流水线机器架构，比如 Intel 的 Pentium 处理器，内部有两个执行单元，在一个时钟周期内允许执行两条指令。此外，在处理器内部，不同指令所需要的处理步骤和时钟周期是不同的，如果严格按照程序的执行顺序执行，那么就无法充分利用处理器的流水线。因此指令有可能被乱序执行。上述三种并行技术对所执行的指令有一个基本要求，即相邻的指令相互没有依赖关系。假如某条指令需要依赖前面一条指令的执行结果数据，那么 pipeline 便失去作用，因为第二条指令必须等待第一条指令完成。因此好的软件必须尽量避免这种代码的生成

硬件特性之分支预测，分支指令对软件性能有比较大的影响。尤其是当处理器采用流水线设计之后，假设流水线有三级，当前进入流水的第一条指令为分支指令。假设处理器顺序读取指令，那么如果分支的结果是跳转到其他指令，那么被处理器流水线预取的后续两条指令都将被放弃，从而影响性能。为此，很多处理器都提供了分支预测功能，根据同一条指令的历史执行记录进行预测，读取最可能的下一条指令，而并非顺序读取指令。分支预测对软件结构有一些要求，对于重复性的分支指令序列，分支预测硬件能得到较好的预测结果，而对于类似 switch case 一类的程序结构，则往往无法得到理想的预测结果

上面的几种处理器特性对软件的性能有很大的影响，然而依赖时钟进行定期采样的 profiler 模式无法揭示程序对这些处理器硬件特性的使用情况。处理器厂商针对这种情况，在硬件中加入了 PMU 单元，即 performance monitor unit。PMU 允许软件针对某种硬件事件设置 counter，此后处理器便开始统计该事件的发生次数，当发生的次数超过 counter 内设置的值后，便产生中断。比如 cache miss 达到某个值后，PMU 便能产生相应的中断
捕获这些中断，便可以考察程序对这些硬件特性的利用效率了

Perf是内置于Linux内核源码树中的性能剖析(profiling)工具。它基于事件采样原理，以性能事件为基础，支持针对处理器相关性能指标与操作系统相关性能指标的性能剖析。常用于性能瓶颈的查找与热点代码的定位。CPU周期(cpu-cycles)是默认的性能事件，所谓的CPU周期是指CPU所能识别的最小时间单元，通常为亿分之几秒，是CPU执行最简单的指令时所需要的时间，例如读取寄存器中的内容，也叫做clock tick

Perf是一个包含22种子工具的工具集，以下是最常用的5种：
```
perf-list
perf-stat
perf-top
perf-record
perf-report
```
<font size=5>一、perf list 用来查看perf所支持的性能事件，有软件的也有硬件的</font>

Description：List all symbolic event types    <br>
perf list [hw | sw | cache | tracepoint | event_glob]

1.性能事件的分布
```
hw：Hardware event，9个                  #是由 PMU 硬件产生的事件
sw：Software event，9个                  #是内核软件产生的事件，比如进程切换，tick 数等
cache：Hardware cache event，26个        #是内核中的静态 tracepoint 所触发的事件，用来判断程序运行期间内核的行为细节
tracepoint：Tracepoint event，736个
```
sw实际上是内核的计数器，与硬件无关；hw和cache是CPU架构相关的，依赖于具体硬件；tracepoint是基于内核的ftrace

2.指定性能事件(以它的属性)
```
-e <event> : u     #userspace
-e <event> : k     #kernel
-e <event> : h     #hypervisor
-e <event> : G     #guest counting (in KVM guests)
-e <event> : H     #host counting (not in KVM guests)
```
3.使用例子
```
[root@dwj ~]# perf top -e cycles:k                   #显示内核和模块中，消耗最多CPU周期的函数
[root@dwj ~]# perf top -e kmem:kmem_cache_alloc      #显示分配高速缓存最多的函数
```
<font size=5>二、perf top 性能事件(默认是CPU周期)，显示消耗最多的函数或指令</font>

Description：System profiling tool. Generates and displays a performance counter profile in real time <br>
perf top [-e <EVENT> | --event=EVENT] [<options>]

主要用于实时分析各个函数在某个性能事件上的热度，能够快速的定位热点函数，包括应用程序函数、模块函数与内核函数，甚至能够定位到热点指令。默认的性能事件为cpu cycles

1.输出格式：[root@dwj ~]# perf top      
![image](https://github.com/dwjlw1314/DWJ-PROJECT/raw/master/PictureSource/4.16.2.png)
```
第一列：符号引发的性能事件的比例，默认是占用的cpu周期比例
第二列：符号所在的DSO(Dynamic Shared Object)，可以是应用程序、内核、动态链接库、模块
第三列：DSO的类型。[.]表示此符号属于用户态的ELF文件，包括可执行文件与动态链接库。[k]表述此符号属于内核或模块
第四列：符号名。有些符号不能解析为函数名，只能用地址表示
```
2.常用交互命令

命令 | 描述
---|---
h |显示帮助
P |将当前信息保存到当前目录的perf.hist.Num文件中
d |过滤掉所有不属于此DSO的符号。非常方便查看同一类别的符号
UP/DOWN/PGUP/PGDN/SPACE | 上下和翻页
a：annotate current symbol | 给出汇编语言的注解，给出各条指令的采样率，只对[.]DSO的类型且能解析符号名的条目有效

3.常用命令行参数

命令 | 描述
---|---
-e <event> | 指明要分析的性能事件，可以通过perf-list查看具体事件
-p <pid> | Profile events on existing Process ID (comma sperated list). 仅分析目标进程及其创建的线程
-k <path> | Path to vmlinux. Required for annotation functionality. 带符号表的内核映像所在的路径
-K | 不显示属于内核或模块的符号
-U| 不显示属于用户态程序的符号
-d <n> | 界面的刷新周期，默认为2s，因为perf top默认每2s从mmap的内存区域读取一次性能数据
-G | 得到函数的调用关系图
perf top -G [fractal] | 路径概率为相对值，加起来为100%，调用顺序为从下往上
perf top -G graph | 路径概率为绝对值，加起来为该函数的热度

4.使用例子
```
[root@dwj ~]# perf top                         #默认配置
[root@dwj ~]# perf top -G                      #得到调用关系图
[root@dwj ~]# perf top -e cycles               #指定性能事件
[root@dwj ~]# perf top -p 23015,32476          #查看这两个进程的cpu cycles使用情况
[root@dwj ~]# perf top -s comm,pid,symbol      #显示调用symbol的进程名和进程号
[root@dwj ~]# perf top --comms nginx,top       #仅显示属于指定进程的符号
[root@dwj ~]# perf top --symbols kfree         #仅显示指定的符号
```
<font size=5>三、perf stat 用于分析指定程序的性能概况</font>

Description：Run a command and gather performance counter statistics   <br>
perf stat [-e <EVENT> | --event=EVENT] [-a] <command>     <br>
perf stat [-e <EVENT> | --event=EVENT] [-a] - <command> [<options>]   <br>
有些程序慢是因为计算量太大，其多数时间都应该在使用CPU进行计算，这叫做CPU bound型；有些程序慢是因为过多的IO，
这种时候其 CPU 利用率应该不高，这叫做 IO bound 型；对于 CPU bound 程序的调优和 IO bound 的调优是不同的

1.输出格式：[root@dwj ~]# perf stat ls
```
Performance counter stats for 'ls':  
0.653782 task-clock                # 0.691 CPUs utilized
0 context-switches                 #  0.000 K/sec
0 CPU-migrations                   #  0.000 K/sec
247 page-faults                    #  0.378 M/sec
1,625,426 cycles                   #  2.486 GHz
1,050,293 stalled-cycles-frontend  # 64.62% frontend cycles idle
838,781 stalled-cycles-backend     # 51.60% backend  cycles idle
1,055,735 instructions             #  0.65  insns per cycle
210,587 branches                   # 322.106 M/sec
10,809 branch-misses               # 5.13% of all branches
12152161 cache-references # 46.252 M/sec (scaled from 96.16%)
7245338 cache-misses # 27.576 M/sec (scaled from 95.49%)
0.000945883 seconds time elapsed  
```
输出包括ls的执行时间，以及10个性能事件的统计

字段名 | 内容描述
---|---
task-clock |任务真正占用的处理器时间，单位为ms。CPUs utilized = task-clock / time elapsed，CPU的占用率
context-switches | 上下文的切换次数，记录了程序运行过程中发生了多少次进程切换，频繁的进程切换是应该避免的
CPU-migrations | 处理器迁移次数。Linux为了维持多个处理器的负载均衡，即被调度器从一个 CPU 转移到另外一个 CPU 上运行
page-faults | 缺页异常的次数。程序请求的页面未建立或不在内存中，或者页面在内存中，但物理和虚拟地址的映射关系未建立时，都会触发。另外TLB不命中，页面访问权限不匹配等情况也会触发缺页异常
cycles | 处理器时钟，一条机器指令可能需要多个cycles。消耗的处理器周期数，可以用cycles/task-clock算出
IPC | 是 Instructions/Cycles 的比值，该值越大越好，说明程序充分利用了处理器的特性
instructions | 执行了多少条指令。IPC为平均每个cpu cycle执行了多少条指令
branches | 遇到的分支指令数；branch-misses是预测错误的分支指令数
Cache-misses | 程序运行过程中总体的 cache 利用情况，如果该值过高，说明程序的 cache 利用不好
Cache-references | cache 命中的次数
Cache-misses | cache 失效的次数

2.常用参数

参数名 | 描述
---|---
-p | 仅分析目标进程及其创建的线程
-a | 从所有CPU上收集性能数据
-r | 重复执行命令求平均
-C | 从指定CPU上收集性能数据
-v | 显示更多性能数据
-n | 只显示任务的执行时间
-x SEP | 指定输出列的分隔符
-o file | 指定输出文件，--append指定追加模式
--pre <cmd> | 执行目标程序前先执行的程序
--post <cmd> | 执行目标程序后再执行的程序

3.使用例子
```
[root@dwj ~]# perf stat -r 10 ls > /dev/null         #执行10次程序，给出标准偏差与期望的比值
[root@dwj ~]# perf stat -v ls > /dev/null            #显示更详细的信息
[root@dwj ~]# perf stat -n ls > /dev/null            #只显示任务执行时间，不显示性能计数器
[root@dwj ~]# perf stat -a -A ls > /dev/null         #单独给出每个CPU上的信息
[root@dwj ~]# perf stat -e syscalls:sys_enter ls     #ls命令执行了多少次系统调用
```
<font size=5>四、perf-record 收集采样信息，记录在数据文件中,用perf-report对文件进行分析</font>

Description：Run a command and record its profile into perf.data.This command runs a command and gathers a performance counter profile from it, into perf.data,without displaying anything. This file can then be inspected later on, using perf report

1.常用参数

参数名 | 描述
---|---
-e |Select the PMU event.
-a | System-wide collection from all CPUs.
-p | Record events on existing process ID (comma separated list).
-A | Append to the output file to do incremental profiling.
-f | Overwrite existing data file.
-o | Output file name.
-g | Do call-graph (stack chain/backtrace) recording.
-C | Collect samples only on the list of CPUs provided.

2.使用例子

    [root@dwj ~]# perf record ls -g                        #记录执行ls时的性能数据
    [root@dwj ~]# perf record -p 'pgrep -d ',' nginx'      #记录nginx进程的性能数据
    [root@dwj ~]# perf record -e syscalls:sys_enter ls     #记录执行ls时的系统调用，可以知道哪些系统调用最频繁

<font size=5>五、perf-report 读取perf record创建的数据文件，并给出热点分析结果</font>   <br>
Description：Read perf.data (created by perf record) and display the profile This command displays the performance counter profile information recorded via perf record.

1.输出格式：

    [root@dwj ~]# perf record -e cpu-clock ./a.out
    [root@dwj ~]# perf report

![image](https://github.com/dwjlw1314/DWJ-PROJECT/raw/master/PictureSource/4.16.3.png)
高的热点代码片段是longa( )函数，如果需要按照调用关系进行显示的统计信息，使用perf的-g选项便可以得到需要的信息：

    [root@dwj ~]# perf record -e cpu-clock -g ./a.out
    [root@dwj ~]# perf report

![image](https://github.com/dwjlw1314/DWJ-PROJECT/raw/master/PictureSource/4.16.4.png)
有'+'号可以按Enter键展开查看详情，按键'a'查看汇编信息
![image](https://github.com/dwjlw1314/DWJ-PROJECT/raw/master/PictureSource/4.16.5.png)

2.常用参数
>-i：Input file name. (default: perf.data)

3.使用例子
>perf report -i perf.data.2

除了以上5个常用工具外，还有一些用于特殊场景， 比如perf-lock内核锁、perf-kmem(slab性能分配器)、probe-sched调度器

perf-lock 内核锁的性能分析  <br>
Description：Analyze lock events  <br>
perf lock {record | report | script | info}

需要编译选项的支持：

    CONFIG_LOCKDEP、CONFIG_LOCK_STAT
    CONFIG_LOCKDEP defines acquired and release events
    CONFIG_LOCK_STAT defines contended and acquired lock events

1.常用选项

    -i <file>：输入文件
    -k <value>：sorting key，默认为acquired，还可以按contended、wait_total、wait_max和wait_min来排序。

2.使用例子

    [root@dwj ~]# perf lock record ls
    [root@dwj ~]# perf lock report

3.输出样式
```
      Name           acquired  contended total   wait (ns)   max wait (ns)   min wait (ns)   
&mm->page_table_...     382          0              0            0            0   
&mm->page_table_...     72           0              0            0            0   
&fs->lock               64           0              0            0            0   
dcache_lock             62           0              0            0            0   
vfsmount_lock           43           0              0            0            0   
&newf->file_lock...     41           0              0            0            0   

Name：      内核锁的名字
max wait：  为了获得该锁，最大的等待时间
min wait：  为了获得该锁，最小的等待时间
aquired：   该锁被直接获得的次数，因为没有其它内核路径占用该锁，此时不用等待
contended： 该锁等待后获得的次数，此时被其它内核路径占用，需要等待
total wait：为了获得该锁，总共的等待时间
```
