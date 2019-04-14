ClamAV是一个C语言开发的开源病毒扫描工具，用于检测木马，病毒，恶意软件和其他恶意威胁的一个开源杀毒引擎。可以在线更新病毒库，Linux系统的病毒较少，但是对于诸如邮件或者归档文件中夹杂的病毒往往难以防范，而ClamAV则能起到防护作用

ClamAV功能特性

项目 | 详细
---|---
主要用途 | 邮件网关的病毒扫描，内建支持多种邮件格式
高性能   | 提供多线程的扫描进程
命令行   | 提供命令行扫描方式
扫描对象 | 可以对要发送的邮件或者文件进行扫描
文件格式 | 支持多种文件格式
病毒库更新频度 | 多次进行病毒库更新
归档文件 | 支持扫描多种归档文件,比如ZIP、RAR、DMG、TAR、OLE2、Cabinet、CHM、BinHex、SIS等
文档 | 支持流行的文档文件，比如：MS Office文、MacOffice文件、HTML、Flash、RT、PDF

ClamAV的安装环境
```
环境：Linux RedHat_6.4
版本：clamav-0.101.2.tar.gz
依赖：pcre* zlib* libssl* openssl*
```
ClamAV的源码编译安装

解压clamav源码包
>[root@dwj Desktop]# tar -xvf clamav-0.101.2.tar.gz

进入解压目录运行configure命令
>[root@dwj clamav-0.101.2]# ./configure --prefix=/usr/local/clamav --with-pcre

编译和链接
>[root@dwj clamav-0.101.2]# make && make install

系统环境变量添加可执行程序目录
>[root@dwj clamav]# echo 'PATH=$PATH:/usr/local/clamav/bin' >> /etc/profile  <br>
>[root@dwj clamav]# echo 'export PATH' >> /etc/profile

查看版本验证是否正确安装
>[root@dwj clamav]# clamscan -V

扫描配置文件clamd.conf导出，注释掉'Example'这一行
>[root@dwj clamav]# clamconf -g clamd.conf > /usr/local/clamav/etc/clamd.conf

病毒库配置文件freshclam.conf导出，注释掉'Example'这一行
>[root@dwj clamav]# clamconf -g freshclam.conf > /usr/local/clamav/etc/freshclam.conf

创建用户
>[root@dwj etc]# useradd clamav -s /sbin/nologin

创建存放日志目录
>[root@dwj etc]# mkdir -p /usr/local/clamav/share/logs

创建存放pid等临时文件或状态文件目录
>[root@dwj etc]# mkdir -p /usr/local/clamav/share/worktmp

创建存放病毒库目录
>[root@dwj etc]# mkdir -p /usr/local/clamav/share/clamav

更改病毒库目录所属权限
>[root@dwj etc]# chown clamav:clamav /usr/local/clamav/share/clamav

创建病毒库更新日志文件
>[root@dwj etc]# touch /usr/local/clamav/share/logs/freshclam.log

更改病毒库日志文件所属权限
>[root@dwj etc]# chown clamav:clamav /usr/local/clamav/share/logs/freshclam.log

修改clamav和freshclam配置文件内容
```
----------------clamav.conf----------------
LogFile /usr/local/clamav/share/logs/clamav.log
PidFile /usr/local/clamav/share/worktmp/clamav.pid
DatabaseDirectory /usr/local/clamav/share/clamav
----------------freshclam.conf-------------
UpdateLogFile /usr/local/clamav/share/logs/freshclam.log
PidFile /usr/local/clamav/share/worktmp/freshclam.pid
DatabaseDirectory /usr/local/clamav/share/clamav
DatabaseMirror database.clamav.net
```
安静模式下载更新病毒库
>[root@dwj etc]# freshclam --quiet

如果无法更新，则使用wget方式下载
```
[root@dwj etc]# cd /usr/local/clamav/share/clamav
[root@dwj clamav]# wget http://database.clamav.net/main.cvd
[root@dwj clamav]# wget http://database.clamav.net/daily.cvd
[root@dwj clamav]# wget http://database.clamav.net/bytecode.cvd
[root@dwj clamav]# chown clamav:clamav *
```
查看病毒库文件是否正确更新，如果有如下文件表示ok
>[root@dwj etc]# ls /usr/local/clamav/share/clamav  <br>
  bytecode.cvd  daily.cvd  main.cvd  mirrors.dat

下载病毒测试文件到/opt目录，其他地方拷贝也可以
>[root@dwj etc]# wget -P /opt http://www.eicar.org/download/eicar.com

扫描并删除感染文件
>[root@dwj etc]# clamscan -l /usr/local/clamav/share/logs/clamav.log --remove /opt

clamscan是扫描病毒的命令，这里简单的列举一部分常用指令参数
```
clamdscan           //一般用yum安装才能使用，需要启动clamd服务，执行速度快
clamscan            //通用，不依赖服务，不加参数即扫描当前目录文件，执行速度稍慢
clamscan -V         //查看 clamAV 的版本
clamscan -r         //递归扫描子文件夹
clamscan -i         //仅仅显示被感染的文件
clamscan -o         //跳过显示状态 ok 的文件
clamscan --remove   //检测到有病毒时，直接删除
clamscan --no-summary    //不显示统计信息
clamscan -l clamav.log   //将扫描日志写入clamav.log文件

//以上命令都可以在末尾添加文件夹，来扫描指定目录，如
clamscan --remove -rio /opt  //扫描 /opt 目录下的所有文件，只显示病毒文件，并同时删除
```

其他
```
clamd      #多线程守护程序，当clamd运行时，使用这些工具来与它进行交互：
clamdtop   #要监视的命令行 GUI clamd
clamdscan  #通过命令行程序扫描文件和目录clamd
freshclam  #签名数据库(cvd)更新工具
libclamav  #clamav库，可以将 ClamAV 引擎构建到程序中
sigtool    #一个签名数据库(cvd)操纵工具-用于恶意软件分析师和签名编写者
clambc     #另一种专门用于字节码签名的签名操作工具
clamconf   #用于检查或生成 ClamAV 配置文件并收集有助于远程调试问题所需的其他信息的工具
clamav-config #用于检查 ClamAV 如何编译的附加工具
```
