------ rhel-server-6.4-x86_64 系统 --------

常见问题：<br>
530 Permission denied.  <br>
配置文件中userlist_enable=YES（如果启用即YES，则看userlist_deny=YES/NO，如果为NO，则要把登录的用户写入文件/etc/vsftpd/user_list...）;

227 Entering Passive Mode.  <br>
和防火墙设置有关

550 错误 <br>
编辑 /etc/selinux/config , 修改SELINUX=enforcing为SELINUX=disabled

vsftp提供3种远程的登录方式：

* 匿名登录方式

　　就是不需要用户名，密码。就能登录到服务器电脑里面

* 本地用户方式

　　需要帐户名和密码才能登录。而且，这个帐户名和密码，都是在你linux系统里面，已经有的用户。

* 虚拟用户方式

　　同样需要用户名和密码才能登录。但是和上面的区别就是，这个用户名和密码，在你linux系统中是没有的(没有该用户帐号)

1.检查系统是否安装过ftp服务
>[root@dwj /]# rpm -q vsftpd  #如果提示package vsftpd is not install 说明没有安装

2.安装vsftpd包

    [root@dwj /]# rpm -ivh vsftpd-2.2.2-11.el6.x86_64.rpm    #or
    [root@dwj /]# yum -y install vsftpd-2.2.2-11.el6.x86_64.rpm
    上述任意一种都可以进行安装,出现下图表示安装成功
    warning: vsftpd-2.2.2-11.el6_4.1.x86_64.rpm: Header V3 RSA/SHA256 Signature, key ID fd431d51: NOKEY
    Preparing...        ########################################### [100%]
    	package vsftpd-2.2.2-11.el6_4.1.x86_64 is already installed

3.编辑主配置文件 [vsftpd.conf]

    vsftpd.conf: 主配置文件; <br>
    ftpusers:  指定哪些用户不能访问FTP服务器
    user_list: 指定的用户是否可以访问ftp服务器由vsftpd.conf文件中的userlist_deny的取值来决定

>[root@dwj /]# vim /etc/vsftpd/vsftpd.conf   #字段含义
```
    anonymous_enable=NO    (不允许匿名登陆)
    dirmessage_enable=yes （切换目录时，显示目录下.message的内容）
    chroot_local_user=yes （本地所有帐户都只能在自家目录，需要修改用户根目录写权限，chmod a-w /var/ftp/dwj）
    chroot_list_enable=yes （设置指定用户执行chroot，文件中的名单可以跳转目录，可以不设置）
    chroot_list_file=/etc/vsftpd/vsftpd.chroot_list（可以不设置）
    注意：vsftpd.chroot_list 是没有创建的需要自己添加，要想控制帐号就直接在文件中加帐号即可
    userlist_enable=yes (用userlist来限制用户访问，开启后匿名帐号不能登陆)
    userlist_deny=no    (名单中的人允许访问)
    allow_writeable_chroot=yes (允许在用户目录下写权限)
```
--------------------------------------------------------------------------------
>[root@dwj ~]# useradd -s /sbin/nologin -d /var/ftp/dwj dwj   #创建ftp用户，-d是指定目录，-s指定该用户不能返回上一级

添加dwj用户到/etc/vsftpd/user_list文件中，这样就允许访问了

<font color=#FF0000 size=4> <p align="center">多用户不同根目录设置</p></font>

1.创建两个用户 : dwj和lgl

2.修改配置文件

    [root@dwj vsftpd]# vim vsftpd.conf
    chroot_local_user=YES 锁定用户到各自目录为其根目录
    user_config_dir=/etc/vsftpd/userconfig 用户配置目录

3.创建userconfig目录

     [root@dwj vsftpd]# mkdir userconfig
     [root@dwj vsftpd]# cd userconfig/

4.配置各自用户访问根目录

    [root@dwj vsftpd]# vim dwj
        local_root=/var/ftp/ant/
    [root@dwj vsftpd]# vim lgl
         local_root=/var/ftp/oms/

5.重启启动ftp服务

    [root@dwj vsftpd]# /etc/init.d/vsftpd restart

--------------------------------------------------------------------------------

启动/停止/重启

    [root@dwj /]# service vsftpd start
    [root@dwj /]# service vsftpd stop
    [root@dwj /]# service vsftpd restart

查看vsftpd 启动状态

    [root@dwj /]# chkconfig --list vsftpd
    vsftpd  0:off   1:off   2:off   3:off   4:off   5:off   6:off

添加vsftpd自启动

    [root@dwj /]# chkconfig  vsftpd on
    [root@dwj /]# chkconfig --list vsftpd
    vsftpd     0:off   1:off   2:on    3:on    4:on    5:on    6:off

默认情况下从2到5设置为on了。我们也可以加 level 选项来指定

    [root@dwj /]# chkconfig --level 0 vsftpd on
    [root@dwj /]# chkconfig --list vsftpd   
    vsftpd     0:on    1:off   2:on    3:on    4:on    5:on    6:off

我们看到0已经设置为on了。传统的init定义了7个运行级，每一个级别都代表系统应该补充运行的某些特定服务  <br>
0级是完全关闭系统的级别；1级或者S级代表单用户模式；2-5级是多用户级别；6级是重新引导的级别

查看谁登陆了FTP,并杀死它的进程

    [root@dwj /]#  ps –xf |grep ftp
    [root@dwj /]#  kill 进程号

<font color=#FF0000 size=4> <p align="center">脚本安装方式</p></font>
```bash
#! /bin/sh

ftpfile=$(ls | grep vsftpd-*.rpm)

if [ -f "$ftpfile" ]; then

    rpm -ivh ftpfile

    echo "install ftp success"

    echo "config environment start"

	#grant ftp

	chown ftp:ftp /var/ftp  

	chmod 775 /var/ftp

	#edit /etc/vsftpd/vsftpd.conf

	sed -i  's|anonymous_enable=YES|anonymous_enable=NO|g'    /etc/vsftpd/vsftpd.conf

	#set selinux

	#sestatus -b| grep ftp

	setsebool -P  ftpd_full_access on

	setsebool -P ftp_home_dir on

	setenforce 0

    echo "config environment success"

	#restart vsftpd,set vsftpd start onboot

	systemctl restart vsftpd.service

	systemctl enable vsftpd.service

    echo "install success"

fi
```
