
GRUB的密码设置可分为全局密码和菜单密码，为了防止他人通过GRUB修改root密码大家需要设置一个全局密码,在splashimage这个参数的下一行可以加上password=密码，保存后重新启动计算机，再次登录到GRUB菜单页面的时候就会发现，这时已经不能直接使用e命令编辑启动标签了，须先使用p命令，输入正确的密码后才能够对启动标签进行编辑

虽然我们设置了全局密码，但是如果他人得到了全局密码后仍然可以修改GRUB启动标签从而修改root密码；这样我们就可以设置菜单密码，设置菜单密码也非常简单，我们只需要在title的下一行加上password --md5 PASSWORD，然后保存退出。这样即使有了全局密码也必需输入菜单密码才能够引导系统

此外，如果直接对GRUB进行明文加密也是非常不安全的，所以就要使用MD5对其进行加密。在终端中输入grub-md5-crypt回车，这时系统会要求输入两次相同的密码，之后系统便会输出MD5码。然后在按照password --md5 MD5密文这个格式设置全局或者菜单密码，保存退出，重启计算机即可

1、计算grub的MD5加密密码：
```
[root@dwj ~]# grub-md5-crypt
Password:
Retype password:
    输入两遍密码进行确认以后，就会计算出你所输入密码的MD5加密值，如：$1$iBmEv/$yTgeFNcwYvOdaLqO2gFz10
```
2、设置grub密码：
```
[root@dwj ~]# vim /boot/grub/grub.conf    #文件的内容大概如下所示：
default=0
timeout=5
splashimage=(hd0,0)/grub/splash.xpm.gz
hiddenmenu
title Red Hat Enterprise Linux (2.6.32-431.el6.x86_64)
    root (hd0,0)
    kernel /vmlinuz-2.6.32-431.el6.x86_64 ro root=/dev/mapper/vg_dwj-lv_root rd_NO_LUKS rd_LVM_LV=vg_dwj/lv_swap LANG=en_US.UTF-8 rd_NO_MD rd_LVM_LV=vg_dwj/lv_root SYSFONT=latarcyrheb-sun16 crashkernel=128M  KEYBOARDTYPE=pc KEYTABLE=us rd_NO_DM rhgb quiet
    initrd /initramfs-2.6.32-431.el6.x86_64.img
#更改下面的实例:
default=0
timeout=5
password --md5 $1$iBmEv/$yTgeFNcwYvOdaLqO2gFz10
splashimage=(hd0,0)/grub/splash.xpm.gz
hiddenmenu
title Red Hat Enterprise Linux (2.6.32-431.el6.x86_64)
    root (hd0,0)
```
使用password,lock命令实现几种加密方法如下：
```
标注： ^^^^^其中lock部分系统可以不加^^^^^
1、单纯对GRUB界面加密，而不对被引导的系统加密在timeout一行下面加一行：password --md5 PASSWORD
2、对GRUB界面加密，同时对被引导的系统加密在timeout一行下面加一行：password --md5 PASSWORD 在title一行下面加一行：lock
  在lock一行下面紧贴着再加一行：password --md5 PASSWORD 注：lock不能单独使用
3、同时存在多个被引导系统，针对特定的系统实例分别加密(未对GRUB操作界面加密) 在title一行下面加一行：lock
  在lock一行下面紧贴着再加一行：password --md5 PASSWORD
```
3、解密，当Linux系统管理员密码忘记，又不能编辑grub菜单进行单用户模式重置密码，可以对grub进行解密，步骤如下:
```
1、用Linux第一张安装光盘引导系统，进入修复模式
2、光盘引导系统，选择区域和语言，不用启动网卡
3、出现Rescue菜单，选择Continue，再弹出一个Rescue菜单，会显示“chroot /mnt/sysimage”，OK
4、[root@dwj ~]# chroot /mnt/sysimage，若不执行此步，进系统不能进行改写
5、执行passwd对用户密码进行重置，完后重启正常进入系统
6、编辑/boot/grub/grub.conf，去掉“password --md5 $1$iBmEv/$yTgeFNcwYvOdaLqO2gFz10”这一行即可
```
使用password,lock命令实现几种加密方法如下：
* 单纯对GRUB界面加密，而不对被引导的系统加密在timeout一行下面加一行：password --md5 PASSWORD
* 对GRUB界面加密，同时对被引导的系统加密在timeout一行下面加一行：password --md5 PASSWORD 在title一行下面加一行：lock  <br>
  在lock一行下面紧贴着再加一行：password --md5 PASSWORD 注：lock不能单独使用
* 同时存在多个被引导系统，针对特定的系统实例分别加密(未对GRUB操作界面加密) 在title一行下面加一行：lock  <br>
  在lock一行下面紧贴着再加一行：password --md5 PASSWORD

3、解密，当Linux系统管理员密码忘记，又不能编辑grub菜单进行单用户模式重置密码，怎么办呢，只有对grub进行解密了

    1、用Linux第一张安装光盘引导系统，进入修复模式
    2、光盘引导系统，选择区域和语言，不用启动网卡
    3、出现Rescue菜单，选择Continue，再弹出一个Rescue菜单，会显示“chroot /mnt/sysimage”，OK
    4、[root@dwj ~]# chroot /mnt/sysimage，若不执行此步，进系统不能进行改写
    5、执行passwd对用户密码进行重置，完后重启正常进入系统
    6、编辑/boot/grub/grub.conf，去掉“password --md5 $1$iBmEv/$yTgeFNcwYvOdaLqO2gFz10”这一行即可
