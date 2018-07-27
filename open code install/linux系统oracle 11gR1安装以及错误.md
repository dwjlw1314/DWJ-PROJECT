前提: Oracle 11g ;  Redhat x86-64;

前提将安装包linux.x64_11gR1_database.zip上传到/opt目录下，执行解压命令  <br>
解压后生成database文件夹，其他格式安装包另行解压，如tar.gz格式包

    [root@gjsy opt]# unzip linux.x64_11gR1_database.zip

添加host主机信息,先备份/etc/hosts文件，添加主机信息

    [root@gjsy /]# vim /etc/hosts
    172.16.10.200 gjsy

关闭防火墙

    [root@gjsy /]# service iptables stop

创建oralce相关用户和组

    [root@gjsy /]# groupadd oinstall
    [root@gjsy /]# groupadd dba
    [root@gjsy /]# groupadd oper
    [root@gjsy /]# useradd oracle

设置oracle用户所属组

    [root@gjsy /]# usermod -g oinstall -G dba oracle （dba为管理组）

设置oracle用户密码

    [root@gjsy /]# passwd oracle

创建oracle相关目录

    [root@gjsy /]# mkdir -p /opt/oracle/product
    [root@gjsy /]# mkdir -p /opt/oracle/product/OraHome
    [root@gjsy /]# mkdir -p /opt/oralnventory
    [root@gjsy /]# mkdir -p /opt/oracle/oradata
    [root@gjsy /]# mkdir -p /var/opt/oracle

设置目录所有者所属组和权限

    [root@gjsy /]# chown -R oracle.oinstall /opt/oracle
    [root@gjsy /]# chown -R oracle.oinstall /opt/oracle/oradata
    [root@gjsy /]# chown -R oracle.oinstall /opt/oracle/product/OraHome
    [root@gjsy /]# chown -R oracle.dba /opt/oraInventory
    [root@gjsy /]# chown -R oracle.dba /var/opt/oracle
    [root@gjsy /]# chmod -R 755 /opt/oracle
    [root@gjsy /]# chmod -R 755 /opt/oraInventory
    [root@gjsy /]# chmod -R 755 /var/opt/oracle

把oracle安装程序拷到用户目录下面

    [root@gjsy /]# cp –r /opt/database /home/oracle/

把目录设置成具有读写和执行的权限，用户和组改为oracle

    [root@gjsy /]# chmod –R 755 /home/oracle/database
    [root@gjsy /]# chown -R oracle /home/oracle/database
    [root@gjsy /]# chgrp -R oinstall /home/oracle/database

<font color=#FF0000 size=5><p align="center">可选配置<配置内核参数和资源限制>(根据实际设置)</p></font>

    [root@gjsy /]# vim /etc/sysctl.conf  (在最后添加下面的参数)
    fs.aio-max-nr = 1048576
    fs.file-max = 6815744
    kernel.shmall = 2097152
    kernel.shmmax = 980842496
    kernel.shmmni = 4096
    kernel.sem = 250 32000 100 128
    net.ipv4.ip_local_port_range = 9000 65500
    net.core.rmem_default = 262144
    net.core.rmem_max = 4194304
    net.core.wmem_default = 262144
    net.core.wmem_max = 1048576
    [root@gjsy /]# sysctl –p   #使配置的内核参数生效

    [root@gjsy /]# vim /etc/security/limits.conf   (追加下面数据配置)
    oracle soft nofile  1024
    oracle hard nofile  65536
    oracle soft nproc  2047
    oracle hard nproc 16384                                                                   
    oracle soft stack  10240
    oracle hard stack  32768

设置用户oracle的环境变量(必填项)

    [root@gjsy /]# vim /home/oracle/.bash_profile   (添加如下内容)
    export ORACLE_BASE=/opt/oracle
    export ORACLE_HOME=$ORACLE_BASE/product/OraHome
    export ORACLE_SID=orcl
    export ORACLE_OWNER=oracle
    export ORACLE_TERM=vt100
    export PATH=$PATH:$ORACLE_HOME/bin:$HOME/bin
    export PATH=$ORACLE_HOME/bin:$ORACLE_HOME/Apache/bin:$PATH
    LD_LIBRARY_PATH=$ORACLE_HOME/lib:/lib:/usr/lib:/usr/local/lib
    export LD_LIBRARY_PATH
    CLASSPATH=$ORACLE_HOME/JRE:$ORACLE_HOME/jlib:$ORACLE_HOME/rdbms/jlib
    CLASSPATH=$CLASSPATH:$ORACLE_HOME/network/jlib
    export CLASSPATH
    PATH=$PATH:/usr/sbin; export PATH
    PATH=$PATH:/usr/bin; export PATH
    ORA_NLS33=$ORACLE_HOME/nls/admin/data
    [root@gjsy /]# source /home/oracle/.bash_profile   #使配置的环境参数生效

