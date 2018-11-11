一． 在VMware设置中添加或者扩展硬盘，或者挂载一块新物理硬盘

如果是虚拟机设置，可以点击添加一个硬盘一直下一步就可以，也可以选择扩展输入想扩展后的硬盘大小点击扩展就可以

![image](https://github.com/dwjlw1314/DWJ-PROJECT/raw/master/PictureSource/1.17.1.jpg)

二． 在linux系统中创建分区和进行格式化

首先fdisk –l命令查看现在的分区情况
>[root@dwj ~]# fdisk -l
```
需要特别注意对于扩展分区的话如果你sda3没有都用的话，再建立的新分区sda3大小只能为1M。
所以这时需要创建一个sda3新分区后再创建一个分区sda4才可以完全利用剩下的你刚刚扩展的硬盘空间。
这是因为磁盘分区采用的mbr模式，sda1,2,3为主分区大小地址有限制
```

创建分区，依次键入 n,p,w
>[root@dwj ~]# fdisk /dev/sda         #扩展硬盘输入  <br>
>[root@dwj ~]# fdisk /dev/sdb         #添加硬盘输入

重启linux，对新建分区格式化
>[root@dwj ~]# mkfs.ext4 /dev/sda4     #扩展硬盘的方法  <br>
>[root@dwj ~]# mkfs.ext4 /dev/sdb1     #添加新硬盘的方法

挂载分区,在根目录下创建新disk目录
>[root@dwj ~]# mkdir /disk              <br>
>[root@dwj ~]# mount /dev/sda4 /disk    #将新分区mount到/disk目录上

设置开机启动挂载，添加如下内容
>[root@dwj ~]# vim /etc/fstab
/dev/sdb1   /oracle   ext4    defaults   0 0
