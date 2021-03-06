<font color=#FF0000 size=5> <p align="center">ORA-00845</p></font>

```
command: SQL> startup
descript: MEMORY_TARGET not supported on this

oracle的官方解释是：

Starting with Oracle Database 11g, the Automatic Memory Management feature requires more shared memory (/dev/shm)and file descriptors. The size of the shared memory should be at least the greater of MEMORY_MAX_TARGET and MEMORY_TARGET for each Oracle instance on the computer. If MEMORY_MAX_TARGET or MEMORY_TARGET is set to a non zero value, and an incorrect size is assigned to the shared memory, it will result in an ORA-00845 error at startup.

MEMORY_MAX_TARGET的设置不能超过/dev/shm的大小，在oracle11g中新增的内存自动管理的参数MEMORY_TARGET,它能自动调整SGA和PGA，这个特性需要用到/dev/shm共享文件系统，而且要求/dev/shm必须大于MEMORY_TARGET，如果/dev/shm比MEMORY_TARGET小就会报错

解决方案：
[root@dwj ~]# vim /etc/fstab      #修改2个地方
1.swap后面修改成defaults,size=11G
2.tmpfs后面修改成defaults,size=11G

修改完后，需要重新挂载一下，才能生效：
[root@dwj ~]# mount -o remount,size=11G /dev/shm
```

<font color=#FF0000 size=5> <p align="center">EXP-00008</p></font>

```
[root@dwj ~]# exp c##antman/ant@orcl file=~/backup.dmp log=~/backup.log full=y
descript: ORACLE error 1013 encountered

解决方案：
#以DBA身份连接数据库执行两条语句
[oracle@diwj ~]$ sqlplus / as sysdba
SQL> grant execute on SYS.DBMS_DEFER_IMPORT_INTERNAL to c##antman;
SQL> grant execute on SYS.DBMS_EXPORT_EXTENSION to c##antman;
```

<font color=#FF0000 size=5> <p align="center">EXP-00056/00000、ORA-12154错误组合</p></font>

```
[root@dwj ~]# exp c##antman/ant@orcl file=~/backup.dmp log=~/backup.log full=y
EXP-00056: ORACLE error 12154 encountered
ORA-12154: TNS:could not resolve the connect identifier specified
EXP-00000: Export terminated unsuccessfully

解决方案：
[oracle@diwj ~]$ cat $ORACLE_HOME/network/admin/tnsnames.ora
查看文件(该文件路径可以通过lsnrctl status查看)，内容如下：
PGUAYAS =
  (DESCRIPTION =
    (ADDRESS = (PROTOCOL = TCP)(HOST = DBserver)(PORT = 1521))
    (CONNECT_DATA =
      (SERVER = DEDICATED)
      (SERVICE_NAME = pguayas)
    )
  )
然后修改执行导出表命令如下，导出成功
[root@dwj ~]# exp c##antman/ant@PGUAYAS file=~/backup.dmp log=~/backup.log full=y
```

<font color=#FF0000 size=5> <p align="center">ORA-01756</p></font>
```
错误描述：ORA-01756: quoted string not properly terminated

解决方案：这是由于引号内的字符串没有正确结束,或者sql语句不完整或有语法错误
```

<font color=#FF0000 size=5> <p align="center">EXP-00091</p></font>

```
[root@dwj ~]# expdp antman/ant@ocrl file=~/backup.dmp log=~/backup.log owner=antman
EXP-00091: Exporting questionable statistics

oracle的官方描述是：
Cause: Export was able export statistics, but the statistics may not be usuable. The statistics are questionable because one or more of the following happened during export: a row error occurred, client character set or NCHARSET does not match with the server, a query clause was specified on export, only certain partitions or subpartitions were exported, or a fatal error occurred while processing a table.
Action: To export non-questionable statistics, change the client character set or NCHARSET to match the server, export with no query clause, export complete tables. If desired, import parameters can be supplied so that only non-questionable statistics will be imported, and all questionable statistics will be recalculated.

解决方案：
查看database中的NLS_CHARACTERSET的值
SQL> select * from v$nls_parameters where parameter='NLS_CHARACTERSET';

根据第一步查出来的NLS_CHARACTERSET(即AL32UTF8)来设定
windows环境：cmd > set NLS_LANG=AMERICAN_AMERICA.AL32UTF8
linux环境：[root@dwj ~]# export NLS_LANG=AMERICAN_AMERICA.AL32UTF8
```

<font color=#FF0000 size=5> <p align="center">ORA-03113</p></font>

