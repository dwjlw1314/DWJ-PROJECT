RAID0 创建过程，首先添加sdb硬盘，分2个1G的主分区sdb1和sdb2；
* 创建RAID0
* 导出阵列配置文件
* 格式化并挂载到指定的目录
* 修改/etc/fstab实现开机自动挂载

生成名字为md0的RAID0阵列，等级为0，-n是两个raid，指定两个硬盘，sdb1和sdb2
>[root@dwj ~]# mdadm -C -v /dev/md0 -l 0 -n 2 /dev/sdb1 /dev/sdb2

查看md0的创建用户和uuid信息
>[root@dwj ~]# mdadm Ds

显示md0的详细信息
>[root@dwj ~]# mdadm Ds /dev/md0

生成md0 的配置文件，指定配置文件名
>[root@dwj ~]# mdadm -Ds > /etc/mdadm.conf

查看mdadm.conf文件
>[root@dwj ~]# cat /etc/mdadm.conf  <br>
ARRAY /dev/md0 metadata=1.2 name=dwj:0 UUID=8660605b:2ef7d6c5:52d7ea8b:435158aa   <br>
ARRAY /dev/md0 metadata=1.2 spares=1 name=dwj:1 UUID=8660605b:2ef7d6c5:52d7ea8b:435158aa

给md0进行分区
>[root@dwj ~]# fdisk /dev/md0

格式化md0p1分区
>[root@dwj ~]# mkfs.ext4 /dev/md0p1

创建分区挂载目录
>[root@dwj ~]# mkdir /raid0

把分区挂载到创建的挂载目录上
>[root@dwj ~]# mount /dev/md0p1 /raid0/

开机自动挂载md0p1，修改fstab文件，最后添加一行
>/dev/md0p1   /raid0   ext4  defaults  0  0

* 创建RAID1，添加sdc硬盘，分3个1G的主分区sdc1和sdc2 sdc3；

生成名字为md1的RAID1阵列，等级为1，-n是两个raid，-x指定1个热备盘
>[root@dwj ~]# mdadm -C -v /dev/md1 -l 1 -n 2 -x 1 /dev/sdc{1,2,3}

查看RAID运行情况
```
[root@dwj raid0]# cat /proc/mdstat
Personalities : [raid0] [raid1]
md1 : active raid1 sdc3[2](S) sdc1[1] sdc1[0]
	  1059200 blocks super 1.2 [2/2] [UU]
md0 : active raid0 sdb2[1] sdb1[0]
	  2119680 blocks super 1.2 512k chunks
```
生成md1 的配置文件，指定配置文件名
>[root@dwj raid0]# mdadm -Ds > /etc/mdadm.conf

给md1进行分区
>[root@dwj ~]# fdisk /dev/md1

格式化md1p1分区
>[root@dwj ~]# mkfs.ext4 /dev/md1p1

创建分区挂载目录
>[root@dwj ~]# mkdir /raid1

把分区挂载到创建的挂载目录上
>[root@dwj ~]# mount /dev/md1p1 /raid1/

开机自动挂载md1p1，修改fstab文件，最后添加一行
>/dev/md0p1   /raid0   ext4  defaults  0  0    <br>
 /dev/md1p1   /raid1   ext4  defaults  0  0

 下面实现模拟RAID1中/dev/sdc1出现故障，观察/dev/sdc3备用盘自动顶替故障盘

 将/dev/sdc1 指定为故障状态
 >[root@dwj ~]# mdadm -f /dev/md1 /dev/sdc1

 查看阵列状态
```
 [root@dwj ~]# watch -n 1 cat /proc/mdstat
 Every 1.0s: cat /proc/mdstat
 Personalities : [raid0] [raid1]
 md1 : active raid1 sdc3[2] sdc2[1] sdc1[0](F)
 	  1059200 blocks super 1.2 [2/2] [UU]
 md0 : active raid0 sdb2[1] sdb1[0]
 	  2119680 blocks super 1.2 512k chunks
 unused devices:<none>
```
重建成功后，/dev/sdc3后的S消失了，成功顶替故障盘

移除故障盘,重新生成配置文件，防止重启后出现各种问题
>[root@dwj ~]# mdadm -r /dev/md1 /dev/sdc1   <br>
 [root@dwj ~]# mdadm -Ds > /etc/mdadm.conf

* 创建RAID5，添加sde硬盘，分3个1G的主分区sde1和sde2 sde3,和两个逻辑分区sde5,sde6；