注销root用户，用oracle用户登录  <br>
进入/home/oracle/database目录运行图形界面开始安装

    [oracle@diwj ~]$ cd database
    [oracle@diwj database]$ ./runInstaller

![image](https://github.com/dwjlw1314/DWJ-PROJECT/raw/master/PictureSource/oraclegR1/3.14.1.jpg)
选择NEXT

![image](https://github.com/dwjlw1314/DWJ-PROJECT/raw/master/PictureSource/oraclegR1/3.14.2.jpg)
选择skip software updates

![image](https://github.com/dwjlw1314/DWJ-PROJECT/raw/master/PictureSource/oraclegR1/3.14.3.jpg)
选择install database software only

![image](https://github.com/dwjlw1314/DWJ-PROJECT/raw/master/PictureSource/oraclegR1/3.14.4.jpg)
选择Singe instance database installation

![image](https://github.com/dwjlw1314/DWJ-PROJECT/raw/master/PictureSource/oraclegR1/3.14.5.jpg)
选择English，Traditional Chinese，Simplified Chinese，Spanish

![image](https://github.com/dwjlw1314/DWJ-PROJECT/raw/master/PictureSource/oraclegR1/3.14.6.jpg)
选择企业版Enterprise Edition

![image](https://github.com/dwjlw1314/DWJ-PROJECT/raw/master/PictureSource/oraclegR1/3.14.7.jpg)
指定安装目录 Oracle Base：/opt/oracle   <br>
Software Location:/opt/oracle/product/OraHome

![image](https://github.com/dwjlw1314/DWJ-PROJECT/raw/master/PictureSource/oraclegR1/3.14.8.jpg)
选择Inventory Directory:/opt/oraInventory

![image](https://github.com/dwjlw1314/DWJ-PROJECT/raw/master/PictureSource/oraclegR1/3.14.9.jpg)
选择相应的Group：oper

![image](https://github.com/dwjlw1314/DWJ-PROJECT/raw/master/PictureSource/oraclegR1/3.14.10.jpg)
选择Fix&Check Again   <br>
如果有错误，可以选择忽略   <br>
打开终端，以root登陆   <br>
#cd /tmp/CVU_11.2.0..3.0_oracle   <br>
#./runfixup.sh

![image](https://github.com/dwjlw1314/DWJ-PROJECT/raw/master/PictureSource/oraclegR1/3.14.11.jpg)
若存在这些警告，在linux系统安装这些包,然后重新选择Check Again或者勾选【Ignore All】

![image](https://github.com/dwjlw1314/DWJ-PROJECT/raw/master/PictureSource/oraclegR1/3.14.12.jpg)
选择Install

![image](https://github.com/dwjlw1314/DWJ-PROJECT/raw/master/PictureSource/oraclegR1/3.14.13.jpg)
出现提示进入相应目录，用root执行2个脚本   <br>
#./orainstRoot.sh   <br>
#./root.sh

![image](https://github.com/dwjlw1314/DWJ-PROJECT/raw/master/PictureSource/oraclegR1/3.14.14.jpg)
数据库安装成功

安装监听器，oracle用户进入图形界面

    [oracle@diwj database]$ netca

![image](https://github.com/dwjlw1314/DWJ-PROJECT/raw/master/PictureSource/oraclegR1/3.14.15.jpg)
选择Listener configuration

![image](https://github.com/dwjlw1314/DWJ-PROJECT/raw/master/PictureSource/oraclegR1/3.14.16.jpg)
勾选Add

![image](https://github.com/dwjlw1314/DWJ-PROJECT/raw/master/PictureSource/oraclegR1/3.14.17.jpg)
根据需求修改监听名称LISTENER

![image](https://github.com/dwjlw1314/DWJ-PROJECT/raw/master/PictureSource/oraclegR1/3.14.18.jpg)
选择协议TCPS和TCP

![image](https://github.com/dwjlw1314/DWJ-PROJECT/raw/master/PictureSource/oraclegR1/3.14.19.jpg)
选择Use the standard port number of 1521端口号

![image](https://github.com/dwjlw1314/DWJ-PROJECT/raw/master/PictureSource/oraclegR1/3.14.20.jpg)
选择默认 no

![image](https://github.com/dwjlw1314/DWJ-PROJECT/raw/master/PictureSource/oraclegR1/3.14.21.jpg)
单击Finish 安装结束

安装完成后使用ps -ef|grep LISTENER  查案listener进程

    [oracle@diwj database]$ ps -ef | grep LISTENER
    oracle    24563     1   0  20:30  ?     00:00:00  /opt/oracle/product/OraHome/bin/
    tnslsnr LISTENER -inherit
    oracle    24605  10528  0  20:31 pts/0  00:00:00  grep LISTENER

新建数据库，oracle用户进入图形界面

    [oracle@diwj database]$ dbca

![image](https://github.com/dwjlw1314/DWJ-PROJECT/raw/master/PictureSource/oraclegR1/3.14.22.jpg)
选择Next

![image](https://github.com/dwjlw1314/DWJ-PROJECT/raw/master/PictureSource/oraclegR1/3.14.23.jpg)
勾选Create a Database

![image](https://github.com/dwjlw1314/DWJ-PROJECT/raw/master/PictureSource/oraclegR1/3.14.24.jpg)
选择next

![image](https://github.com/dwjlw1314/DWJ-PROJECT/raw/master/PictureSource/oraclegR1/3.14.25.jpg)
根据提示输入全集数据库名称，例如orcl.com

![image](https://github.com/dwjlw1314/DWJ-PROJECT/raw/master/PictureSource/oraclegR1/3.14.26.jpg)
选择Configure Enterprise Manager和Configure Database Control for local management

![image](https://github.com/dwjlw1314/DWJ-PROJECT/raw/master/PictureSource/oraclegR1/3.14.27.jpg)
选择第二个选项Use the Same Administrative for all Accounts，根据提示输入密码并确认

![image](https://github.com/dwjlw1314/DWJ-PROJECT/raw/master/PictureSource/oraclegR1/3.14.28.jpg)
选择Use Database File Location from Template

![image](https://github.com/dwjlw1314/DWJ-PROJECT/raw/master/PictureSource/oraclegR1/3.14.29.jpg)
保留Fast Recovery Area以及Fast Recovery Area Size的默认数值

![image](https://github.com/dwjlw1314/DWJ-PROJECT/raw/master/PictureSource/oraclegR1/3.14.30.jpg)
选择Sample Schemas

![image](https://github.com/dwjlw1314/DWJ-PROJECT/raw/master/PictureSource/oraclegR1/3.14.31.jpg)
选择Use Unicode(AL32UTF8), 在“Memory”页签中修改“Memory Size”大小为内存的50％

![image](https://github.com/dwjlw1314/DWJ-PROJECT/raw/master/PictureSource/oraclegR1/3.14.32.jpg)
选择next

![image](https://github.com/dwjlw1314/DWJ-PROJECT/raw/master/PictureSource/oraclegR1/3.14.33.jpg)
勾选Create Database,选择Finish

![image](https://github.com/dwjlw1314/DWJ-PROJECT/raw/master/PictureSource/oraclegR1/3.14.34.jpg)
确认所创建的数据库，选择ok

![image](https://github.com/dwjlw1314/DWJ-PROJECT/raw/master/PictureSource/oraclegR1/3.14.35.jpg)
数据库创建完成，选择Exit

==== SERVICE_NAME = idc   //可以通过下面语句查看

    SQL> sqlplus / as sysdba;
    SQL> show parameter service_name;
    SQL> alter system set service_names=new;  //修改sevice_name

创建数据表空间分为四步

    SQL> show parameter create
    SQL> alter system set db_create_file_dest='/opt/oracle/oradata/';

第1步：创建临时表空间

    SQL> create temporary tablespace gjsy_temp
    tempfile '/opt/oracle/oradata/orcl/gjsy_temp.dbf'
    size 50m
    autoextend on
    next 50m maxsize 20480m
    extent management local;

第2步：创建数据表空间

    SQL> create tablespace gjsy_data
    logging
    datafile '/opt/oracle/oradata/orcl/gjsy_data.dbf '
    size 50m
    autoextend on
    next 50m maxsize 20480m
    extent management local;

第3步：创建用户并指定表空间

    SQL> create user ant2 identified by password
    default tablespace gjsy_data
    temporary tablespace gjsy_temp;

第4步：给用户授予权限

    SQL> grant connect,resource,dba to ant2;

<font color=#FF0000 size=5> <p align="center">常用命令</p></font>

    Sqlplus> alter user ant identified by “oracle”;       #修改密码
    Sqlplus> alter user antvideo identified by “oracle”;
    Sqlplus> shutdown immediate                           #关闭服务器
    Sqlplus> startup                                      #启动服务器
    Sqlplus> drop user ant cascade;                       #删除用户
    Sqlplus> drop user antvideo cascade;
    Sqlplus> show parameter db;                           #查看数据库信息

<font color=#FF0000 size=5> <p align="center">Oracle dbstart和dbshut</p></font>  <br>
一、用dbstart和dbshut启动和关闭数据库实例

先启动监听 lsnrctl start

启动实例  dbstart

使用dbstart命令启动数据库比较方便，但是在linux上安装好oracle之后，第一次使用dbstart命令可能会报如下错误

ORACLE_HOME_LISTNER is not SET, unable to auto-start Oracle Net Listener

Usage: /u01/app/oracle/oracle/product/10.2.0/db_1/bin/dbstart ORACLE_HOME

原因：dbstart和dbshut脚本文件中ORACLE_HOME_LISTNER的设置有问题，分别打开两个文件找到，

用vim编辑dbstart，ORACLE_HOME_LISTNER=$1，修改为ORACLE_HOME_LISTNER=$ORACLE_HOME

然后保存退出，此时再运行dbstart，已经不报错了，但是没有任何反应，ps一下进程，没有oracle的进程，说明oracle实例没有正常启动

此时的原因是在/etc/oratab的设置问题，我们vim一下，发现 dwj:/home/oracle/product/10g:N

最后设置的是"N"（我的环境中只有一个实例，因此只有一行配置语句），我们需要把“N”修改为“Y”

以上的工作做好之后，dbstart就可以正常使用了

[oracle@redhat bin]$ lsnrctl start                                   --“启动监听”

[oracle@redhat bin]$ dbstart                                         --“启动数据库实例”

Processing Database instance "dwj": log file /home/oracle/product/10g/startup.log

[oracle@redhat bin]$ dbshut                                    --“关闭数据库实例”

[oracle@redhat bin]$ lsnrctl stop                              --“关闭监听”

二、使数据库实例和linux系统一起启动   <br>
在/etc/rc.d/rc.local中加入如下语句即可实现同系统启动实例：

    su - oracle -c "lsnrctl start"
    su - oracle -c "dbstart"

<font color=#FF0000 size=5> <p align="center">00020-error</p></font>  <br>
```
sqlplus> select count(*) from v$process;              //取得数据库目前的进程数。
sqlplus> select value from v$parameter where name = 'processes';      //取得进程数的上限。
sqlplus> alter system set processes=500 scope=spfile sid=‘orcl’      //设置进程数上限。
修改后重新启动数据库就好了。
show parameter processes;查看
```
<font color=#FF0000 size=5> <p align="center">Oracle RMAN 备份及恢复步骤</p></font>  <br>

1、切换服务器归档模式，如果已经是归档模式可跳过此步：
```
[oracle@diwj ~]sqlplus /nolog      (启动sqlplus)
SQL> conn / as sysdba              (以DBA身份连接数据库)
SQL> shutdown immediate;           (立即关闭数据库)
SQL> startup mount                 (启动实例并加载数据库，但不打开)
SQL> alter database archivelog;    (更改数据库为归档模式)
SQL> alter database open;          (打开数据库)
SQL> alter system archive log start;(启用自动归档)
SQL> exit                          (退出)
```
2、连接：

rman target=sys/comeon@orcl;       (启动恢复管理器)

3、基本设搜索置：
```
RMAN> configure default device type to disk;        (设置默认的备份设备为磁盘)
RMAN> configure device type disk parallelism 2;     (设置备份的并行级别，通道数)
RMAN> configure channel 1 device type disk fromat '/backup1/backup_%U';     (设置备份的文件格式，只适用于磁盘设备)
RMAN> configure channel 2 device type disk fromat '/backup2/backup_%U';     (设置备份的文件格式，只适用于磁盘设备)
RMAN> configure controlfile autobackup on;     (打开控制文件与服务器参数文件的自动备份)
RMAN> configure controlfile autobackup format for device type disk to '/backup1/ctl_%F';  (设置控制文件与服务器参数文件自动备份的文件格式)
```
4、查看所有设置：

RMAN> show all

5、查看数据库方案报表：

RMAN> report schema;

6、备份全库：

RMAN> backup database plus archivelog delete input;   (备份全库及控制文件、服务器参数文件与所有归档的重做日志，并删除旧的归档日志)

7、备份表空间：

RMAN> backup tablespace system plus archivelog delete input;  (备份指定表空间及归档的重做日志，并删除旧的归档日志)

8、备份归档日志：

RMAN> backup archivelog all delete input;

9、复制数据文件：

RMAN> copy datafile 1 to '/oracle/dbs/system.copy';

10、查看备份和文件复本：

RMAN> list backup;

11、验证备份：

RMAN> validate backupset 3;

12、从自动备份中恢复服务器参数文件：

RMAN> shutdown immediate;     (立即关闭数据库)

RMAN> startup nomount;        (启动实例)

RMAN> restore spfile to pfile '/backup1/mydb.ora' from autobackup;     (从自动备份中恢复服务器参数文件)

13、从自动备份中恢复控制文件：

RMAN> shutdown immediate;     (立即关闭数据库)

RMAN> startup nomount;        (启动实例)

RMAN> restore controlfile to '/backup1' from autobackup;     (从自动备份中恢复控制文件)

13、恢复和复原全数据库：

RMAN> shutdown immediate;     (立即关闭数据库)

RMAN> exit     (退出)

%mv /oracle/dbs/tbs_12.f /oracle/dbs/tbs_12.bak     (将数据文件重命名)

%mv /oracle/dbs/tbs_13.f /oracle/dbs/tbs_13.bak     (将数据文件重命名)

%mv /oracle/dbs/tbs_14.f /oracle/dbs/tbs_14.bak     (将数据文件重命名)

%mv /oracle/dbs/tbs_15.f /oracle/dbs/tbs_15.bak     (将数据文件重命名)

%rman target=rman/rman@mydb                         (启动恢复管理器)

RMAN> startup pfile=/oracle/admin/mydb/pfile/initmydb.ora     (指定初始化参数文件启动数据库)

RMAN> restore database;     (还原数据库)

RMAN> recover database;     (恢复数据库)

RMAN> alter database open;     (打开数据库)

14、恢复和复原表空间：

RMAN> sql 'alter tablespace users offline immediate';     (将表空间脱机)

RMAN> exit     (退出恢复管理器)

%mv /oracle/dbs/users01.dbf /oracle/dbs/users01.bak       (将表空间重命名)

%rman target=rman/rman@mydb         (启动恢复管理器)

RMAN> restore tablespace users;     (还原表空间)

RMAN> recover tablespace users;     (恢复表空间)

RMAN> sql 'alter tablespace users online';     (将表空间联机)

15、增量备份与恢复：

第一天的增量基本备份：

RMAN> backup incremental level=0 database plus archivelog delete input;

第二天的增量差异备份：

RMAN> backup incremental level=2 database plus archivelog delete input;

第三天的增量差异备份：

RMAN> backup incremental level=2 database plus archivelog delete input;

第四天的增量差异备份：

RMAN> backup incremental level=1 database plus archivelog delete input;

第五天的增量差异备份：

RMAN> backup incremental level=2 database plus archivelog delete input;

第六天的增量差异备份：

RMAN> backup incremental level=2 database plus archivelog delete input;

第七天的增量差异备份：

RMAN> backup incremental level=0 database plus archivelog delete input;


增量恢复：

RMAN> shutdown immediate;

RMAN> exit

%mv /oracle/dbs/tbs_12.f /oracle/dbs/tbs_12.bak

%mv /oracle/dbs/tbs_13.f /oracle/dbs/tbs_13.bak

%mv /oracle/dbs/tbs_14.f /oracle/dbs/tbs_14.bak

%mv /oracle/dbs/tbs_15.f /oracle/dbs/tbs_15.bak

%rman target=rman/rman@mydb

RMAN> startup pfile=/oracle/admin/mydb/pfile/initmydb.ora

RMAN> restore database;

RMAN> recover database;

RMAN> alter database open;
