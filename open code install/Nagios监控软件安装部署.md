------ rhel-server-6.4-x86_64 系统 --------

<font color=#FF0000 size=5>一、Nagios简介</font> <br>

1.主要功能

网络服务监控（SMTP、POP3、HTTP、NNTP、ICMP、SNMP、FTP、SSH）

主机资源监控（CPU load、disk usage、system logs），也包括Windows主机（使用NSClient++ plugin）

可以指定自己编写的Plugin通过网络收集数据来监控任何情况（温度、警告……）

可以通过配置Nagios远程执行插件远程执行脚本

远程监控支持SSH或SSL加通道方式进行监控

简单的plugin设计允许用户很容易的开发自己需要的检查服务，支持很多开发语言（shell scripts、C++、Perl、ruby、Python、PHP、C#等）

包含很多图形化数据Plugins（Nagiosgraph、Nagiosgrapher、PNP4Nagios等）

可并行服务检查

能够定义网络主机的层次，允许逐级检查，就是从父主机开始向下检查

当服务或主机出现问题时发出通告，可通过email, pager, sms 或任意用户自定义的plugin进行通知

能够自定义事件处理机制重新激活出问题的服务或主机

自动日志循环

支持冗余监控

包括Web界面可以查看当前网络状态，通知，问题历史，日志文件等

2.Nagios工作原理

Nagios的功能是监控服务和主机，但是他自身并不包括这部分功能，所有的监控、检测功能都是通过各种插件来完成的

启动Nagios后，它会周期性的自动调用插件去检测服务器状态，同时Nagios会维持一个队列，所有插件返回来的状态信息都进入队列，Nagios每次都从队首开始读取信息，并进行处理后，把状态结果通过web显示出来

在nagios主目录下的/libexec里放有自带的可使用的所有插件，如check_disk是检查磁盘空间的插件，check_load是检查CPU负载的。每一个插件可以通过运行./check_xxx –h 来查看其使用方法和功能

