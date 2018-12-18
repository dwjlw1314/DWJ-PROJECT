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
