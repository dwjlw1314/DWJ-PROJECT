描述:
```
系统 / 分区下空间不足，用新添加的磁盘空间或者是其它空闲逻辑卷补充
```

总体过程：
```
把/home内容备份，然后将/home文件系统所在的逻辑卷删除，扩大/root文件系统，新建/home ，恢复/home内容
```

1.查看分区
>[root@dwj ~]# df -h

2.备份home分区文件
>[root@dwj ~]# tar cvf /tmp/home.tar /home

3.卸载/home，如果无法卸载，先终止使用/home文件系统的进程
>[root@dwj ~]# fuser -km /home/

>[root@dwj ~]# umount /home

4.删除/home所在的lv
>[root@dwj ~]# lvremove /dev/mapper/centos-home

5.扩展/root所在的lv，增加800G
>[root@dwj ~]# lvextend -L +800G /dev/mapper/centos-root

6.扩展/root文件系统
>[root@dwj ~]# xfs_growfs /dev/mapper/centos-root

7.重新创建home lv
>[root@dwj ~]# lvcreate -L 73G -n /dev/mapper/centos-home

8.创建文件系统
>[root@dwj ~]# mkfs.xfs /dev/mapper/centos-home

9.挂载home
>[root@dwj ~]# mount /dev/mapper/centos-home

10.home文件恢复
>[root@dwj ~]# tar xvf /tmp/home.tar -C /home/

>[root@dwj ~]# cd /home/home/

>[root@dwj ~]# mv * ../