Nagios可以识别4种状态返回信息，即 0(OK)表示状态正常/绿色、1(WARNING)表示出现警告/黄色、2(CRITICAL)表示出现非常严重的错误/红色、3(UNKNOWN)表示未知错误/深黄色。Nagios根据插件返回来的值，来判断监控对象的状态，并通过web显示出来，以供管理员及时发现故障
![image](https://github.com/dwjlw1314/DWJ-PROJECT/raw/master/PictureSource/3.11.1.jpg)

Nagios 安装，是指基本平台，也就是Nagios软件包的安装。它是监控体系的框架，也是所有监控的基础

Nagios 系统提供了一个插件NRPE。Nagios 通过周期性的运行它来获得远端服务器的各种状态信息。它们之间的关系如下图所示：
![image](https://github.com/dwjlw1314/DWJ-PROJECT/raw/master/PictureSource/3.11.2.jpg)

Nagios 通过NRPE 来远端管理服务

1.Nagios 执行安装在它里面的check_nrpe 插件，并告诉check_nrpe 去检测哪些服务

2.通过SSL，check_nrpe 连接远端机子上的NRPE daemon

3.NRPE 运行本地的各种插件去检测本地的服务和状态(check_disk,..etc)

4.最后，NRPE 把检测的结果传给主机端的check_nrpe，check_nrpe 再把结果送到Nagios状态队列中

5.Nagios 依次读取队列中的信息，再把结果显示出来

<font color=#FF0000 size=5>二、部署测试环境</font>  <br>

Host Name | OS | IP | Software
:--|:--|:--|:--
Nagios-Server | RedHat6.4 X86-64 | 172.16.10.50 | Apache、Php、Nagios、nagios-plugins
Nagios-Linux  | RedHat6.4 X86-64 | 172.16.10.51 | nagios-plugins、nrpe
Nagios-Windows| Windows 7        | 172.16.10.52 | NSClient++

Server 安装了nagios软件，对监控的数据做处理，并且提供web界面查看和管理。当然也可以对本机自身的信息进行监控。

Client 安装了NRPE等客户端，根据监控机的请求执行监控，然后将结果回传给监控机。

防火墙已关闭/iptables: Firewall is not running.

/etc/selinux/config文件字段设置 SELINUX=disabled

测试环境主机实现监控功能：
![image](https://github.com/dwjlw1314/DWJ-PROJECT/raw/master/PictureSource/3.11.3.jpg)

<font color=#FF0000 size=5>三、Nagios软件安装配置</font>  <br>

安装包版本：nagios-4.2.4.tar.gz；nagios-plugins-2.1.4.tar.gz

1.安装相关依赖包

    查找相关依赖包是否安装
    [root@dwj ~]# rpm -q gcc glibc glibc-common gd gd-devel xinetd openssl-devel

    如果系统中没有，使用yum进行安装，gd-devel需要单独进行安装
    [root@dwj ~]# yum -y install gd xinetd xinetd

2.创建nagios用户和用户组，已经程序存放的目录

    [root@dwj ~]# useradd -s /sbin/nologin nagios
    [root@dwj ~]# mkdir /usr/local/nagios
    [root@dwj ~]# chown -R nagios.nagios /usr/local/nagios/
    查看nagios目录的权限是否正确
    [root@dwj Desktop]# ll -d /usr/local/nagios/
    drwxr-xr-x. 2 nagios nagios 4096 Mar 17 02:16 /usr/local/nagios

3.编译安装nagios

    解压软件包后进入相应目录
    [root@dwj Desktop]# tar -zxvf nagios-4.2.4.tar.gz
    [root@dwj Desktop]# cd nagios-4.2.4
    [root@dwj nagios-4.2.4]# ./configure --prefix=/usr/local/nagios/ --with-command-group=nagios
![image](https://github.com/dwjlw1314/DWJ-PROJECT/raw/master/PictureSource/3.11.4.png)

    [root@dwj nagios-4.2.4]# make all
![image](https://github.com/dwjlw1314/DWJ-PROJECT/raw/master/PictureSource/3.11.5.png)
![image](https://github.com/dwjlw1314/DWJ-PROJECT/raw/master/PictureSource/3.11.6.png)

    [root@dwj nagios-4.2.4]# make install
![image](https://github.com/dwjlw1314/DWJ-PROJECT/raw/master/PictureSource/3.11.7.png)

    [root@dwj nagios-4.2.4]# make install-init
![image](https://github.com/dwjlw1314/DWJ-PROJECT/raw/master/PictureSource/3.11.8.png)

    [root@dwj nagios-4.2.4]# make install-commandmode
![image](https://github.com/dwjlw1314/DWJ-PROJECT/raw/master/PictureSource/3.11.9.png)

    [root@dwj nagios-4.2.4]# make install-config
![image](https://github.com/dwjlw1314/DWJ-PROJECT/raw/master/PictureSource/3.11.10.png)

    [root@dwj nagios-4.2.4]# make install-webconf
![image](https://github.com/dwjlw1314/DWJ-PROJECT/raw/master/PictureSource/3.11.11.png)

4.设置系统自启动

    [root@dwj nagios-4.2.4]# chkconfig --add nagios
    [root@dwj nagios-4.2.4]# chkconfig --level 35 nagios on
    [root@dwj nagios-4.2.4]# chkconfig --list nagios
    nagios  0:off  1:off   2:on   3:on   4:on   5:on   6 off

5.验证程序是否被正确安装   <br>
    切换目录到安装路径/usr/local/nagios，看是否存在以下目录，如果存在则表明程序安装成功

Directory | meaning
:--|-:-
bin   | Nagios 可执行程序所在目录
etc   | Nagios 配置文件所在目录
sbin  | Nagios CGI 文件所在目录，也就是执行外部命令所需文件所在的目录
share | Nagios网页文件所在的目录
libexec | Nagios 外部插件所在目录
var   | Nagios 日志文件、lock 等文件所在的目录
var/archives | Nagios 日志自动归档目录
var/rw | 用来存放外部命令文件的目录

6.安装nagios插件

    解压软件包后进入相应目录
    [root@dwj Desktop]# tar -zxvf nagios-plugins-2.1.4.tar.gz
    [root@dwj Desktop]# cd nagios-plugins-2.1.4
    [root@dwj nagios-plugins-2.1.4]# ./configure --prefix=/usr/local/nagios/
    [root@dwj nagios-plugins-2.1.4]# make && make install

7.安装Apache和Php

    参照zabbix监控软件安装部署的安装方式即可

8.配置apache，打开的配置文件/usr/local/apache2/conf/httpd.conf 修改如下内容：

    User nagios
    Group nagios
    ServerName dwj            #主机名
    <IfModule dir_module>
    　　    DirectoryIndex index.html index.php
    </IfModule>

    #添加如下内容：
    AddType application/x-httpd-php .php

    #为了安全起见，让nagios 的web 监控页面经过授权才能访问，在httpd.conf 文件最后添加如下信息：

    #setting for nagios

    ScriptAlias /nagios/cgi-bin "/usr/local/nagios/sbin"

    <Directory "/usr/local/nagios/sbin">

         AuthType Basic

         Options ExecCGI

         AllowOverride None

         Order allow,deny

         Allow from all

         AuthName "Nagios Access"

         AuthUserFile /usr/local/nagios/etc/htpasswd

         Require valid-user

    </Directory>

    Alias /nagios "/usr/local/nagios/share"

    <Directory "/usr/local/nagios/share">

         AuthType Basic

         Options None

         AllowOverride None

         Order allow,deny

         Allow from all

         AuthName "nagios Access"

         AuthUserFile /usr/local/nagios/etc/htpasswd

         Require valid-user

    </Directory>

创建Apache的目录验证文件

[root@dwj apache2]# /usr/local/apache2/bin/htpasswd -c /usr/local/nagios/etc/htpasswd dwj

通过http://localhost/nagios/ 访问时就需要输入用户名和密码了

查看认证文件内容

[root@dwj apache2]# cat /usr/local/nagios/etc/htpasswd

    david: $apr1$XKC9h8iu$xHOXsJrDQwqlLhsFuthIu/

启动Apache服务

[root@dwj apache2]# /usr/local/apache2/bin/apachectl start

到这里nagios 的安装也就基本完成了，你可以通过web来访问了

9.配置可参考官网，下面是引用别人的配置数据  <br>
http://www.cnblogs.com/mchina/archive/2013/02/20/2883404.html