```
SQL> startup     #Oracle启动时报如下错误：
ORA-03113: end-of-file on communication channel

查看oracle启动日志，确定具体错误原因
操作日志:$ORACLE_HOME/startup.log
启动日志:$ORACLE_BASE/diag/rdbms/orcl/orcl/trace/alert_orcl.log
启动日志如果找不到，可以在trace目录下执行ls -alcr|grep alert(c时间排序,r倒序)

归档日志满了->解决方案有3个：

#将归档设置到其他目录
SQL> alter system set log_archive_dest = 其他路径;
#转移或者删除闪回区里的归档日志(本次错误采用这种修复方式)
#增大闪回恢复区
SQL> alter system set db_recovery_file_dest_size = 8G;

1. 启动到mount状态
SQL> startup mount;

2.查看恢复区（闪回区）位置及大小
SQL> show parameter db_recovery;

3.查询当前的使用状态
SQL> select file_type,PERCENT_SPACE_USED,NUMBER_OF_FILES from v$flash_recovery_area_usage

4.物理清除归档路径下的日志文件,物理日志文件清理后，还需要在ramn管理中清理一次，不然还是显示的空间没有释放
[oracle@dwj ~]# cd /opt/oracle/fast_recovery_area/orcl/ORCL/archivelog
[oracle@dwj archivelog]$ rm -rf [delete_dir]

5.通过rman管理工具清理
RMAN> crosscheck backup;
RMAN> delete obsolete;
RMAN> delete expired backup;
RMAN> crosscheck archivelog all;
RMAN> delete expired archivelog all;

6.删除完毕后查看使用结果
SQL> select * from V$FLASH_RECOVERY_AREA_USAGE;

7.重新启动数据库ok
```

<font color=#FF0000 size=5> <p align="center">ORA-65096</p></font>

```
错误描述：ORA-65096: invalid common user or role name错误

解决方案：
查看12c DB是否为容器数据库
SQL> select cdb from v$database;

查看12c DB当前用户是 COMMON_USERS 还是 LOCAL_USERS 模式
SQL> show con_name

如果是 CDB$ROOT 模式，需要使用如下方式创建用户
SQL> create user c##ant identified by ant;

切换 ORCLPDB 模式,就可以使用旧模式创建用户
SQL> alter session set container=ORCLPDB;

查看12c DB服务上ORCLPDB容器状态
SQL> select con_id, dbid, guid, name, open_mode from v$pdbs;

新建用户出现 ORA-01109: database not open错误
SQL> alter pluggable database ORCLPDB open;
```

<font color=#FF0000 size=5> <p align="center">ORA-65048</p></font>

```
[root@dwj ~]# create user c##ogg identified by ogg default tablespace OGG
              temporary tablespace TEMP quota unlimited on OGG;
descript:
ORA-65048: error encountered when processing the current DDL statement in pluggable database ORCLPDB
ORA-00959: tablespace 'OGG' does not exist

解决方案：(在其他PDB内，也创建同名的表空间)
SQL> alter session set container=orclpdb;
Session altered.

#datafile路径需要正确指定
SQL> create tablespace OGG logging datafile '/opt/oracle/oradata/ogg.dbf' size 20m autoextend on uniform size 2m;

SQL> alter session set container=cdb$root;
Session altered.

SQL> create user c##ogg identified by ogg default tablespace OGG temporary tablespace TEMP quota unlimited on OGG;
User created.
```

<font color=#FF0000 size=5> <p align="center">ORA-30036</p></font>

```
错误描述：ORA-30036: unable to extend segment by 8 in undo tablespace 'UNDOTBS3'

解决方案：
1.查询undo表空间的使用大小空间
SQL> select a.tablespace_name,ROUND(a.total_size) "total_size(MB)",
ROUND(a.total_size) - ROUND(b.free_size, 3) "used_size(MB)",
ROUND(b.free_size, 3) "free_size(MB)",
ROUND(b.free_size / total_size * 100, 2) || '%' free_rate
from (select tablespace_name, SUM(bytes) / 1024 / 1024 total_size
from dba_data_files group by tablespace_name) a,
(select tablespace_name, SUM(bytes) / 1024 / 1024 free_size from dba_free_space
group by tablespace_name) b where a.tablespace_name = b.tablespace_name(+);

2.增加undo表空间大小
SQL> alter database datafile 'D:\ORACLE\PRODUCT\10.2.0\ORADATA\SUREDD\UNTOTBS_NEW_01.DBF' resize 2048M;

3.给undo表空间新增dbf文件，语句参考《oracle数据库基本命令》
```

<font color=#FF0000 size=5> <p align="center">ORA-31626</p></font>

