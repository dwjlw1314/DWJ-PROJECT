>[root@dwj ~]# free -m      #通常我们使用该命令看内存的剩余情况的

![image](https://github.com/dwjlw1314/DWJ-PROJECT/raw/master/PictureSource/1.9.1.png)

上图数据信息可以通过下图来进行解释

![image](https://github.com/dwjlw1314/DWJ-PROJECT/raw/master/PictureSource/1.9.2.jpg)

上面的情况下我们总的内存有1863M，用掉了1234M。 其中buffer+cache总共159+635=794M, 由于这种类型的内存是可以回收的，虽然我们用掉了1234M，但是实际上我们如果实在需要的话，这部分buffer/cache内存是可以放出来的

可以做如下演示：
>[root@dwj ~]# sysctl vm.drop_caches=3
>[root@dwj ~]# free -m

![image](https://github.com/dwjlw1314/DWJ-PROJECT/raw/master/PictureSource/1.9.3.png)

我们把buffer/cache大部分都清除干净了，只用了88M，所以我们这次used的空间是457M，到现在我们比较清楚几个概念：
```
1.总的内存多少
2.buffer/cache内存可以释放的
3.used的内存的概率
```
used的空间(457M)到底用到哪里去了？

首先我们通过nmon这个工具，显示内存的使用情况，界面比较直观
>[root@dwj Desktop]# ./nmon_x86_64_rhel54     #nmon_x86_64_rhel54是nmon对应的redhat可执行程序

![image](https://github.com/dwjlw1314/DWJ-PROJECT/raw/master/PictureSource/1.9.4.png)

使用的内存的去向我们很自然的就想到操作系统系统上的各种进程需要消耗各种内存，透过top命令查看sshd进程：

![image](https://github.com/dwjlw1314/DWJ-PROJECT/raw/master/PictureSource/1.9.5.png)

我们看进程的RES这一项，这个数字通过strace对top和nmon的追踪和结合源码，我们确定这个值是从/proc/PID/statm的第二个字段读取出计算来的
/proc/[PID]/status 中的VmRSS和RES相同，也就是每个进程用了具体的多少页的内存。由于linux系统采用的是虚拟内存，进程的代码，库，堆和栈使用的内存都会消耗内存，但是申请出来的内存，只要没真正touch过，是不算的，因为没有真正为之分配物理页面

我们实际进程使用的物理页面应该用resident set size来算的，遍历所有的进程，就可以知道所有的所有的进程使用的内存，实验RSS的使用情况：
>[root@dwj Desktop]# vim RSS.sh                 #添加如下内容，添加执行权限

```bash
	#/bin/bash
	for PROC in `ls /proc/|grep "^[0-9]"`
	do
	if [ -f /proc/$PROC/statm ]; then
	TEP=`cat /proc/$PROC/statm | awk '{print ($2)}'`
	RSS=`expr $RSS + $TEP`
	fi
	done
	RSS=`expr $RSS \* 4`
	echo $RSS"KB"
```
>[root@dwj Desktop]# ./RSS.sh                     #显示如下内容

![image](https://github.com/dwjlw1314/DWJ-PROJECT/raw/master/PictureSource/1.9.6.png)

从当前数字来看，我们的进程使用了大概583M内存，距离691M还有108M内存哪去了，简单的说内核为了高性能每个需要重复使用的对象都会有个池，
这个slab池会cache大量常用的对象，所以会消耗大量的内存，我们再回头来仔细看下nmon的内存统计表
其中slab和PageTables也使用了一定的内存，通过slabtop查看内核片内存占用信息

>[root@dwj Desktop]# slabtop

![image](https://github.com/dwjlw1314/DWJ-PROJECT/raw/master/PictureSource/1.9.7.png)

从图我们可以看出各种对象的大小和数目，遗憾的是不知道slab和PageTables消耗了多少内存，使用如下脚本进行计算
>[root@dwj Desktop]# echo `cat /proc/slabinfo |awk 'BEGIN{sum=0;}{sum=sum+$3*$4;}END{print sum/1024/1024}'` MB

把每个对象的数目*大小，再累加我们就得到了slab总的内存消耗量：86.3835 MB
>[root@dwj Desktop]# echo `grep PageTables /proc/meminfo | awk '{print $2}'` kb

我们就得到了PageTables总的内存消耗量：29224 kb

以上可以看出内存的去向主要有3个：1. 进程消耗； 2. slab消耗 ；3. pagetables消耗 <br>
通过下面脚本把三种消耗汇总下和free结果进行比对
>[root@dwj Desktop]# vim cm.sh                     #添加如下内容，添加执行权限

```bash
	#/bin/bash
	for PROC in `ls /proc/|grep "^[0-9]"`
	do
	if [ -f /proc/$PROC/statm ]; then
	TEP=`cat /proc/$PROC/statm | awk '{print ($2)}'`
	RSS=`expr $RSS + $TEP`
	fi
	done
	RSS=`expr $RSS \* 4`
	PageTable=`grep PageTables /proc/meminfo | awk '{print $2}'`
	SlabInfo=`cat /proc/slabinfo |awk 'BEGIN{sum=0;}{sum=sum+$3*$4;}END{print sum/1024/1024}'`
	echo $RSS"KB", $PageTable"KB", $SlabInfo"MB"
	printf "rss+pagetable+slabinfo=%sMB\n" `echo $RSS/1024 + $PageTable/1024 + $SlabInfo|bc`
	free -m
```
>[root@dwj Desktop]# ./cm.sh                   #显示如下内容

![image](https://github.com/dwjlw1314/DWJ-PROJECT/raw/master/PictureSource/1.9.8.png)

free报告说379M， CM脚本报告说676.5M， CM多报了297M，是什么原因？<br>
重新校对后和nmon来比对下，slab和pagetable的值是吻合的。 那最大的问题可能在进程的消耗计算上
resident resident set size 包括使用的各种库和so等共享的模块，在前面的计算中重复计算了

>[root@dwj Desktop]# pmap `pgrep bash`

![image](https://github.com/dwjlw1314/DWJ-PROJECT/raw/master/PictureSource/1.9.9.png)

多出的297M正是共享库重复计算的部分
