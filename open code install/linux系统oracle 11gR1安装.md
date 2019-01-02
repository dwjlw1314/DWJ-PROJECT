前提: Oracle 11gR1;  Redhat x86-64;

前提将安装包linux.x64_11gR1_database.zip上传到/opt目录下，执行解压命令

解压后生成database文件夹，其他格式安装包另行解压，如tar.gz格式包
>[root@gjsy opt]# unzip linux.x64_11gR1_database.zip

添加host主机信息,先备份/etc/hosts文件，添加主机信息
```
[root@gjsy /]# vim /etc/hosts
172.16.10.200 gjsy
```
关闭防火墙
>[root@gjsy /]# service iptables stop

创建oralce相关用户和组
```
[root@gjsy /]# groupadd oinstall
[root@gjsy /]# groupadd dba
[root@gjsy /]# groupadd oper
[root@gjsy /]# useradd oracle
```
设置oracle用户所属组
>[root@gjsy /]# usermod -g oinstall -G dba oracle （dba为管理组）

设置oracle用户密码
>[root@gjsy /]# passwd oracle

创建oracle相关目录
```
[root@gjsy /]# mkdir -p /opt/oracle/product
[root@gjsy /]# mkdir -p /opt/oracle/product/OraHome
[root@gjsy /]# mkdir -p /opt/oraInventory
[root@gjsy /]# mkdir -p /opt/oracle/oradata
[root@gjsy /]# mkdir -p /var/opt/oracle
```
设置目录所有者所属组和权限
```
[root@gjsy /]# chown -R oracle.oinstall /opt/oracle
[root@gjsy /]# chown -R oracle.oinstall /opt/oracle/oradata
[root@gjsy /]# chown -R oracle.oinstall /opt/oracle/product/OraHome
[root@gjsy /]# chown -R oracle.dba /opt/oraInventory
[root@gjsy /]# chown -R oracle.dba /var/opt/oracle
[root@gjsy /]# chmod -R 755 /opt/oracle
[root@gjsy /]# chmod -R 755 /opt/oraInventory
[root@gjsy /]# chmod -R 755 /var/opt/oracle
```
把oracle安装程序拷到用户目录下面
>[root@gjsy /]# cp –r /opt/database /home/oracle/

把目录设置成具有读写和执行的权限，用户和组改为oracle.oinstall
```
[root@gjsy /]# chmod -R 755 /home/oracle/database
[root@gjsy /]# chown -R oracle /home/oracle/database
[root@gjsy /]# chgrp -R oinstall /home/oracle/database
```

<font color=#FF0000 size=5><p align="center">可选配置<配置内核参数和资源限制>(根据实际设置)</p></font>

```
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

<--! eg: [root@gjsy /]# sysctl -w net.core.rmem_default = 262144 -->
```
设置用户oracle的环境变量(必填项)
```
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
```
注销root用户,用oracle用户登录,进入/home/oracle/database目录运行图形界面开始安装
```
[oracle@diwj ~]$ cd database
[oracle@diwj database]$ ./runInstaller
```
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
指定安装目录 Oracle Base：/opt/oracle && Software Location:/opt/oracle/product/OraHome