```
[oracle@GuayaDB3 dir]$ expdp antman/ant parfile=gps_data20181101.par
错误描述：expdp ORA-31626: job does not exist

这种错误通常都是由于Oracle软件升级之后和库不一致产生的，需要重新执行SQL来配置后台数据字典

解决方案：
SQL> @?/rdbms/admin/catalog.sql
SQL> @?/rdbms/admin/catproc.sql

后续此库出现问题：EXP-00056: ORACLE error 932 encountered
SQL> connect / as sysdba
SQL> @?/rdbms/admin/catmetx.sql
SQL> @?/rdbms/admin/utlrp.sql
```

<font color=#FF0000 size=5> <p align="center">ORA-01502</p></font>

```
错误描述：ORA-01502: index or partition of such index is in usable state tips

oracle的官方描述是：
Cause: An attempt has been made to access an index or index partition that has been marked unusable by a direct load or by a DDL operation
Action: DROP the specified index, or REBUILD the specified index, or REBUILD the unusable index partition

说明：
1.在session级别跳过无效索引作查询
SQL> alter session set skip_unusable_indexes=true;

2.分区索引应适用user_ind_partitions

3.状态分4种：
  N/A说明这个是分区索引需要查user_ind_partitions或者user_ind_subpartitions来确定每个分区是否可用
  VAILD说明这个索引可用
  UNUSABLE说明这个索引不可用
  USABLE 说明这个索引的分区是可用的

4.查询当前索引的状态
SQL> select distinct status from user_indexes;

5.查询那个索引无效
SQL> select index_name from  user_indexes where status <> 'VALID';

7.批量rebuild下
SQL> select 'alter index '||index_name||' rebuild online;' from  user_indexes where status <> 'VALID' and index_name not like'%$$';

解决方案：
1.重建表索引
SQL> alter index index_name rebuild (online);  或者
SQL> alter index index_name rebuild;

2.如果是分区索引只需要重建那个失效的分区
SQL> alter index index_name rebuild partition SYS_P123322 (online);  或者
SQL> alter index index_name rebuild partition SYS_P123322;

3.或者改变当前索引的名字
```

<font color=#FF0000 size=5> <p align="center">ORA-31626</p></font>

```
错误描述：ORA-04030: out of process memory when trying to allocate 20504 bytes

oracle的官方描述是：
ORA-04030 out of process memory when trying to allocate string bytes (string,string)
Cause: Operating system process private memory has been exhausted.
Action: See the database administrator or operating system administrator to increase process memory quota. There may be a bug in the application that causes excessive allocations of process memory space.

需要注意的是max_sga_size和sga_target的设置：
sga_max_size指的是可动态分配的最大值﹐而sga_target是当前已分配的最大sga
sga_max_size是不可以动态修改的﹐而sga_target是可动态修改﹐直到sga_max_size的值

如果在实例启动时﹐sga_max_size < sga_target或sga_max_size没设定﹐则启动后sga_max_size的值会等于sga_target的值
如果内存占用超过sga_target，也可能会出现ORA-04030的错误

解决方案：
查看数据库的内存分配图
SQL> show parameter pool
SQL> show parameter sga
SQL> show parameter pga

SQL查看结果：
sga_target=30G
sga_max_size=32G
pga_aggregate_target=16G

1.设置rman从SGA取内存
SQL> alter system set dbwr_io_slaves=2 scope=spfile;
SQL> alter system set backup_tape_io_slaves=true scope=spfile;

2.调整SGA大小
SQL> alter system set sga_target=30G;
SQL> alter system set sga_max_size=32G scope=spfile;

3.设置使用内存最大大小
SQL> alter system set large_pool_size=1024M;

4.重启oracle service

5.查看sga,pga,pool的大小
```

<font color=#FF0000 size=5> <p align="center">ORA-00054</p></font>

```
SQL> truncate table vehicle_location;
错误描述：ORA-00054: resource busy and acquire with NOWAIT specified

解决方案：
SQL> select * from v$session s,v$locked_object l where l.PROCESS = s.PROCESS
and module = 'Storage.exe'

通过上面得到的sid和serial#，然后对该进程进行终止
SQL> alter system kill session 'sid,serial#';

一些ORACLE中的进程被杀掉后，状态被置为"killed"，但是锁定的资源很长时间不释放
1.重启数据库
2.在OS一级对其kill

首先执行下面的语句获得进程(线程)号：
SQL> select spid, osuser, s.program from v$session s,v$process p where s.paddr=p.addr and s.sid=24

在unix上，用root身份执行命令:
[root@dwj ~]# kill -9 12345 (即上面查询出的spid)

在windows(unix也适用)用oracle提供的一个可执行命令orakill杀死线程，语法为：orakill sid thread
sid：表示要杀死的进程属于的实例名
thread：是要杀掉的线程号，即上面查询出的spid
eg: c:> orakill orcl 12345
```

