------ rhel-server-6.4-x86_64 系统 --------

<font color=#FF0000 size=4>一、Zabbix简介</font>

Linux下常用的系统监控软件有Nagios、Cacti、Zabbix、Monit等

zabbix是一个基于WEB界面的提供分布式系统监视以及网络监视功能的企业级的开源解决方案。

zabbix由zabbix server与可选组件zabbix agent两部门组成。

zabbix server可以通过SNMP，zabbix agent，ping，IPMI端口监视等方法提供对远程服务器/网络状态的监视。

IPMI：Agent的另一种方式，主要应用于设备的物理性能监控，例如设备的温度、风扇的转速等。

zabbix agent需要安装在被监视的目标服务器上，它主要完成对硬件信息或与操作系统有关的内存，CPU等信息的收集。

Zabbix主要功能：CPU负荷,内存使用,磁盘使用,网络状况,端口监视,日志监视

<font color=#FF0000 size=4>二、软件版本</font>

zabbix的安装需要LAMP或者LNMP环境（Linux，Apache/Nginx，Mysql/MariaDB+Perl/PHP/Python）

1. linux系统使用RedHat-6.4.x86-64；
2. MySQL-5.6.19-1.linux_glibc2.5.x86_64.rpm-bundle.tar；
3. zabbix-3.2.4.tar.gz；
4. httpd-2.2.32.tar.bz；
5. php-7.1.2.tar.gz

<font color=#FF0000 size=4>三、Zabbix软件安装配置</font>

    增加zabbix用户和组
    [root@dwj ~]# groupadd zabbix
    [root@dwj ~]# useradd-g zabbix zabbix

mysql数据库安装步骤:  <br>
1.查找以前是否装有mysql  <br>
>[root@dwj ~]# rpm -qa|grep mysql

如果提示如下相关信息,需要先删除mysql,如果没有则跳过后面步骤2

    mysql-libs-5.1.66-2.e16_3.x86_64
    mysql-5.1.66-2.e16_3.x86_64
    mysql-devel-5.1.66-2.e16_3.x86_64

2.删除已经安装的3个mysql程序、以及开发头文件、库、用户以及配置文件   <br>
>[root@dwj ~]# rpm -ev --nodeps mysql-libs-5.1.66-2.el6_3.x86_64  <br>
>[root@dwj ~]# rpm -ev --nodeps mysql-libs-5.1.66-*    #匹配删除

    检查各个mysql文件夹是否删除干净
    [root@dwj ~]# find / -name mysql
    /usr/lib64/mysql
    [root@dwj ~]# rm -rf /usr/lib64/mysql/
    删除var和etc下的文件
    [root@dwj ~]# rm -rf /etc/my.cnf
    [root@dwj ~]# rm -rf /var/lib/mysql
    删除mysql用户及用户组
    [root@dwj ~]# userdel mysql
    [root@dwj ~]# groupdel mysql

3.安装RPM格式mysql包

    [root@dwj software]# rpm -ivh MySQL-server-5.6.19-1.el6.x86_64.rpm
    [root@dwj software]# rpm -ivh MySQL-client-5.6.19-1.linux_glibc2.5.x86_64.rpm
    [root@dwj software]# rpm -ivh MySQL-devel-5.6.19-1.linux_glibc2.5.x86_64.rpm
    [root@dwj software]# rpm -ivh MySQL-shared-5.6.19-1.linux_glibc2.5.x86_64.rpm
    [root@dwj software]# rpm -ivh MySQL-shared-compat-5.6.19-1.linux_glibc2.5.x86_64.rpm

4.将MySQL的配置文件拷贝到/etc目录下

    [root@dwj software]# cp /usr/share/mysql/my-default.cnf /etc/my.cnf

5.初始化MySQL及设置密码

    [root@dwj software]# /usr/bin/mysql_install_db         #初始化MySQL
    [root@dwj mysql]# service mysql start                  #启动MySQL
    [root@dwj mysql]# cat /root/.mysql_secret              #查看root账号的初始密码，如图
    [root@dwj software]# mysql -u root -psAeqiIooBuVnRG4d  #登录MySQL
    mysql> set password=password('123456');                #修改密码

6.设置开机启动

    [root@dwj software]# chkconfig mysql on
    chkconfig --list |grep mysql
    mysql  0:off  1:off   2:on   3:on   4:on   5:on   6 off
    上面打印出来的内容中，2~5为on就是开机启动了

