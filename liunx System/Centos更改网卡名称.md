1.先编辑网卡的配置文件将里面的NAME DEVICE项修改为eth0

>[root@dwj ~]#  vim /etc/sysconfig/network-scripts/ifcfg-eno16777736

![image](https://github.com/dwjlw1314/DWJ-PROJECT/raw/master/PictureSource/1.7.1.png)

2.重命名该配置文件

>[root@dwj ~]#  cd /etc/sysconfig/network-scripts/

>[root@dwj network-scripts]# mv ifcfg-eno16777736 ifcfg-eth0

3.编辑/etc/default/grub并加入"net.ifnames=0 biosdevname=0"

![image](https://github.com/dwjlw1314/DWJ-PROJECT/raw/master/PictureSource/1.7.2.png)

4.运行命令grub2-mkconfig -o /boot/grub2/grub.cfg 来重新生成GRUB配置并更新内核参数。

>[root@dwj network-scripts]# grub2-mkconfig -o /boot/grub2/grub.cfg

![image](https://github.com/dwjlw1314/DWJ-PROJECT/raw/master/PictureSource/1.7.3.png)

5.重新启动机器，启动完之后网卡名称就变成了eth0

>[root@dwj  network-scripts]# init 6