```
Device Boot  Start   End   Blocks    Id   System
/dev/sde1        1   132   1060258+  83   Linux
/dev/sde1      133   264   1060290   83   Linux
/dev/sde1      265   396   1060290   83   Linux
/dev/sde1      397   1305  7301542+  83   Extended
/dev/sde1      397   528   1060258+  83   Linux
/dev/sde1      529   1305  6241221   83   Linux
注释：：： Blocks后面+号是表示计算时有了舍入，实际值应小些或大些
```
生成名字为md5的RAID5阵列，等级为5，-n是3个raid，-c是chuck，指定4个硬盘，sde1、sde2、sde3、sde5
```
[root@dwj ~]# mdadm -C -v /dev/md5 -l 5 -n 3 -c 32 -x 1 /dev/sde{1,2,3,5}
Personalities : [raid0] [raid1] [raid6] [raid5] [raid4]
md5 : active raid5 sde3[4] sde5[3](S) sdde2[1] sde1[0]
	  2118400 blocks super 1.2 level 5,32k chunk, algorithm 2 [3/3] [UUU]
md1 : active raid1 sdc3[2] sdc2[1]
	  1059200 blocks super 1.2 [2/2] [UU]
md0 : active raid0 sdb2[1] sdb1[0]
	  2119680 blocks super 1.2 512k chunks
```
生成md5的配置文件，指定配置文件名
>[root@dwj ~]# mdadm -Ds > /etc/mdadm.conf

停止md5
>[root@dwj ~]# mdadm -S /dev/md5

激活md5
>[root@dwj ~]# mdadm -As

给md5进行分区
>[root@dwj ~]# fdisk /dev/md5

格式化md5p1分区
>[root@dwj ~]# mkfs.ext4 /dev/md5p1

创建分区挂载目录
>[root@dwj ~]# mkdir /raid5

把分区挂载到创建的挂载目录上
>[root@dwj ~]# mount /dev/md5p1 /raid5

* 新添加一块硬盘分区/dev/sde6，希望扩展RAID5到4块硬盘

卸载/raid5
>[root@dwj ~]# umount /raid5

查看状态，发现新添加sde6为热备盘
```
[root@dwj ~]# cat /proc/mdstat
Personalities : [raid0] [raid1] [raid6] [raid5] [raid4]
md5 : active raid5 sde6[5](S) sde3[4] sde5[3](S) sdde2[1] sde1[0]
	  2118400 blocks super 1.2 level 5,32k chunk, algorithm 2 [3/3] [UUU]
md1 : active raid1 sdc3[2] sdc2[1]
	  1059200 blocks super 1.2 [2/2] [UU]
md0 : active raid0 sdb2[1] sdb1[0]
	  2119680 blocks super 1.2 512k chunks
```
把sde6扩展为第4块盘，非备份盘
>[root@dwj ~]# mdadm -G /dev/md5 -n 4

重新生成md5 的配置文件，指定配置文件名
>[root@dwj ~]# mdadm -Ds > /etc/mdadm.conf

* 创建RAID1+0双层架构，方法是先创建raid1，然后通过raid1设备创建raid0  <br>
  添加sdf硬盘，分成4个主分区sdf1，sdf2，sdf3，sdf4

首先创建2个raid---md11，md12
>[root@dwj ~]# mdadm -C -v /dev/md11 -l 1 -n 2 /dev/sdf{1,2}  <br>
 [root@dwj ~]# mdadm -C -v /dev/md12 -l 1 -n 2 /dev/sdf{3,4}

创建上层raid---md10
>[root@dwj ~]# mdadm -C -v /dev/md10 -l 0 -n 2 /dev/md11 /dev/md12

查看状态
```
[root@dwj ~]# cat /proc/mdstat
Personalities : [raid0] [raid1] [raid6] [raid5] [raid4]
md10 : active raid0 md12[1] md11[0]
	  2117632 blocks super 1.2 512k chunks
md12 : active raid1 sdf4[1] sdf3[0]
	  1059200 blocks super 1.2 [2/2] [UU]  
md11: active raid1 sdf2[1] sdf1[0]
	  1059200 blocks super 1.2 [2/2] [UU]	  
md5 : active raid5 sde6[5] sde3[4] sde5[3](S) sdde2[1] sde1[0]
	  3177600 blocks super 1.2 level 5,32k chunk, algorithm 2 [4/4] [UUUU]
md1 : active raid1 sdc3[2] sdc2[1]
	  1059200 blocks super 1.2 [2/2] [UU]	  
md0 : active raid0 sdb2[1] sdb1[0]
	  2119680 blocks super 1.2 512k chunks
unused devices: <none>
```
生成md10 的配置文件，指定配置文件名
>[root@dwj ~]# mdadm -Ds > /etc/mdadm.conf

* 停止和删除raid

停止所有raid--其中小s搜索配置文件，命令运行之前需要umount所有挂载目录
>[root@dwj ~]# mdadm -Ss

删除raid配置文件
>[root@dwj ~]# rm -rf /etc/mdadm.conf