7.MySQL的默认安装位置

   /var/lib/mysql/       #数据库目录
   /usr/share/mysql      #配置文件目录
   /usr/bin              #相关命令目录
   /etc/init.d/mysql     #启动脚本

8.创建 /var/lib/mysql/data/ 目录

9.修改字符集和数据存储路径  <br>
配置/etc/my.cnf文件，设置MySQL的字符集，配置MySQL表明不区分大小写，在[mysqld]下面加入如下内容：
```
   character_set_server=utf8  ；  character_set_client=utf8  ；  collation-server=utf8_general_ci
   lower_case_table_names=1          #表名区分大小写，列名不区分大小写；0：区分大小写，1：不区分大小写
   max_connections=1000              #设置最大连接数，默认为151，MySQL服务器允许的最大连接数16384;
   port = 3306
   server_id = 2
   user=root
   password=123456
   # datadir = /var/lib/mysql/data                  #自定义目录，可以不修改
   # socket = /var/lib/mysql/data/mysql.sock        #自定义目录，可以不修改
   # pid-file =/var/lib/mysql/data/mysqld.pid       #自定义目录，可以不修改
```
10.重新启动数据库

    [root@dwj mysql]# service mysql restart

11.登录mysql，查看修改后的字符集
![image](https://github.com/dwjlw1314/DWJ-PROJECT/raw/master/PictureSource/3.10.1.png)


Apache服务器安装步骤:   <br>
1.源码包解压，编译并安装 Apache

    [root@dwj software]# tar -jxvf httpd-2.2.32.tar.bz2
    [root@dwj httpd-2.2.32]# ./configure --enable-so
    [root@dwj httpd-2.2.32]# make && make install

2.修改apahce配置文件

    [root@dwj ~]# vim /usr/local/apache2/conf/httpd.conf
    修改ServerName字段
    ServerName dwj      #dwj 是linux系统主机名，或者是ip地址

3.启动apached服务器

    [root@dwj ~]# /usr/local/apache2/bin/apachectl start

![image](https://github.com/dwjlw1314/DWJ-PROJECT/raw/master/PictureSource/3.10.2.png)

4.验证是否成功  <br>
![image](https://github.com/dwjlw1314/DWJ-PROJECT/raw/master/PictureSource/3.10.3.png)

PHP服务器安装步骤:   <br>
需要安装相关依赖包： freetype，libmcrypt，curl，jpegsrc

1.源码包解压，编译并安装 PHP

    [root@dwj software]# tar -zxvf php-7.1.2.tar.gz
    [root@dwj php-7.1.2]# mkdir /usr/local/php          #创建目录
    [root@dwj php-7.1.2]# ./configure --prefix=/usr/local/php --enable-fpm --with-zlib --enable-mbstring  --with-mysqli --with-mysql-sock --with-gd --enable-gd-native-ttf --enable-pdo --with-pdo-mysql --with-gettext  --with-pdo-mysql --enable-sockets --enable-bcmath --enable-xml --with-bz2 --with-freetype-dir=/usr/local/freetype/ --with-libxml-dir --with-zlib-dir --with-curl=/usr/local/curl --with-jpeg-dir=/usr/local/jpeg --with-png-dir --with-mcrypt=/usr/local/libmcrypt --with-apxs2=/usr/local/apache2/bin/apxs --with-config-file-path=/usr/local/lib/
    [root@dwj php-7.1.2]# make && make install

2.修改php配置

    [root@dwj php-7.1.2]# cp php.ini-development /usr/local/lib/php.ini   #复制php.ini-development到 lib 下，并改名为php.ini
    [root@dwj php-7.1.2]# vim /usr/local/lib/php.ini                      #修改如下内容
    extension_dir = "/usr/local/php/lib/php/extensions/"
    去掉extension=php_curl.dll，extension=php_gd2.dll，extension=php_mbstring.dll，extension=php_mysql.dll，extension=php_mysqli.dll，extension=php_pdo_mysql.dll，extension=php_xmlrpc.dll前面 的分号
    short_open_tag = Off 把它修改成short_open_tag = On，让其支持短标签

3.修改apache配置文件

    [root@dwj php]# vim /usr/local/apache2/conf/httpd.conf         #修改以下内容
    LoadModule php7_module        modules/libphp7.so               #libphp7.so是在php源码目录下的libs
    User zabbix
    Group zabbix
    ServerName 192.168.120.130            #本机ip地址
    将所有出现的 Deny 替换为 Allow

>[root@dwj php]# Include conf/extra/httpd-vhosts.conf

    =====================新增======================
    <FilesMatch "\.ph(p[2-6]?|tml)$">
    SetHandler application/x-httpd-php
    Allow from all
    </FilesMatch>
    <FilesMatch "\.phps$">
    SetHandler application/x-httpd-php-source
    Allow from all
    </FilesMatch>
    ===============================================

>[root@dwj php]# vim /usr/local/apache2/conf/extra/httpd-vhosts.conf    #打开并修改一下内容
![image](https://github.com/dwjlw1314/DWJ-PROJECT/raw/master/PictureSource/3.10.4.png)

4.在apache根文件中增加一个test.php文件，<?php phpinfo(); ?>;然后访问http://localhost/test.php,查看结果

5.重新启动apache服务器

    [root@dwj php]# /usr/local/apache2/bin/apachectl restart

6.验证是否正确
![image](https://github.com/dwjlw1314/DWJ-PROJECT/raw/master/PictureSource/3.10.5.png)

7.源码包编译ext扩展库(gettext；mbstring；gd)

    [root@dwj php-7.1.2]# cd ext/gettext/
    [root@dwj gettext]# ./configure --with-php-config=/usr/local/php/bin/php-config
    [root@dwj gettext]# make & make install

zabbix服务器安装步骤:  <br>
安装相关依赖包 ： libcurl，net-snmp

1.zabbix 数据库设置，登录数据库，创建帐号和设置权限

    [root@dwj ~]# mysql -uroot -p12345
    mysql> use mysql;
    mysql>create database zabbix character set utf8;
    mysql>grant all privileges on zabbix.* to zabbix@'localhost' identified by '123456';
    mysql>grant all privileges on zabbix.* to zabbix@'10.34.120.134' identified by '123456';
    mysql>grant all privileges on zabbix.* to zabbix@'%' identified by '123456';
    mysql>flush privileges;    # 刷新系统授权表
    mysql>quit

2.导入数据库表

    [root@dwj zabbix-3.2.4]# cd database/mysql
    #mysql -uroot -p123456 zabbix<schema.sql
    #mysql -uroot -p123456 zabbix<images.sql
    #mysql -uroot -p123456 zabbix<data.sql

3.创建安装目录

    [root@dwj database]# mkdir /usr/local/zabbix

4.编译安装zabbix

    [root@dwj zabbix-3.2.4]# ./configure --prefix=/usr/local/zabbix --with-mysql --with-net-snmp --with-libcurl
                              --enable-server --enable-agent --enable-proxy
    [root@dwj zabbix-3.2.4]# make & make install

5.添加服务端口，如果已经存在，可以跳过

    [root@dwj  zabbix-3.2.4]# vim /etc/services  #内容如下：
    zabbix-agent 10050/tcp # Zabbix Agent
    zabbix-agent 10050/udp # Zabbix Agent
    zabbix-trapper 10051/tcp # Zabbix Trapper
    zabbix-trapper 10051/udp # Zabbix Trapper

6.修改server配置文件，修改以下数据
>[root@dwj etc]# vim /usr/local/zabbix/etc/zabbix_server.conf

    LogFile=/tmp/zabbix_server.log
    PidFile=/tmp/zabbix_server.pid
    DBHost=localhost
    DBName=zabbix
    DBUser=zabbix
    DBSocket=/var/lib/mysql/mysql.sock
    DBPassword=123456          #指定zabbix数据库密码
    AlertScriptsPath=/usr/local/zabbix/share/zabbix/alertscripts

7.修改Agentd配置文件，修改以下数据
>[root@dwj etc]# vim /usr/local/zabbix/etc/zabbix_agentd.conf

    PidFile=/tmp/zabbix_agentd.pid         #进程PID
    LogFile=/tmp/zabbix_agentd.log         #日志保存位置
    EnableRemoteCommands=1                 #允许执行远程命令
    Server=192.168.1.4，127.0.0.1          #被动工作模式，zabbix server端的ip，可以有多个ip1，ip2
    Hostname=dwj                           #必须与zabbix创建的host name相同，服务端界面添加
    ServerActive=192.168.1.4:10051         #主动工作模式，zabbix server端ip和端口
    StartAgents=8                          #启动的客户端进程数
    Timeout=30                             #超时时间

    UnsafeUserParameters=1                 #自定义的时候可以添加特殊字符
    #UserParameter=count.line.passwd,wc -l /etc/passwd|awk '{print $1}'       #自定义监控脚本，需要重启agent
    使用格式：Format: UserParameter=<key>,<shell command>

验证命令：/usr/local/zabbix/bin/zabbix_get -s 127.0.0.1 -k count.line.passwd  <br>
#Include=/usr/local/zabbix/etc/zabbix_agentd.conf.d/       #自定义配置文件路径，与 自定义监控脚本是两种方式

8.添加web前段php文件

    [root@dwj zabbix-3.2.4]# cd frontends
    [root@dwj frontends]# cp -rf php/* /usr/local/apache2/htdocs/zabbix/
    [root@dwj frontends]# chown -R zabbix:zabbix /usr/local/apache2/htdocs/zabbix/

9.添加开机启动脚本---生产环境使用

    [root@dwj zabbix-3.2.4]# cp misc/init.d/fedora/core/zabbix_server /etc/rc.d/init.d/zabbix_server   #服务端
    [root@dwj zabbix-3.2.4]# cp misc/init.d/fedora/core/zabbix_agentd /etc/rc.d/init.d/zabbix_agentd   #客户端
    [root@dwj zabbix-3.2.4]# chmod +x /etc/rc.d/init.d/zabbix_server    #添加脚本执行权限
    [root@dwj zabbix-3.2.4]# chmod +x /etc/rc.d/init.d/zabbix_agentd    #添加脚本执行权限
    [root@dwj zabbix-3.2.4]# chkconfig zabbix_server on                 #添加开机启动
    [root@dwj zabbix-3.2.4]# chkconfig zabbix_agentd on                 #添加开机启动

修改zabbix开机启动脚本中的zabbix安装目录
>[root@dwj zabbix-3.2.4]# vim /etc/rc.d/init.d/zabbix_server          #编辑服务端配置文件

    BASEDIR=/usr/local/zabbix/         #zabbix安装目录

>[root@dwj zabbix-3.2.4]# vim /etc/rc.d/init.d/zabbix_agentd       #编辑客户端配置文件

    BASEDIR=/usr/local/zabbix/         #zabbix安装目录

10.启动服务器

    [root@dwj zabbix-3.2.4]# service zabbix_server start     #启动zabbix服务端
    [root@dwj zabbix-3.2.4]# service zabbix_agentd start     #启动zabbix客户端
    [root@dwj zabbix-3.2.4]# service iptables stop           #关闭防火墙

使用方法：

1.在浏览器中输入  http://localhost/zabbix/setup.php
![image](https://github.com/dwjlw1314/DWJ-PROJECT/raw/master/PictureSource/3.10.6.png)

提示php配置文件参数不符的字段进行修改

    date.timezone = Asia/Shanghai
    max_execution_time = 300
    max_input_time = 300
    post_max_size = 16M

2.重新启动apache服务器

    [root@dwj ~]# /usr/local/apache2/bin/apachectl restart

3.选择数据库参数
![image](https://github.com/dwjlw1314/DWJ-PROJECT/raw/master/PictureSource/3.10.7.png)

4.选择zabbix服务器ip和端口，选择完成即可

5.http://localhost/zabbix/index.php   使用账号和密码进行登录    账号：admin ；密码：zabbix
![image](https://github.com/dwjlw1314/DWJ-PROJECT/raw/master/PictureSource/3.10.8.png)

<font color=#FF0000 size=4>zabbix_server.conf参数含义</font>
```
DBName=zabbix                       zabbix所属数据库名称
DBUser=zabbix                       zabbix所属数据库用户
DBPassword=123456                   zabbix数据库密码
StartPollers=30    　               轮询的初始值（0-1000）
StartIPMIPollers=4                  IPMI轮询的初始值（0-1000）
StartPollersUnreachable=30          轮询不可达的主机数（包括IPMI 0-1000）
StartTrappers=8     　              捕获的初始值（0-1000）
StartPingers=4                      ping的初始值（0-1000）
StartDiscoverers=0                  自动发现的初始值（0-250）
CacheSize=384M                      缓存大小
CacheUpdateFrequency=300            缓存更新的频率
StartDBSyncers=8                    数据库同步时间
TrendCacheSize=128M                 总趋势缓存大小
AlertScriptsPath=/usr/bin           脚本的存放位置
LogSlowQueries=1000                 日志慢查询设定
```
