<font color=#FF0000 size=5><p align="center">概念和机制</p></font>

Golden Gate（简称OGG）提供异构环境下数据的实时捕捉、变换、传输

OGG的特性：
```
对生产系统影响小：实时读取数据日志，以低资源占用实现大量数据实时复制
高性能、使用数据库本地接口访问，并行处理体系
灵活的拓扑结构：支持一对一、一对多、多对一、多对多和双向复制等
支持数据过滤和转换，可以自定义基于表和行的过滤规则
可以对实时数据执行灵活影射和变换
提供数据压缩和加密：降低传输所需带宽，提高传输安全性
```
OGG的工作原理：

![image](https://github.com/dwjlw1314/DWJ-PROJECT/raw/master/PictureSource/3.26.1.jpg)

OGG的进程：

![image](https://github.com/dwjlw1314/DWJ-PROJECT/raw/master/PictureSource/3.26.2.jpg)

![image](https://github.com/dwjlw1314/DWJ-PROJECT/raw/master/PictureSource/3.26.3.jpg)

```
Manager进程是GoldenGate的控制进程，运行在源端和目标端上。它主要作用有以下几个方面：启动、监控、重启Goldengate的其他进程，报告错误及事件，分配数据存储空间，发布阀值报告等。在目标端和源端有且只有一个manager进程

Extract运行在数据库源端，负责从源端数据表或者日志中捕获数据。Extract的作用可以按照阶段来划分为：
  1.初始时间装载阶段：在初始数据装载阶段，Extract进程直接从源端的数据表中抽取数据
  2.同步变化捕获阶段：初始数据同步完成以后，Extract进程负责捕获源端数据的变化(DML和DDL)

Data Pump进程运行在数据库源端，其作用是将源端产生的本地trail文件，把trail以数据块的形式通过TCP/IP协议发送到目标端，这通常也是推荐的方式。pump进程本质是extract进程的一种特殊形式，如果不使用trail文件，那么extract进程在抽取完数据以后，直接投递到目标端，生成远程trail文件

Collector进程与Data Pump进程对应的叫Server Collector进程，这个进程不需要引起关注，因为在实际操作过程中，无需对其进行任何配置，所以它是透明的。它运行在目标端，其任务就是把Extract/Pump投递过来的数据重新组装成远程trail文件

Replicat进程，通常叫做应用进程。运行在目标端，是数据传递的最后一站，负责读取目标端trail文件中的内容，并将其解析为DML或 DDL语句，然后应用到目标数据库中
```
关于OGG的Trail文件：
```
为了更有效、更安全的把数据库事务信息从源端投递到目标端。GoldenGate引进trail文件的概念。前面提到extract抽取完数据以 后 Goldengate会将抽取的事务信息转化为一种GoldenGate专有格式的文件。然后pump负责把源端的trail文件投递到目标端，所以源、 目标两端都会存在这种文件。该文件存在的目的旨在防止单点故障，将事务信息持久化，并且使用checkpoint机制来记录其读写位置，如果故障发生，则数据可以根据checkpoint记录的位置来重传
```
OGG主要包含Manager进程、Extract进程、Pump进程、Replicat进程，其对应的目录含义如下：
```
dirprm：该进程所配置的参数文件，(edit param 进程组名)配置该文件
dirchk：检查点文件，记录了该进程的检查点信息
dirdat：trial文件的默认存放位置，2个用户定义的字符+6个数字组成
dirrpt：report文件，(view report 进程组名)可以查看该进程运行时的报错信息等
dirdef：存放生成的源端或目标端数据定义文件
dirtmp: 超出分配内存的事务临时存储目录
dirjar：OGGmonitor相关的jar包
dirpcs：进程状态文件
dirsql：Sql脚本
```

<font color=#FF0000 size=5><p align="center">GoldenGate的安装</p></font>

Oracle®GoldenGate安装和配置Oracle GoldenGate for Oracle官方文档参考：
> https://docs.oracle.com/goldengate/1212/gg-winux/GIORA/toc.htm

安装包准备：Oracle GoldenGate 12.2.0.2.2 for Oracle on Linux x86-64

GoldenGate实施环境
```
source database：oracle 11.2.0.1.0-64bit(RAC)
target database：oracle 12.1.0.2.0-64bit
```
GoldenGate说明：
```
1.GoldenGate的DDL的抓取不是通过日志抓取来捕获的，而是通过触发器来实现，所以对源数据库的性能影响很大
2.禁用容灾端数据库的外键，trigger和有DML操作的JOB
3.该版本已经自动建立子目录，无需再使用(create subdirs)命令创建
4.GoldenGate对符号比较敏感，ggsci里面不要有分号
5.Trigger based DDL Replication is not supported on a Multitenant database
```
创建安装包存放目录
>[root@node1 ~]# mkdir -p /data/goldengate

解压安装包
>[root@node1 goldengate]# unzip 122022_fbo_ggs_Linux_x64_shiphome.zip

创建源端和目标端安装目录(Oracle用户)
>[oracle@node1 ~]# mkdir -p /u01/app/oracle/product/ogg_src   <br>
>[oracle@node2 ~]# mkdir -p /data/oracle/product/ogg_trg

Rlwrap工具的安装和配置(rlwrap包依赖于readline和readline-devel)
```
#下载使用的是<rlwrap 0.43>版本
[root@node1 rlwrap-master]# autoreconf --install
[root@node1 rlwrap-master]# ./configure
[root@node1 rlwrap-master]# make && make install
[root@node1 rlwrap-master]# rlwrap -v
rlwrap 0.43
```
源端数据库配置环境变量，添加以下内容：
>[oracle@node1 ~]$ vim ~/.bash_profile

```
#OGG_HOME为GoldenGate的安装目录
export OGG_HOME=/u01/app/oracle/product/ogg_src
export PATH=$OGG_HOME:$PATH
export LD_LIBRARY_PATH=$OGG_HOME:$LD_LIBRARY_PATH
#配置快捷命令(路径均为GoldenGate的安装目录)
alias ggsci="rlwrap /u01/app/oracle/product/ogg_src/ggsci"
```
使环境变量生效
>[oracle@node1 ~]$ source ~/.bash_profile

目标端数据库配置环境变量，添加以下内容：
>[oracle@dwj ~]$ vim ~/.bash_profile

```
#OGG_HOME为GoldenGate的安装目录
export OGG_HOME=/data/oracle/product/ogg_trg
export PATH=$OGG_HOME:$PATH
export LD_LIBRARY_PATH=$OGG_HOME:$LD_LIBRARY_PATH
#配置快捷命令(路径均为GoldenGate的安装目录)
alias ggsci="rlwrap /data/oracle/product/ogg_trg/ggsci"
```
使环境变量生效
>[oracle@dwj ~]$ source ~/.bash_profile

运行安装程序进行图形化界面安装(以源端数据库安装为例)
>[oracle@node1 /data/goldengate/.../Disk1]# ./runInstaller

![image](https://github.com/dwjlw1314/DWJ-PROJECT/raw/master/PictureSource/3.26.4.jpg)
根据使用的数据库版本选择相应的版本

![image](https://github.com/dwjlw1314/DWJ-PROJECT/raw/master/PictureSource/3.26.5.jpg)
选择安装目录为上面创建的安装目录,其他都默认，接下来的步骤都默认直至安装完成

安装完成后提示安装日志路径
```
[oracle@node1 Disk1]$ You can find the log of this install session at:
 /u01/app/oraInventory/logs/installActions2018-11-06_11-40-23AM.log
```
查看GoldenGate是否安装成功
>[oracle@node1 Disk1]$ ggsci

```
Oracle GoldenGate Command Interpreter for Oracle
Version 12.2.0.2.2 OGGCORE_12.2.0.2.0_PLATFORMS_170630.0419_FBO
Linux, x64, 64bit (optimized), Oracle 11g on Jun 30 2017 14:42:26
Operating system character set identified as UTF-8.

Copyright (C) 1995, 2017, Oracle and/or its affiliates. All rights reserved.

GGSCI (node1) 1> view param mgr
PORT 7809

GGSCI (node1) 2> info mgr
Manager is running (IP port node1.7809, Process ID 76650)
```
GoldenGate创建独立的用户和分配必要的权限
>[oracle@node1 ogg_src]$ sqlplus / as sysdba

创建源端数据库表空间，只用于DDL模式不可用于其他模式
>SQL> create tablespace OGG logging datafile '+DATA/santdb/datafile/ogg.dbf'
size 20m autoextend on uniform size 2m;

创建源端数据库用户
>SQL> create user ogg identified by ogg default tablespace OGG temporary tablespace TEMP
quota unlimited on OGG;

给源端数据库用户授权
```
SQL> grant unlimited tablespace to ogg;
SQL> grant connect, resource, dba to ogg;
SQL> grant lock any table to ogg;
SQL> exec dbms_streams_auth.grant_admin_privilege('ogg');
```
查看目标端监听参数文件路径
>[oracle@dwj ~]$ lsnrctl status    <br>
Listener Parameter File   /opt/oracle/product/OraHome/network/admin/

编辑目标端tnsnames.ora文件,添加以下节点内容
>[oracle@dwj ~]$ vim /opt/oracle/product/OraHome/network/admin/tnsnames.ora

```
ORCLPDB =
   (DESCRIPTION =
    (ADDRESS = (PROTOCOL = TCP)(HOST = dwj)(PORT = 1521))
    (CONNECT_DATA =
      (SERVER = DEDICATED)
      (SERVICE_NAME = orclpdb.com)
    )
  )
```
查看目标端数据库PDB模式名字
>SQL> show pdbs;

数据库由CDB$ROOT模式切换至PDB模式
>SQL> alter session set container=orclpdb;

创建目标端数据库表空间
>SQL> create tablespace OGG logging datafile '/opt/oracle/oradata/orcl/orclpdb/ogg.dbf'
size 20m autoextend on uniform size 2m;

创建目标端数据库用户
>SQL> create user ogg identified by ogg default tablespace OGG temporary tablespace TEMP
quota unlimited on OGG;

给目标端数据库用户授权
```
SQL> grant unlimited tablespace to ogg;
SQL> grant connect, resource, dba to ogg;
SQL> grant lock any table to ogg;
SQL> exec dbms_streams_auth.grant_admin_privilege('ogg');
SQL> exec dbms_goldengate_auth.grant_admin_privilege('ogg');
```
创建测试用表和数据
>SQL> create table ogg.togg(id primary key, name, type, CREATED, update_date) as select object_id, object_name, object_type, CREATED, sysdate from dba_objects where rownum < 101;

开启数据库附加日志(oracle 11.2.0.4和oracle 12.1.0.2及之后的版本需要执行该语句)
>SQL> alter system set enable_goldengate_replication=true scope=both;

查看归档模式、附加日志和强制日志是否开启
>SQL> select log_mode, supplemental_log_data_min, force_logging from v$database;

```
LOG_MODE     SUPPLEME FOR
------------ -------- ---
NOARCHIVELOG NO       NO
```
修改未开启的模式
```
SQL> shutdown immediate;                         #立即关闭数据库
SQL> startup mount;                              #装载数据库而不启动
SQL> alter database archivelog;                  #启用归档模式
SQL> alter database open;                        #打开数据库
SQL> alter database force logging;               #启用强制日志
SQL> alter database add supplemental log data;   #启用附加日志
SQL> alter system switch logfile;                #切换日志文件
修改后的查询结果
LOG_MODE     SUPPLEME FOR
------------ -------- ---
ARCHIVELOG   YES      YES
```
用户级别的附加日志和表级别的附加日志只需配置其中之一(该模式为DDL同步，可以不做配置)

由于Oracle版本早于11.2.0.2，需要将10423000补丁应用于源数据库，解决用户级别的附加日志bug

源端数据库用户级别的附加日志(该文档选择方向)
>[oracle@node1 ogg_src]$ ggsci

```
GGSCI (node1) 1> dblogin userid ogg,password ogg
Successfully logged into database
GGSCI (node1 as ogg@santdb1) 2> add schematrandata ogg
```
源端数据库表级别的附加日志
>[oracle@node1 ogg_src]$ ggsci

```
GGSCI (node1) 1> dblogin userid ogg,password ogg
Successfully logged into database
GGSCI (node1 as ogg@santdb1) 2> add trandata ogg.togg
注：add trandata 用户名.表名
如果设置错误可以使用delete trandata ogg.togg删除
```
目标端数据库用户级别的附加日志(c##ogg是CDB$ROOT模式用户)
>[oracle@dwj ~]$ ggsci

```
GGSCI (dwj) 2> dblogin userid c##ogg,password ogg
Successfully logged into database CDB$ROOT.
GGSCI (dwj as c##ogg@orcl/CDB$ROOT) 3> add schematrandata cdb$root.c##ogg
This operation will modify an object at the root level of a consolidated databas
e, continue? (Y/N): y
```
目标端数据库表级别的附加日志
```
[oracle@node1 ogg_src]$ ggsci
...
GGSCI (dwj) 1> dblogin userid c##ogg,password ogg
Successfully logged into database CDB$ROOT.
GGSCI (dwj as c##ogg@orcl/CDB$ROOT) 3> add trandata cdb$root.c##ogg.togg
This operation will modify an object at the root level of a consolidated databas
e, continue? (Y/N): y
注：add trandata 容器名.用户名.表名
```
查看该数据库OPatch版本(精简)
>[oracle@dwj ~]$ $ORACLE_HOME/OPatch/opatch lsinventory

进入GoldenGate安装目录执行配置脚本(目标端数据库用户名由ogg改成c##ogg)
>[oracle@node1 ogg_src]$ sqlplus / as sysdba

```
SQL> @marker_setup
输入GoldenGate用户名ogg

SQL> @ddl_setup
输入GoldenGate用户名ogg

SQL> @role_setup
输入GoldenGate用户名ogg

SQL> grant ggs_ggsuser_role to ogg;

SQL> @ddl_enable
```
DDL同步配置完成

运行GoldenGate需要配置的进程:
```
source database：extract、data pump
target database：replicat
```

配置Manager进程(管理进程)
```
进程必须在源端和目标端运行，并且在Extract和Replicat进程之前启动，没有该进程OGG无法做其它的操作，它管理启动GoldenGate进程、启动动态进程、分配端口给GoldenGate进程、管理trail file、创建事件，错误和诊断报告工作
```
确认Manager进程已经运行
>[oracle@node1 OPatch]$ ggsci

```
GGSCI (node1) 1> info mgr
Manager is running (IP port node1.7809, Process ID 76650)
```
源端管理进程参数配置，添加内容如下
>GGSCI (node1) 1> edit param mgr

```
#通信端口7809，源端和目标端需要保持一致
PORT 7809

#动态端口的范围。当指定端口被占用或者出现通信故障，管理进程将会选择下一个端口尝试连接，避免单点故障
DYNAMICPORTLIST  7840-7939

#当mgr进程启动后启动extract进程
AUTOSTART EXTRACT *

#当extract进程中断后尝试自动重启，每隔2分钟尝试启动一次，尝试5次
AUTORESTART EXTRACT *, RETRIES 5, WAITMINUTES 2

#定期清理dirdat路径下的本地队列(local trail)，保留期限10天，过期后自动删除。控制队列文件目录无序增长
PURGEOLDEXTRACTS /u01/app/oracle/product/ogg_src/dirdat/*, USECHECKPOINTS, MINKEEPDAYS 10

#每隔一小时检查各进程延时情况，并记录到goldengate report文件
LAGREPORTHOURS 1

#进程复制延时超过30分钟，向日志文件记录一条错误日志
LAGINFOMINUTES 30

#传输延时超过45分钟将写入警告日志
LAGCRITICALMINUTES 45
```
配置Extract进程(只在源端配置)
```
Extract 运行在源端，抽取捕获系统变更的数据；它可以配置为初始化数据加载(直接从数据源中加载静态的数据)和在某个时间点后源端与服务端变更数据同步(从在线日志或归档日志抽取捕获变更的数据)，它也可以在支持DDL变更的系统中抽取捕获DDL

当配置为数据同步时，extract进程抽取捕获extract配置文件里配置的对象的任何DML和DDL(需要额外配置)的操作，extract进程记录这些操作，直到用户提交或回滚事务；当收到回滚(rollback)时，extract撤销这些记录；当收到(commit)操作后，extract进程记录保存这些操作到一个或多个trail文件里并以队列的形式发送到目标端，以确保数据传输速度和数据的一致性
```
配置主抽取进程(Primary Extract)，使用ogg用户登录GoldenGate
>GGSCI (node1) 2> dblogin userid ogg,password ogg

创建主抽取进程,RAC使用<threads 2>标识
>GGSCI (node1 as ogg@santdb1) 3> add extract ext1,tranlog,threads 2,begin now

```
OGG-00868 ext1.prm: The number of Oracle redo threads (2) is not the same as the number of
checkpoint threads (1),EXTRACT groups on RAC systems should be created with the THREADS parameter
(e.g., ADD EXT <group name>, TRANLOG, THREADS 2, BEGIN...)
```
创建源端trail文件并指定路径
>GGSCI (node1 as ogg@santdb1) 4> add exttrail /u01/app/oracle/product/ogg_src/dirdat/sr,extract ext1

主进程的作用是抽取捕获系统变更数据并将这些数据保存到trail文件里，所以要配置trail文件目录和trail文件名

编辑配置文件ext1，添加以下内容
>GGSCI (node1 as ogg@santdb1) 5> edit param ext1

```
extract ext1

#设置Oracle数据库实例sid
SETENV(ORACLE_SID="santdb1")

#设置goldengate的字符集变量信息，此处值会覆盖操作系统级别的变量。该值需要和数据库字符集匹配一致
SETENV(NLS_LANG=AMERICAN_AMERICA.AL32UTF8)

userid ogg, password ogg

#每隔30分钟报告一次从程序开始到现在的抽取进程或者复制进程的事物记录数，并汇报进程的统计信息
REPORTCOUNT EVERY 30 MINUTES, RATE

#将执行失败的记录保存在discard file中，discard file文件记录了GoldenGate进程错误、数据库错误、GoldenGate操作等信息
#该文件位于.../dirrpt/extsr.dsc,大小为1024MB。文件中已经包含记录的话，再后面继续追加，不删除之前的记录
DISCARDFILE /u01/app/oracle/product/ogg_src/dirrpt/extsr.dsc, APPEND, MEGABYTES 1024

#为了防止discard file被写满，每天3：00做一次文件过期设定
DISCARDROLLOVER AT 3:00

#队列文件路径, trail文件存放路径
EXTTRAIL /u01/app/oracle/product/ogg_src/dirdat/sr

#有时候开启OGG进程的时候较慢，可能是因为需要同步的表太多，OGG在开启进程之前会将需要同步的表建立一个记录并且存入到磁盘中
#这样就需要耗费大量的时间。使用该参数来解决此问题
DYNAMICRESOLUTION

#用于阻止抽取进程抽取数据时由于表含有unused列而导致进程异常终止(abend)
#使用该参数，抽取进程抽取到unused列时也会向日志文件记录一条警告信息
DBOPTIONS  ALLOWUNUSEDCOLUMN

#默认值为 usesnapshot，表示利用数据库闪回读取数据。Nousesnapshot表示直接从原表读取相关数据
FETCHOPTIONS NOUSESNAPSHOT

#当使用了HANDLECOLLISIONS时，请使用该参数。复制进程出现丢失update记录（missing update）并且更新的是主键，
#update将转换成insert。由于插入的记录可能不是完整的行，若要保证完整需要加入此参数
FETCHOPTIONS FETCHPKUPDATECOLS

#需要复制的对象列表
table ogg.*;
```
配置投递进程(Data Pump)
```
Data Pumps是第二种类型的GoldenGate extract配置，如果不使用Data Pump，extract进程必须发送抽取捕获的操作数据到目标端trail；如果配置了Data Pump，extract进程抽取捕获数据写入到trail，Data pump读取trail并且通过网络发送trail到目标端trail，data pump 加强了源端和目标端抽取捕获数据的可用性,创建并指定源数据库trail文件位置，必须包含两个字符，这个路径和主抽取进程(Primary Extract)中指定的trail目录和trail文件命名必须相同，因为Data Pump进程要从此读取主抽取进程生成的trail文件

主要优点:
1.保护网络传输失败和目标端失败
2.可以实现复杂的数据过滤和转换
3.可以结合多个数据源到目标端
4.可以同步一个源数据到多个目标端
```
>GGSCI (node1 as ogg@santdb1) 7> add extract dpump1, exttrailsource /u01/app/oracle/product/ogg_src/dirdat/sr

编辑Data Pump进程参数文件
>GGSCI (node1 as ogg@santdb1) 8> edit param dpump1

```
extract dpump1
SETENV(ORACLE_SID="santdb1")
SETENV(NLS_LANG=AMERICAN_AMERICA.AL32UTF8)
#目标端主机IP，管理进程端口号，投递前压缩队列文件
RMTHOST 192.168.0.109, mgrport 7809, COMPRESS

#表示传输进程直接跟抽取进程交互，而不再和数据库进行交互，减少数据库资源的利用
PASSTHRU

#目标端保存队列文件的目录
RMTTRAIL /data/oracle/product/ogg_trg/dirdat/tr

#动态解析表名
DYNAMICRESOLUTION

#复制范围和抽取进程对应即可
table ogg.*;
```
指定Data Pump进程发送trail文件到目标端的位置(目标端trail文件添加到队列中)
>GGSCI (node1 as ogg@santdb1) 9> add rmttrail /data/oracle/product/ogg_trg/dirdat/tr, extract dpump1

目标端管理进程参数配置，添加内容如下
>GGSCI (dwj) 1> edit param mgr

```
#通信端口7809，源端和目标端需要保持一致
PORT 7809

#GoldenGate用户登录数据库的用户名和密码，密码未做加密处理
#如果密码需要加密使用：GGSCI (dbtrg) 1> encrypt password pwd ,ENCRYPTKEY default
#可以得到加密后的密码字符串，之后配置进程若使用加密过的密码，需要带参数(ENCRYPTKEY default)
#例如：USERID ogg, PASSWORD xxx(加密过的密码) ，ENCRYPTKEY default
USERID ogg@orclpdb, PASSWORD ogg

DYNAMICPORTLIST  7840-7939
AUTOSTART REPLICAT *
AUTORESTART REPLICAT *, RETRIES 5, WAITMINUTES 2
PURGEOLDEXTRACTS /data/oracle/product/ogg_trg/dirdat/*, USECHECKPOINTS, MINKEEPDAYS 10

#删除DDL历史表，最小保存7天，最大保存10天
PURGEDDLHISTORY MINKEEPDAYS 7, MAXKEEPDAYS 10

#删除MARKER历史表，最小保存7天，最大保存10天
PURGEMARKERHISTORY MINKEEPDAYS 7, MAXKEEPDAYS 10

LAGREPORTHOURS 1
LAGINFOMINUTES 30
LAGCRITICALMINUTES 45
```

目标端配置Replicat进程
```
Replicat进程运行在目标端读取tail文件和重构DML、DDL并应用到目标数据库；Replicat编译SQL一次，当变量值不同时重复使用编译过的SQL；Replicat进程可以像extract进程一样配置初始化数据加载(直接从数据源中加载静态的数据)和在某个时间点后源端与服务端变更数据同步(从在线日志或归档日志抽取捕获变更的数据)

Checkpoint存储从文件读取和写入的检测点位置，用于还原和恢复数据，Checkpoint确保发生变化并提交(commit)的数据被extract抽取捕获和被replicat进程应用到目标端；保证在系统、网络或者GoldenGate需要重启进程时发生的错误不会导致数据丢失；在复杂的同步配置中checkpoints启用多个extract和replicat进程从同一个trail集中读取数据
```
使用GoldenGate用户(c##ogg)登录
>GGSCI (dwj) 1> dblogin userid c##ogg,password ogg

创建和配置Checkpoint Table
>GGSCI (dwj as c##ogg@orcl/CDB$ROOT) 2> add checkpointtable cdb$root.c##ogg.checkpoint

编辑GoldenGate配置文件,并且GLOBALS文件以大写命名并且没有扩展名创建在OGG_home根目录
>GGSCI (dwj as c##ogg@orcl/CDB$ROOT) 3> edit param ./GLOBALS

```
GGSCHEMA c##ogg
CHECKPOINTTABLE cdb$root.c##ogg.checkpoint
```
创建replicat进程
>GGSCI (dwj as c##ogg@orcl/CDB$ROOT) 4> add replicat rep1, exttrail /data/oracle/product/ogg_trg/dirdat/tr, checkpointtable cdb$root.c##ogg.checkpoint

编辑replicat进程参数文件
>GGSCI (dwj as c##ogg@orcl/CDB$ROOT) 5> edit param rep1

```
REPLICAT rep1
SETENV(NLS_LANG=AMERICAN_AMERICA.AL32UTF8)
USERID ogg@orclpdb, PASSWORD ogg

#每天06:00定期生成一个report文件
REPORT AT 06:00

#每隔30分钟报告一次从程序开始到现在的抽取进程或者复制进程的事物记录数，并汇报进程的统计信息
REPORTCOUNT EVERY 30 MINUTES, RATE

#为了防止report file被写满，每天2：00做一次文件过期设定
REPORTROLLOVER AT 02:00

#除了特殊指定的REPERROR语句，报告所有复制期间出现的错误，回滚非正常中断的事物和进程
#遇到不能处理的错误就自动abend，启动需要人工干预处理
REPERROR DEFAULT, ABEND

#当源表有排除列情况或者有目标表不存在的列时，当更新这列goldengate默认报错
#应用该参数后，即可让goldengate生成一条警告信息而不是报错
ALLOWNOOPUPDATES

#使用ASSUMETARGETDEFS参数时，用MAP语句中指定的生产库源表和灾备端目标表具有相同的列结构
#它指示的Oracle GoldenGate不在生产端查找源表的结构定义
ASSUMETARGETDEFS

#用于goldengate自动过滤冲突记录，严格保证数据一致性
HANDLECOLLISIONS

#将执行失败的记录保存在discard file中，discard file文件记录了GoldenGate进程错误、数据库错误、GoldenGate操作等信息
#该文件位于./dirrpt/repsa.dsc,大小为1024MB。 文件中已经包含记录的话，再后面继续追加，不删除之前的记录
DISCARDFILE /data/oracle/product/ogg_trg/dirrpt/repsa.dsc, APPEND, MEGABYTES 1024

#为了防止discard file被写满，每天2：00做一次文件过期设定
DISCARDROLLOVER AT 02:00

#对应需要复制的对象，默认一一对应传输进程
MAP ogg.*, target orclpdb.ogg.*;
```

初始化数据
```
DML操作包括INSERT、UPDATE、DELETE、SELECT操作，而在这些操作中UPDATE、DELETE操作Redo只记录了变更的数据列以及行ID(ROWID)，GoldenGate抽取数据后将其转换为自己的格式发送都目标端
在同步开始前目标端没有初始化数据(目标端为空数据)，那么事物产生的UPDATE、DELETE DML操作发送到目标端，目标端GoldenGate Replicat进程会因为找不到数据而报错从而导致Replicat进程崩溃停止(ABENDED)
所以这就需要在同步前初始化数据，初始化完后再同步，这样大大降低错误率
同步数据的方式可以通过DBLINK、EXP/IMP、SQLLDR或者表空间迁移等方式同步
```

数据初始化后，分别启动目标端MGR进程、Replicat进程，源端MGR进程、主抽取进程、Data Pump进程(Secondly Extract)

启动GoldenGate(目标端)
```
>GGSCI (dwj) 1> stop mgr
>GGSCI (dwj) 2> start mgr
>GGSCI (dwj) 3> info all

Program     Status      Group       Lag at Chkpt  Time Since Chkpt

MANAGER     RUNNING
REPLICAT    RUNNING     REP1        00:00:00      00:00:06
GGSCI (dwj) 4> stats rep1

Sending STATS request to REPLICAT REP1 ...
No active replication maps.
```
启动GoldenGate(源端)
```
>GGSCI (node1) 1> stop mgr
>GGSCI (node1) 2> start mgr
>GGSCI (node1) 3> info all

Program     Status      Group       Lag at Chkpt  Time Since Chkpt

MANAGER     RUNNING
EXTRACT     RUNNING     DPUMP1      00:00:00      00:00:06
EXTRACT     RUNNING     EXT1        00:00:00      17:12:13
```

因为在mgr都相应的配置了extract进程和replicat进程的自启动，所以在mgr进程启动后会自动启动extract进程和replicat进程

如果启动失败，可以查看GoldenGate安装目录下的错误日志(ggserr.log)
>[oracle@node1 ogg_src]$ tail -f ggserr.log

查看进程启动日志，格式如下：
>GGSCI (node1) 1> view report EXT1
