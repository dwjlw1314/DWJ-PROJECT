命令格式：mount [-t vfstype] [-o options] device dir

1.[-t vfstype] 指定文件系统的类型，通常不必指定。mount 会自动选择正确的类型。常用类型有：
```
光盘或光盘镜像：iso9660
DOS fat16文件系统：msdos
Windows 9x fat32文件系统：vfat
Windows NT ntfs文件系统：ntfs
Mount Windows文件网络共享：smbfs
UNIX(LINUX) 文件网络共享：nfs
```
2.[-o options] 主要用来描述设备或档案的挂接方式。常用的参数有：

参数名 | 描述
---|---
loop | 用来把一个文件当成硬盘分区挂接上系统
ro | 采用只读方式挂接设备
rw | 采用读写方式挂接设备
iocharset | 指定访问文件系统所用字符集

3.device要挂接(mount)的设备

4.dir设备在系统上的挂接点(mount point)

例子：本地iso镜像挂载：
>[root@dwj /]# mount -o loop /usr/local/dwj.iso /mnt

挂接移动硬盘：USB接口的移动硬盘是当作SCSI设备对待的，插入之前，用下面命令查看系统的硬盘和硬盘分区情况
```
[root@dwj /]# fdisk -l                   #查看系统硬盘挂载情况
[root@dwj /]# parted -l                  #查看系统硬盘挂载情况，与fdisk相似
[root@dwj /]# more /proc/partitions      #查看系统分区简单信息
major   minor   #blocks    name
    8       0   52428800   sda
	  8       1      51200   sda1
	  8       2   51915776   sda2
  253       0   47816704   dm-0
  253       1     409600   dm-1
```
接好移动硬盘后，再用fdisk –l或more /proc/partitions查看系统的硬盘和硬盘分区情况.多了一个SCSI硬盘和它的两个磁盘分区
```
[root@dwj /]# mount -t ntfs /dev/sdc1 /mnt/usbhd1     #磁盘名字以实际情况为准
[root@dwj /]# mount -t vfat /dev/sdc5 /mnt/usbhd2     #磁盘名字以实际情况为准  
注：对ntfs格式的磁盘分区应使用-t ntfs参数，对fat32格式的磁盘分区应使用-t vfat参数
```
若文件名显示为乱码或不显示，可使用下面的命令格式
```
[root@dwj /]# mount -t ntfs -o iocharset=cp936 /dev/sdc1 /mnt/usbhd1    #磁盘名字以实际情况为准  
[root@dwj /]# mount -t vfat -o iocharset=cp936 /dev/sdc5 /mnt/usbhd2    #磁盘名字以实际情况为准
```
挂载U盘：和USB接口的移动硬盘一样对linux系统而言U盘也是当作SCSI设备对待的。使用方法和移动硬盘完全一样
```
[root@dwj /]# mount -t vfat /dev/sdd1 /mnt/usb     #磁盘名字以实际情况为准  
注：现在可以通过/mnt/usb来访问U盘了, 若汉字文件名显示为乱码或不显示，参考硬盘方式处理
```
挂接Windows文件共享：

Windows网络共享的核心是SMB/CIFS，在linux下要挂接磁盘共享，就必须安装和使用samba软件包
当windows系统共享设置好以后，就可以在linux客户端挂接(mount)了，具体操作如下：
```
[root@dwj /]# mkdir –p /mnt/samba      #建立一个目录用来作挂接点(mount point)
[root@dwj /]# mount -t smbfs -o username=administrator,password=q123456. //172.18.8.13/c$ /mnt/samba
注：administrator 和 q123456. 是ip地址为172.18.8.13 windows计算机的一个用户名和密码，c$是这台计算机的一个磁盘共享
```
linux客户端挂接其他linux系统或UNIX系统的NFS共享：
```
[root@dwj /]# mkdir –p /mnt/nfs        #建立一个目录用来作挂接点(mount point)
[root@dwj /]# mount -t nfs -o rw 172.18.8.13:/root/sunky /mnt/nfs
```
注：这里我们假设172.18.8.13是NFS服务端的主机IP地址，当然这里也可以使用主机名，
但必须在本机/etc/hosts文件里增加服务端ip定义；/root/sunky 为服务端共享的目录

把两个文件夹进行同步关联：
```
[root@dwj /]# mount --bind olddir newdir    #把olddir和newdir两个文件夹实现同步读取
注：可以通过--> umount newdir 来取消绑定
```
设置开机自动挂载本地系统镜像文件：
```
[root@dwj /]# vim /etc/fstab      #以下代码实现开机自动挂载
/opt/dwj.iso  /media/cdrom   iso9660    defaults,ro,loop  0 0
备注：iso9660使用df  -T查看设备
```
退出挂载时，须使用umout命令，否则光驱就会一直处于死锁状态：
>[root@dwj /]# umount /mnt/cdrom      #cdrom是假设的挂载点，以实际为准

<font color=#FF0000>linux基本命令中fuser和lsof可以解决无法umount问题</font>  <br>

临时开启分区ACL权限：
```
[root@dwj /]# mount -o remount,acl /  #重新挂载根分区，并挂载加入ACL权限
[root@dwj /]# vim /etc/fstab          #第二种修改ACL权限方式，然后mount或者重启生效
[root@dwj /]# mount -o remount /      #重新挂载已存在的文件系统或重启系统，使修改永久生效    
[root@dwj /]# dumpe2fs -h /dev/sda1   #查看分区是否加入了ACL功能
```
>[root@dwj /]# tune2fs -l /dev/sda1        #与dumpe2fs效果一样

>[root@dwj /]# resize2fs /dev/sda1         #更新sda1的分区大小