<font color=#FF0000 size=5> <p align="center">ORA-22992</p></font>

```
SQL> select * from USER_AUTHORITY@SANDB3
错误描述：ORA-22992: cannot use LOB locators selected from remote tables

解决方案：
1、创建一张临时表
SQL> create global temporary table table_temp as select * from USER_AUTHORITY where 1=2

2、然后利用database link把远程数据先insert到临时表中，insert后先不要commit，否则commit后临时表中数据就会丢失
SQL> insert into table_temp select * from USER_AUTHORITY@SANDB3

3、将临时表中的数据insert到目标库表
SQL> insert into USER_AUTHORITY select * from table_temp

4、完毕，将临时表drop掉
SQL> drop table table_temp
```

<font color=#FF0000 size=5> <p align="center">ORA-02292</p></font>

```
错误描述：ORA-02292: integrity constraint MGV_VEHICLE_FK violated

解决方案：
1. 查找相关约束信息,其中constraint_name是报错中提示点外键约束名
SQL> select a.constraint_name, a.table_name, b.constraint_name
      from user_constraints a, user_constraints b
      where a.constraint_type = 'R'
      and b.constraint_type = 'P'
      and a.r_constraint_name = b.constraint_name
      and a.constraint_name = 'MGV_VEHICLE_FK'

2. 删除子表中的所有记录
3. 再次删除主表报错数据
```

<font color=#FF0000 size=5> <p align="center">OGG-00868</p></font>

```
GGSCI (GuayaDB1) 2> start ANTEX_G
错误描述：OGG-00868  Error code 1291, error message: ORA-01291: missing logfile
 (Missing Log File <unknown>. Read Position SCN: Unknown)

解决方案：
GGSCI (GuayaDB1) 3> dblogin userid ggu,password ggu
Successfully logged into database.

GGSCI (GuayaDB1 as antman@santdb1) 4> UNREGISTER EXTRACT ANTEX_G DATABASE
2019-05-06 17:51:10 INFO OGG-01750  Successfully unregistered EXTRACT ANTEX_G from database.

GGSCI (GuayaDB1 as antman@santdb1) 5> REGISTER EXTRACT ANTEX_G DATABASE
2019-05-06 17:17:19 INFO OGG-02003  Extract ANTEX_G successfully registered with database at SCN 452197.
```

<font color=#FF0000 size=5> <p align="center">SP2-0734</p></font>

```
SQL> @/home/oracle/.xx.sql
错误描述：
P2-0734: unknown command beginning "WHEN conn...." - rest of line ignored.
SP2-0734: unknown command beginning "cat.catalo..." - rest of line ignored.
SP2-0734: unknown command beginning "END Qualif..." - rest of line ignored.
SP2-0042: unknown command "FROM" - rest of line ignored.
SP2-0044: For a list of known commands enter HELP
and to leave enter EXIT.

解决方案：
1. 将sql文件语句总所有中文注释改为英文；
2. sql文件开头使用 set sqlblanklines on 因为空行导致sql语句加载到机器内存中截断了
```

<font color=#FF0000 size=5> <p align="center">SP2-0341</p></font>

```
SQL> @/home/oracle/.xx.sql
错误描述：SP2-0341: line overflow during variable substitution (>3000 characters at line 1)

解决方案：
1. 将sql文件行进行折行处理；
2. 设置行显示大小  set linesize 4000
```

<font color=#FF0000 size=5> <p align="center">ORA-12514</p></font>

```
错误描述：ORA-12514  TNS:listener does not currently know of service requested in connect descriptor

问题描述：Oracle 12c 修改RAC集群的rac-scan

1.查看scan ip的状态信息
[grid@dwj ~]# srvctl config scan

2.停止scan_listener,scan
[grid@dwj ~]# srvctl stop scan_listener
[grid@dwj ~]# srvctl stop scan

3.确认 scan_listener,scan 的状态
[grid@dwj ~]# srvctl status scan_listener
[grid@dwj ~]# srvctl status scan

4.在所有节点中 /etc/hosts 文件中修改 rac-scan 对应的ip

5.查看srvctl命令所在文件夹
[grid@dwj ~]# cd $ORACLE_HOME
[grid@dwj ~]# cd bin/ && pwd

6. 使用root命令修改rac-scan，即修改为/etc/hosts里面rac-scan对应的ip
[root@dwj /u01/app/11.2.0/grid/bin]# ./srvctl modify scan -n rac-scan
注：-n后面跟的是 /etc/hosts 下 rac-scan 的名称
[root@dwj /u01/app/11.2.0/grid/bin]# ./srvctl config scan

7. 启动scan_listener,scan并查看状态
[grid@dwj ~]# srvctl status scan
[grid@dwj ~]# srvctl start scan_listener
[grid@dwj ~]# srvctl start scan

8. 使用新的scan ip测试连接
```

