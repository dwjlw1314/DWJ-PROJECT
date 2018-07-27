创建3个1G的分区sdb1，sdb2，sdb3  ---最新文件系统btrfs

----------文件系统--------  <br>
--------LVM------RAID----  <br>
----------物理硬盘--------

使用分区创建pv

    [root@dwj ~]# pvcreate /dev/sdb{1,2}

创建一个名字为Vg1的卷组，-s指定PE大小，默认4Mib

    [root@dwj ~]# vgcreate -s 16M Vg1 /dev/sdb1 /dev/sdb2

通过Vg1的卷组创建逻辑卷，参数-n是名字，-L 是大小

    [root@dwj ~]# lvcreate  -n LV1 -L 1.5G Vg1

查看创建的LVM信息

    [root@dwj ~]# lvs；pvs；vgs
    [root@dwj ~]# lvscan ；pvs；vgs；
    [root@dwj ~]# lvdisplay；pvdisplay；vgdisplay

格式化LV1，挂载到目录

    [root@dwj ~]# mkfs.ext4 /dev/Vg1/LV1
    [root@dwj ~]# mount /dev/Vg1/LV1 /opt/dwj
    [root@dwj ~]# ls !$

<font color=#FF0000 size=5> <p align="center">扩展LV1和Vg1大小</p></font>
查看Vg1和LV1大小，然后进行LV1扩容   <br>
![image](https://github.com/dwjlw1314/DWJ-PROJECT/raw/master/PictureSource/4.3.1.png)

扩展LV1大小

    [root@dwj ~]# lvextend -L +300M /dev/Vg1/LV1

查看LV1大小和挂载目录大小   <br>
![image](https://github.com/dwjlw1314/DWJ-PROJECT/raw/master/PictureSource/4.3.2.png)

更新挂载目录大小

    [root@dwj ~]# resize2fs /dev/Vg1/LV1

扩展Vg1大小

    [root@dwj ~]# vgextend Vg1 /dev/sdb3

<font color=#FF0000 size=5> <p align="center">缩小LV1大小</p></font>
检查文件系统完整性，前提是没有挂载目录，参数-f是强制

    [root@dwj ~]# e2fsck -f /dev/Vg1/LV1

缩小LV1大小

    [root@dwj ~]# resize2fs /dev/Vg1/LV1 1000M
    [root@dwj ~]# lvreduce -L 1000M /dev/Vg1/LV1

重新挂载目录即可使用

<font color=#FF0000 size=5> <p align="center">缩小Vg1大小</p></font>
说明： VG缩减时，可以不卸载正在使用的LV，另外，只能缩减没有使用的pv，否则提示错误

    [root@dwj ~]# vgreduce Vg1 /dev/sdb3

<font color=#FF0000 size=5> <p align="center">删除LVM</p></font>
先删除lv，然后vg和pv

    [root@dwj ~]# lvremove /dev/Vg1/LV1
    [root@dwj ~]# vgremove Vg1
    [root@dwj ~]# pvremove /dev/sdb3

<font color=#FF0000 size=5> <p align="center">LVM快照</p></font>
前提是需要有LV1，-s 是创建快照参数

    [root@dwj ~]# lvcreate -s -n lv1_sp -L 300M /dev/Vg1/LV1

创建备份文件lv_sp的挂载目录

    [root@dwj opt]# mkdir /tmp/lv1_back

挂载备份文件lv1_sp到目录/tmp/lv1_back

    [root@dwj ~]# mount /dev/Vg1/lv1_sp /tmp/lv1_back/

读取相应的备份文件中的数据进行压缩存档，上述方法是热备

<font color=#FF0000 size=5> <p align="center">磁盘配额</p></font>
使用sdb3分区进行实验,磁盘配额需要quota包的支持

    [root@dwj sdb3]# rpm -q quota
      quota-3.17-20.e16.x86_64
    [root@dwj ~]# mkdir /tmp/sdb3
    [root@dwj ~]# mount /dev/sdb3 /tmp/sdb3/

让磁盘开启配额功能

    [root@dwj sdb3]# mount -o remount,usrquota,grpquota /tmp/sdb3/

检查磁盘配额并生成配额文件，参数如下图：

    [root@dwj /]# setenforce 0   #取消selinux权限管理
    [root@dwj /]# quotacheck -cugv /tmp/sdb3
![image](https://github.com/dwjlw1314/DWJ-PROJECT/raw/master/PictureSource/4.3.3.png)

创建用户和配额设置，参数如下图：

    [root@dwj sdb3]# passwd dwj
    [root@dwj sdb3]# edquota -g dwj
    Disk quotas fro group dwj (gid 502):
    filesystem    blocks   soft   hard   inodes   soft
       hard
     /dev/sdb3       0      50     80       0       0
![image](https://github.com/dwjlw1314/DWJ-PROJECT/raw/master/PictureSource/4.3.4.png)

激活磁盘配额，参数如下图：

    [root@dwj sdb3]# quotaon -ugv /tmp/sdb3
    /dev/sdb3 [tmp/sdb3]: group quotas turned on
    /dev/sdb3 [tmp/sdb3]: user quotas turned on
![image](https://github.com/dwjlw1314/DWJ-PROJECT/raw/master/PictureSource/4.3.5.png)

磁盘配额设置验证：

    [root@dwj sdb3]# mkdir /tmp/sdb3/test
    [root@dwj sdb3]# chmod 777 /tmp/sdb3/test/
    [root@dwj sdb3]# su - dwj
    [dwj@dwj ~]$ cd /tmp/sdb3/test
    添加数据范围的 soft < 1*60 < hard 提示 warnning
    [dwj@dwj ~]$ dd if=/dev/zero  of=dwj.txt bs=1k count=60
    添加数据范围的  1*90 > hard  提示 error
    [dwj@dwj test]$ dd if=/dev/zero  of=dwj.txt bs=1k count=90