![image](https://github.com/dwjlw1314/DWJ-PROJECT/raw/master/PictureSource/oraclegR1/3.14.8.jpg)
选择Inventory Directory:/opt/oraInventory

![image](https://github.com/dwjlw1314/DWJ-PROJECT/raw/master/PictureSource/oraclegR1/3.14.9.jpg)
选择相应的Group：oper

![image](https://github.com/dwjlw1314/DWJ-PROJECT/raw/master/PictureSource/oraclegR1/3.14.10.jpg)
选择Fix&Check Again,可以选择忽略一些不重要的问题,打开终端，以root登陆
```
#cd /tmp/CVU_11.2.0..3.0_oracle
#./runfixup.sh
```
![image](https://github.com/dwjlw1314/DWJ-PROJECT/raw/master/PictureSource/oraclegR1/3.14.11.jpg)
若存在这些警告，在linux系统安装这些包,然后重新选择Check Again或者勾选【Ignore All】

![image](https://github.com/dwjlw1314/DWJ-PROJECT/raw/master/PictureSource/oraclegR1/3.14.12.jpg)
选择Install

![image](https://github.com/dwjlw1314/DWJ-PROJECT/raw/master/PictureSource/oraclegR1/3.14.13.jpg)
出现提示进入相应目录，用root执行2个脚本
```
#./orainstRoot.sh
#./root.sh
```
![image](https://github.com/dwjlw1314/DWJ-PROJECT/raw/master/PictureSource/oraclegR1/3.14.14.jpg)
数据库安装成功

安装监听器，oracle用户进入图形界面(net Configuration Assisstant 创建、修改数据库监听)
>[oracle@diwj database]$ netca

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
```
[oracle@diwj database]$ ps -ef | grep LISTENER
oracle    24563     1   0  20:30  ?     00:00:00  /opt/oracle/product/OraHome/bin/
tnslsnr LISTENER -inherit
oracle    24605  10528  0  20:31 pts/0  00:00:00  grep LISTENER
```
新建数据库，oracle用户进入图形界面(Database Configuration Assisstant 创建数据库、更改数据库或删除数据库)
>[oracle@diwj database]$ dbca

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

数据库创建完成，选择 password management,设置system用户密码,完成后选择Exit

数据库监听程序，可以修改listen文件(可选步骤)
>[oracle@dwj admin]$ netmgr

查看该数据库OPatch版本
>[oracle@dwj ~]$ $ORACLE_HOME/OPatch/opatch lsinventory -detail -oh $ORACLE_HOME

<font color=#FF0000 size=5><p align="center">数据库基本术语概念</p></font>

```
SID：是建立一个数据库时系统自动赋予的一个初始ID,主要用于在一些DBA操作以及与操作系统交互，从操作系统的角度访问实例名，
     windows系统下是在注册表HKLM\SOFTWARE\ORACLE中ORACLE_SID变量中。linux系统查看方式：
     [oracle@dwj ~]$ env   #两者皆可
     [oracle@dwj ~]$ sqlplus / as sysdba
     SQL> show parameter instance_name

数据库：依托于实例运行，可以有不同实例加载数据库，但是一个实例只能加载一个数据库
实例：一个运行的服务，不含任何物理数据和内容。一个操作系统可以运行多个数据库实例
表空间：oracle不同用户可以设置不同表空间的访问权，相当于把数据库划分出多个子模块，供不同用户使用
```

数据库安装成功后访问 EM express
```
通过数据库查看em端口：
SQL> select dbms_xdb_config.gethttpsport() from dual;
SQL> select dbms_xdb_config.gethttpport() from dual;
设置em端口： 命令执行后，将取消https端口，重新创建一个http端口(端口号为5502)(可选操作)
SQL> exec dbms_xdb_config.setHTTPSport();
SQL> exec DBMS_XDB_CONFIG.setHTTPPort(5502);
[oracle@dwj ~]$ lsnrctl status
找到后面类似内容 ADDRESS=(PROTOCOL="tcps")(HOST=dwj)(PORT=5500))(Security=*
EM express的URL为：https://hostip:5500/em 如果引号部分是tcp，就要把https换成http
```
在EM Express中可以实现对以下诸类信息的展现和管理

![image](https://github.com/dwjlw1314/DWJ-PROJECT/raw/master/PictureSource/oraclegR1/3.14.36.jpg)

SQL 分为两个部分：数据操作语言 (DML) 和 数据定义语言 (DDL)

SQL (结构化查询语言)是用于执行查询的语法。但是 SQL 语言也包含用于更新、插入和删除记录的语法
```
SELECT - 从数据库表中获取数据
UPDATE - 更新数据库表中的数据
DELETE - 从数据库表中删除数据
INSERT INTO - 向数据库表中插入数据
```
SQL 数据定义语言部分可以创建或删除表格。也可以定义索引(键)，规定表之间的链接，以及施加表间的约束
```
CREATE DATABASE - 创建新数据库
ALTER DATABASE  - 修改数据库
CREATE TABLE    - 创建新表
ALTER TABLE     - 变更(改变)数据库表
DROP TABLE      - 删除表
CREATE INDEX    - 创建索引(搜索键)
DROP INDEX      - 删除索引
```

<font color=#FF0000 size=5><p align="center">oracle的trace清理</p></font>

1.清理adump目录，清理参数audit_file_dest指定的目录，清理的文件为*.aud
```
SQL> show parameter audit_file_dest  <br>
[root@dwj ~]$ find $ORACLE_BASE/admin/santdb/adump -mtime +7 -name "*.aud" | xargs rm -rf
```
2.清理trace文件,清理参数background_dump_dest指定的目录，清理的文件为*.tr
```
SQL> show parameter background_dump_dest
[root@dwj ~]$ find $ORACLE_BASE/diag/rdbms/santdb/santdb1/trace -mtime +7 -name "*.trc" | xargs rm -rf
[root@dwj ~]$ find $ORACLE_BASE/diag/rdbms/santdb/santdb1/trace -mtime +7 -name "*.trm" | xargs rm -rf
```
3.清理xml日志,清理路径为：$ORACLE_BASE/diag/rdbms/$DB_UNIQUE_NAME/ORACLE_SID/alert，清理文件为log_*.xml
>[root@dwj ~]$ find $ORACLE_BASE/admin/diag/rdbms/orcl/orcl/alert -mtime +7 -name "log_*.xml" | xargs rm -rf

4.清理监听日志,清理路径为：$GRID_BASE/diag/tnslsnr/NODE_NAME/listener/alert，清理文件为log_*.xml
>[root@dwj ~]$ find $ORACLE_BASE/admin/diag/rdbms/orcl/orcl/alert -mtime +7 -name "log_*.xml" | xargs rm -rf