<font color=#FF0000 size=5> <p align="center">ORA-15021</p></font>

```
[grid@dwj ~]# rman target /
错误描述：
ORACLE error: ORA-15021: parameter "remote_dependencies_mode" is not valid in ASM instance

解决方案：
[grid@dwj ~]# export ORACLE_SID=实例名
```

<font color=#FF0000 size=5> <p align="center">ORA-28007</p></font>

```
[grid@dwj ~]# rman target /
错误描述：
ORACLE error: ORA-28007: the password cannot be reused

解决方案：
SQL> ALTER PROFILE DEFAULT LIMIT PASSWORD_REUSE_MAX UNLIMITED;
SQL> ALTER PROFILE DEFAULT LIMIT PASSWORD_REUSE_TIME UNLIMITED;
SQL> ALTER USER user ACCOUNT UNLOCK;
SQL> ALTER USER user identified by 123;

相关问题关联描述：
SQL> select profile, resource_name, limit from dba_profiles where
     resource_name in('PASSWORD_REUSE_TIME','PASSWORD_REUSE_MAX') and profile = 'DEFAULT';
resource_name说明:
FAILED_LOGIN_ATTEMPTS    ---> 不知道口令的话尝试登录的次数，达到这个次数之后账户被自动锁定
PASSWORD_LIFE_TIME       ---> 口令的生命周期，超过这段时间口令可能会自动过期，是否过期要看是否设定
PASSWORD_REUSE_TIME      ---> 这个特性限制口令在多少天内不能重复使用
PASSWORD_REUSE_MAX       ---> 这个特性是针对PASSWORD_REUSE_TIME的，说明要想在PASSWORD_REUSE_TIME这个参数指定的时间内重复使用当前口令，那么至少需要修改过口令的次数(修改过的口令当然肯定需要和当前口令不同，因为毕竟还有PASSWORD_REUSE_TIME 特性的限制)
PASSWORD_VERIFY_FUNCTION ---> 密码验证规则函数
PASSWORD_LOCK_TIME       ---> 接着FAILED_LOGIN_ATTEMPTS参数，口令被自动锁定的时间，达到这个时间之后，下次登录时系统自动解除对这个账户的锁定
PASSWORD_GRACE_TIME      ---> 接着PASSWORD_LIFE_TIME特性，如果PASSWORD_LIFE_TIME的期限已到，继续可以使用的天数，在这段时间内如果我们登录系统，会有提示，提示系统在几天内过期
```

<font color=#FF0000 size=5> <p align="center">SP2-0341</p></font>

```
SQL> @/home/oracle/.xx.sql
错误描述：SP2-0341: line overflow during variable substitution (>3000 characters at line 1)

解决方案：
1. 将sql文件行进行折行处理；
2. 设置行显示大小  set linesize 4000
```

<font color=#FF0000 size=5> <p align="center">恢复unused列的方法</p></font>
```
设置unused的作用是为了在cpu、内存等资源不充足的时候，先做上unused标记再等数据库资源空闲的时候用drop set unused删除
设置unused列之后，并不是将该列数据立即删除，而是被隐藏起来，物理上还是存在的

查询要操作表对象的ID
1. SQL> SELECT OBJECT_ID,OBJECT_NAME FROM USER_OBJECTS;    #52717 TEST----OBJECT_ID=52717

2. SQL> conn / as sysdba

3. SQL> select col#,intcol#,name from col$ where obj#=52717;  #SYS_C00003_12092313:06:51$----原来的列名被系统修改

4. SQL> select cols from tab$ where obj#=52717;   #2----系统字段数目发生变化计数

5. SQL> update col$ set col#=intcol# where obj#=52717;

6. SQL> update tab$ set cols=cols+1 where obj#=52717;

7. SQL> update col$ set name='column_name' where obj#=52717 and col#=3;  #column_name是要改动列名

8. SQL> update col$ set property=0 where obj#=52717;

9. SQL> commit;

10. SQL> startup force;  #这一步是必不可少的
```
