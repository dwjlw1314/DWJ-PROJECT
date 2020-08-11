<font color=#FF0000 size=5> <p align="center">Oracle 性能相关查询</p></font>

查询用户 CPU 的使用率,用来显示每个用户的 CPU 使用率
```sql
select ss.username, se.SID, VALUE / 100 cpu_usage_seconds
    from v$session ss, v$sesstat se, v$statname sn
    where se.STATISTIC# = sn.STATISTIC#
    and name like '%CPU used by this session%'
    and se.SID = ss.SID
    and ss.status = 'ACTIVE'
    and ss.username is not null
    order by VALUE desc;
```
获取当前会话 ID，进程 ID，客户端 ID
```sql
select b.sid, b.serial#, a.spid processid, b.process clientpid
    from v$process a, v$session b
    where a.addr = b.paddr
    and b.audsid = userenv('sessionid');
```
在视图中查询并显示实际的 Oracle 连接
```sql
select osuser, username, machine, program
    from v$session
    order by osuser;
```
查询并显示连接 Oracle 的用户和用户的会话数量
```sql
select username usuario_oracle, count (username) numero_sesiones
    from v$session
    group by username
    order by numero_sesiones desc;
```
获取拥有者的对象数量
```sql
select owner, count (owner) number_of_objects
    from dba_objects
    group by owner
    order by number_of_objects desc;
```
会话使用的临时表排序空间
```sql
select S.sid || ',' || S.serial# sid_serial, S.username, S.osuser, P.spid,
      S.module, S.program, SUM(T.blocks) * TBS.block_size / 1024 / 1024 mb_used,
      T.tablespace, COUNT(*) sort_ops
    from v$sort_usage T, v$session S, dba_tablespaces TBS, v$process P
    where T.session_addr = S.saddr
    and S.paddr = P.addr and T.tablespace = TBS.tablespace_name
    group by S.sid, S.serial#, S.username, S.osuser, P.spid, S.module,
      S.program, TBS.block_size, T.tablespace
 order by sid_serial;
```
查询数据库sql命令相关内容以及SQL_TEXT
```sql
select sess.SID,sess.SERIAL#,sqls.SQL_ID,sort.SEGTYPE,sort.BLOCKS * 8 / 1000 "MB",sqls.SQL_TEXT
    from v$sort_usage sort, v$session sess, v$sql sqls
    where sort.SESSION_ADDR = sess.SADDR
    and sqls.ADDRESS = sess.SQL_ADDRESS
    order by sort.BLOCKS desc;
```
查看数据表空间使用情况(undo)
```sql
select a.tablespace_name,
    to_char(b.total/1024/1024,999999.99) as Total,
    to_char((b.total-a.free)/1024/1024,999999.99) as Used,
    to_char(a.free/1024/1024,999999.99) as Free,
    to_char(round((total-free)/total,4)*100,999.99) as Used_Rate
    from (select tablespace_name, sum(bytes) free from DBA_FREE_SPACE group by  tablespace_name) a,
        (select tablespace_name, sum(bytes) total from DBA_DATA_FILES group by tablespace_name ) b
    where a.tablespace_name=b.tablespace_name
    order by a.tablespace_name;
```
```sql
select a.tablespace_name tablespace_name,
total/1024/1024 total_size, free/1024/1024 free_size,
(total - free)/1024/1024 used_size,
Round(( total - free ) / total, 4) * 100 "usage%"
from (select tablespace_name, Sum(bytes) free from DBA_FREE_SPACE
group by tablespace_name) a,
(select tablespace_name, Sum(bytes) total from DBA_DATA_FILES
group by tablespace_name) b
where a.tablespace_name = b.tablespace_name;
```
查看临时表空间使用情况(temp)
```sql
select round((f.bytes_free  + f.bytes_used)/1024/1024/1024, 2) as "total(gb)",
    round(((f.bytes_free  + f.bytes_used) - nvl(p.bytes_used, 0))/1024/1024/1024,2) as "free(gb)",
    d.file_name as "temp_file",
    round(nvl(p.bytes_used, 0)/1024/1024/1024, 2) as "used(gb)" ,
    round((f.bytes_used + f.bytes_free)/1024/1024/1024, 2) as "total(gb)",
    round(((f.bytes_used + f.bytes_free) - nvl(p.bytes_used, 0))/1024/1024/1024, 2) as "free(gb)" ,
    round(nvl(p.bytes_used, 0)/1024/1024/1024, 2) as "used(gb)"
    from sys.v_$temp_space_header f ,dba_temp_files d ,sys.v_$temp_extent_pool p
    where f.tablespace_name(+) = d.tablespace_name
    and f.file_id(+) = d.file_id
    and p.file_id(+) = d.file_id;
```
查询数据表空间剩余字节大小
```sql
select TABLESPACE_NAME, SUM(BYTES)/1024/1024 as "FREE SPACE(M)"
    from DBA_FREE_SPACE
    where TABLESPACE_NAME = '&tablespace_name'
    group by TABLESPACE_NAME;
```
查询临时表空间剩余字节大小
```sql
select TABLESPACE_NAME, FREE_SPACE/1024/1024 as "FREE SPACE(M)"
    from DBA_TEMP_FREE_SPACE
    where TABLESPACE_NAME = '&tablespace_name';
```
查找消耗临时表空间资源比较多的SQL语句
```sql
select se.USERNAME, se.SID||','||se.SERIAL# "sid,serial", s.SQL_ID, s.MODULE, su.EXTENTS,
    su.BLOCKS * to_number(rtrim(p.VALUE)) as space, tablespace, segtype, sql_text
    from v$sort_usage su, v$parameter p, v$session se, v$sql s
    where p.name = 'db_block_size'
    and su.session_addr = se.saddr
    and s.hash_value = su.sqlhash
    and s.address = su.sqladdr
    order by se.username, se.sid;
```
查看回滚段状态和名字
```sql
select t.segment_name, t.tablespace_name, t.segment_id, t.status
    from dba_rollback_segs t
    where t.tablespace_name like 'UN%'
```
循环添加表空间
```sql
declare
    v_i int;
    loop_times int;
    tmp_str varchar(100);
    strsql varchar(4000);
begin
    v_i := 1000;
    loop_times := 1;
    while loop_times < 129 loop
        v_i := v_i + 1;
        tmp_str := 'ANG_COMMON_DATA_HASH_'||substr(to_char(v_i),2,3);

        loop_times := loop_times + 1;
        strsql := 'create tablespace '||tmp_str||' datafile '||chr(39)||
               '+OCR/PRO_BUSI/DATAFILE/'||tmp_str||chr(39)||' size 100m autoextend on';
        execute immediate strsql;
    end loop;
 end;
```

<font color=#FF0000 size=5> <p align="center">Oracle dbstart和dbshut</p></font>

一、用 dbstart 和 dbshut 启动和关闭数据库实例

先启动监听
>[oracle@dwj bin]$ lsnrctl start

启动实例
>[oracle@dwj bin]$ dbstart

第一次使用 dbstart 命令可能会报如下错误
```
ORACLE_HOME_LISTNER is not SET, unable to auto-start Oracle Net Listener
Usage: /opt/oracle/product/OraHome/bin/dbstart ORACLE_HOME

编辑dbstart,找到ORACLE_HOME_LISTNER=$1，修改为ORACLE_HOME_LISTNER=$ORACLE_HOME
编辑/etc/oratab,找到orcl:/opt/oracle/product/OraHome:N 把 "N" 修改为 "Y"
```
关闭数据库实例
>[oracle@dwj bin]$ dbshut

关闭监听
>[oracle@dwj bin]$ lsnrctl stop

二、数据库实例和linux系统一起启动

编辑 /etc/rc.d/rc.local 中加入如下语句即可实现与系统同步启动实例：
```
su - oracle -c "lsnrctl start"
su - oracle -c "dbstart"
```

Oracle sqlplus登陆方式
```
以 sys 用户登陆的话 必须要加上 as sysdba 子句

#以操作系统权限认证的sys管理员登陆
[oracle@dwj ~]$ sqlplus / as sysdba
[oracle@dwj ~]$ sqlplus sys/123456@orcl as sysdba

#以操作系统权限认证的sysbackup管理员登陆
[oracle@dwj ~]$ sqlplus / as sysbackup

#不在terminal当中暴露密码的登陆方式
[oracle@dwj ~]$ sqlplus /nolog
SQL> conn / as sysdba   
SQL> conn sys/password as sysdba

#非管理员用户登陆
[oracle@dwj ~]$ sqlplus scott/tiger

#不显露密码的登陆方式
[oracle@dwj ~]$ sqlplus
Enter user-name: system
Enter password:

#非管理员用户使用tns别名登陆
[oracle@dwj ~]$ sqlplus scott/tiger@orcl

#管理员用户使用tns别名登陆
[oracle@dwj ~]$ sqlplus system/pwd@orcl

通过网络连接，这是需要数据库服务器的 listener 处于监听状态。此时建立一个连接的大致步骤如下:
a. 查询sqlnet.ora，看看名称的解析方式，默认是TNSNAME
b. 查询 tnsnames.ora 文件，从里边找 orcl 的记录，并且找到数据库服务器的主机名或者IP，端口和service_name
c. 如果服务器 listener 进程没有问题的话，建立与 listener 进程的连接
d. 根据不同的服务器模式如专用服务器模式或者共享服务器模式，listener采取连接动作。默认是专用服务器模式
```

unix下查看进程的最后几个字母就是sid
>[oracle@dwj ~]$ ps -ef|grep ora_ 

检查系统当前视图相关参数(v$parameter视图中查询参数的时候其实都是通过x$ksppi和x$ksppcv这两个内部视图中得到的)
```sql
SQL> select count(*) from v$instance;              #数据库实例所有参数字段
SQL> select status from v$instance;                #查看oracle启动状态
SQL> select * from dba_datapump_jobs               #查询EXP/IMP在后台执行的状态
SQL> select * from dba_tablespaces;                #查看数据库表空间信息
SQL> select * from nls_database_parameters;        #查看数据库服务器字符集
SQL> select * from user_tab_partitions;            #查看数据库表分区信息
SQL> select object_name,created from user_objects  #查看数据表名和创建时间
SQL> select name from v$tempfile;                  #查看当前用户临时表空间位置
SQL> select count(*) from v$process;               #查询数据库当前进程的连接数
SQL> select count(*) from v$session;               #查看数据库当前会话的连接数
SQL> select name from v$controlfile;               #查看控制文件
SQL> select member from v$logfile;                 #查看redo日志文件详情
SQL> select name from v$archived_log;              #查看归档日志记录
SQL> select flashback_on from v$database;          #查看闪回是否开启
SQL> select flashback_on from v$database;          #查看数据库service_names
SQL> select created,log_mode from v$database;      #查看数据库的创建日期和归档方式
SQL> select SQL_TEXT,SQL_ID,MODULE from v$sql;     #查看数据库执行语句文本内容
SQL> select * from v$pdbs;                         #12C查看容器详情
SQL> select MESSAGE from v$session_longops;        #捕捉运行很久的SQL
SQL> select * from v$locked_object;                #查看未提交的事务
SQL> select * from v$transaction;                  #查看未提交的事务,一般关联v$session
SQL> select * from v$version;                      #查看数据库的版本
SQL> select * from v$asm_disk_stat;                #查看ASM对应磁盘组以及设备名
SQL> select * from dba_data_files                  #查看表空间对应的数据文件路径
SQL> select * from dba_mviews;                     #查看物化视图刷新状态信息
SQL> select * from dba_dependencies;               #查看用户下的view和trigger
SQL> select * from dba_mview_logs                  #查询物化视图日志(快照)
SQL> select * from dba_mview_refresh_times;        #查看物化视图刷新时间
```
sqlplus的buffer会缓存最后一条sql语句，可以使用"/"来执行最后一条命令，也可以使用"edit"
编辑最后一条sql语句。l(list) 可以显示buffer中最后一条命令

sqlplus模式下查看、修改、删除列格式
>SQL> column vehicle_id   <br>
>SQL> column vehicle_id [heading 'name'] format A15  <br>
>SQL> column vehicle_id clear

系统显示语句执行错误原因查看方法
> SQL> !oerr ora [errcode]    #eg: !oerr ora 956

使用'!'可以在shell和sqlplus间切换
> SQL> !pwd

查看所有环境设置：
> SQL> show all

查看数据库归档设置详情
>SQL> archive log list

检查数据库全部或者部分参数(parameter_name可选模糊匹配)
> SQL> show parameter [parameter_name]        #eg:show parameter service_names

查询日志文件路径
> SQL> show parameter background_dump_dest

查看闪回空间大小
>SQL> show parameter db_recovery_file_dest_size

查看数据库服务器字符集方式一
> SQL> select userenv('language') from dual;

查看数据库服务器字符集方式二
>SQL> select * from v$nls_parameters where parameter='NLS_CHARACTERSET';

查询数据库允许的最大连接数(processes)或会话数(sessions)
>SQL> select value from v$parameter where name = 'processes';

查看数据库对象, owner参数需要大写
>SQL> select object_name, object_type, status from all_objects where owner='SYSTEM';

查询闪回区当前的使用状态
>SQL> select file_type,PERCENT_SPACE_USED,NUMBER_OF_FILES from v$flash_recovery_area_usage;

Oracle的物理结构主要有 1.dbf数据文件 2.log重做日志文件 3.ctl控制文件 4.ora参数文件
>SQL> select file_name,tablespace_name,BYTES/1048576 MB, maxbytes/1048576 Max_MB from dba_data_files;  <br>
>SQL> select file_name,tablespace_name,BYTES/1048576 MB, maxbytes/1048576 Max_MB from dba_temp_files;

关闭数据库，有四种不同的关闭选项
```
SQL> shutdown normal;
#发出该命令后，任何新连接都将不允许再连接到数据库。关闭之前，需要所有的连接都从数据库中退出后才开始关闭数据库。采用这种方式关闭数据库，在下一次启动时无需进行任何的实例恢复，该模式下关闭一个数据库需要几天时间，也许更长
SQL> shutdown immediate;
#当前模式会立即中断所有Oracle处理的SQL语句，任何没有提交的事务全部回滚。采用这种方式关闭数据库也需要一段时间（该事务回滚时间）。系统不等待连接到数据库的任何用户退出系统，强行回滚当前任何的活动事务，然后断开所有的连接用户
SQL> shutdown transactional;
#该选项仅在Oracle 8i后才能够使用。该命令常用来计划关闭数据库，他使当前连接到系统且正在活动的事务执行完毕，运行该命令后，任何新的连接和事务都是不允许的。活动的事务完成后，数据库将和shutdown immediate同样的方式关闭数据库
SQL> shutdown abort;
#在没有任何办法关闭数据库的情况下才不得已的方式，一般不采用
```
重新启动数据库,启动状态有3种可选模式,默认不设置启动状态
```sql
SQL> startup [启动模式];
SQL> startup nomount;                  #启动实例不加载数据库，只创建各种内存结构和服务进程，不打开任何数据库文件
SQL> select status from v$instance;    #STARTED
SQL> select open_mode from v$database; #ERROR at line 1: ORA-01507: database not mounted
SQL> alter database mount;             #启动实例加载数据库但不打开数据库，该模式下4种操作：
1.重命名数据文件
2.添加、删除和重命名文件
3.执行数据库完全恢复操作
4.改变数据库的归档模式
SQL> select status from v$instance;    #MOUNTED
SQL> select open_mode from v$database; #MOUNTED
SQL> alter database open;              #启动实例加载并打开数据库，该模式下可以设置为非受限和受限状态，4种操作：
1.执行数据导入导出
2.使用sql*loader提取外部数据
3.拒绝普通用户访问数据库
4.进行数据库移植或者升级操作
SQL> select status from v$instance;    #OPEN
SQL> select open_mode from v$database; #READ WRITE 或者 READ ONLY
```

查询某用户下的所有表详细信息(desc 适用所有视图参数eg:Tablespaces;Views;Sequences;Indexes;...)
```sql
--查询表的描述信息
SQL> select * from dba_tab_comments where owner='user';
--在dba权限下查询 user 用户下的所有表的信息
SQL> select count(*) from dba_tables where owner='user';
SQL> select count(*) from all_tables where owner='user';
--表名以及各详细内容等相应字段查询
SQL> desc dba_tables / all_tables

--在当前用户权限下查询该用户所有表的信息
SQL> select count(*) from user_tables;
--表名以及各详细内容等相应字段查询;已经不包括owner字段
SQL> desc user_tables
```
查询数据库下的所有用户视图详细信息
```sql
--在dba权限下查询用户的所有信息
SQL> select count(*) from dba_users;
SQL> select count(*) from all_users;
--用户视图所有字段查询
SQL> desc dba_users / all_users

--在当前用户权限下查询该用户所有的信息
SQL> select count(*) from user_users;
--用户视图所有字段查询
SQL> desc user_users
```
查看和启动数据库的监听端口 1521 ,其中有 Listener Parameter File 路径
>[oracle@dwj Desktop]$ lsnrctl status/start

修改数据库用户密码，显示成功后执行 commit;
>SQL> alter user sys identified by new-password;

修改数据库系统参数 <sevice_name> 的值,如果出现ORA-02095错误,添加scope = spfile;
>SQL> alter system set service_names=newvalue;

让修改数据库系统参数生效
>SQL> alter system register;

修改数据库允许的最大连接数(需要重启数据库才能实现连接数的修改)
```
SQL> alter system set processes = 300 scope = spfile;
修改spfile参数的三种模式：
scope=both       #立即并永久生效，(默认模式)
scope=spfile     #下次启动才能生效
scope=memory     #立即生效但下次启动时失效
```
数据库用户scott进行解锁
>SQL> alter user scott account unlock;

数据库用户scott进行加锁
>SQL> alter user scott account lock;

修改数据库的打开模式
>SQL> alter database open read only;   #只读模式  <br>
>SQL> alter database open read write;  #读写模式

查看归档模式参数设置
>SQL> archive log list;

<font color=#FF0000 size=5> <p align="center">创建数据库表空间</p></font>

```
第1步：查询和指定表空间目录
SQL> show parameter create
SQL> alter system set db_create_file_dest='/opt/oracle/oradata/';

第2步：创建临时表空间
SQL> create temporary tablespace gjsy_temp
tempfile '/opt/oracle/oradata/orcl/gjsy_temp.dbf'      #临时文件名
size 50m                                               #初始化大小
autoextend on                                          #自动增长
next 50m maxsize 20480m                                #每次扩展50M
extent management local;                               #本地管理表空间

第3步：创建数据表空间
SQL> create tablespace gjsy_data
logging datafile '/opt/oracle/oradata/orcl/gjsy_data.dbf'
size 50m autoextend on next 50m maxsize unlimited
extent management local;

第4步：创建用户并指定临时表空间和数据表空间
SQL> create user c##antman identified by ant default tablespace gjsy_data
     temporary tablespace gjsy_temp;

第5步：给用户授予权限(以下是不同权限的设置方式，取其一即可)
SQL> grant select any dictionary to c##antman;
SQL> grant connect,select any table to c##antman;
SQL> grant connect,resource,dba to c##antman;

撤销权限(即取消角色)，语法：revoke role-type from username;
SQL> revoke connect,resource from c##antman;
```

给用户授予细粒度权限，privileges(all、select、insert、update、delete、alter、index、references)
>SQL> grant privileges on object to c##antman;    #object 可以是tablesname

oracle为兼容以前版本，提供三种标准角色(role):connect/resource和dba
```
1.connect role(连接角色),临时用户,特指不需要建表的用户,通常只是使用oracle简单权限,这种权限只对其他用户的表有访问权限，包括select/insert/update和delete等,该角色用户还能够创建表、视图、序列(sequence)、簇(cluster)、同义词(synonym)、会话(session)和其他数据的链(link)

2.resource role(资源角色),更可靠和正式的数据库用户可以授予resource role.提供给用户另外的权限以创建他们自己的表、序列、过程(procedure)、触发器(trigger)、索引(index)和簇(cluster)

3.dba role(管理员角色),dba role拥有所有的系统权限,包括无限制的空间限额和给其他用户授予各种权限的能力,system用户由dba角色拥有

查询某一角色的具体权限，(角色名需大写)
SQL> select * from dba_sys_privs where grantee='CONNECT';
```
用户还可以创建自己的 role ,但是用户必须具有 create role 系统权限
```
1.创建角色
SQL> create role c##gjsy_role;

2.授权角色( gjsy_role 角色的所有用户都具有对class表的select查询权限)
SQL> grant select on class to c##gjsy_role;

3.删除角色(与 gjsy_role 角色相关的权限将从数据库全部删除)
drop role c##gjsy_role;

查询当前用户的详细权限
SQL> select * from session_privs;

查询当前用户拥有哪些角色
SQL> select * from user_role_privs;

查询数据库所有用户拥有哪些角色
SQL> select * from dba_role_privs;
```
ORACLE 12C版本中 cdb 和 pdb 关系图

![image](https://github.com/dwjlw1314/DWJ-PROJECT/raw/master/PictureSource/3.22.1.jpg)

CDB与PDB下用户与权限的区别
```
1.在CDB中创建用户必须要以c##开头，称为common用户(公用用户)，在CDB中不允许创建本地用户，在PDB中不允许创建公用用户，
在CDB中创建公用用户默认情况下是在所有PDB下创建了相同的用户，即加了参数container=all;

2.在授权时不加container=all只在当前会话下有相应权限

3.设置所有PDB默认表空间，前提是该表空间存在于每个PDB中

4.可以将公共角色授予本地用户，也可以将本地角色授予公共用户
```
显示12c DB服务上容器的详情(解决plsql如何登录相应容器)
>SQL> select NAME,CON_ID,PDB from v$services;

```
  NAME       CON_ID    PDB
orclpdb.com     3     ORCLPDB
修改tnsnames.ora文件中SERVICE_NAME = orclpdb.com保存即可
```

显示12c DB其他容器状态
>SQL> show pdbs;

查看当前使用容器
>SQL> select sys_context('USERENV','CON_NAME') FROM dual;

查看默认的临时表空间名称
>SQL> select * from database_properties where property_name='DEFAULT_TEMP_TABLESPACE';

查看数据表空间和临时表空间名称
>SQL> select tablespace_name from dba_tablespaces;

查看数据表和临时表空间的使用情况
>SQL> select tablespace_name,bytes/1024 from dba_free_space;   <br>
>SQL> select tablespace_name,free_space/1024 from dba_temp_free_space;

增加数据表空间和临时表空间原有数据文件大小
>SQL> alter database datafile FILE_ID resize 31G;  #FILE_ID用实际值替换<br>
>SQL> alter database datafile '/opt/oracle/oradata/orcl/gjsy_data.dbf' resize 100M;  <br>
>SQL> alter database tempfile '/opt/oracle/oradata/orcl/gjsy_temp.dbf' resize 100M;

给数据表空间和临时表空间增加和删除数据文件
>SQL> alter tablespace GJSY_DATA add datafile 'add.dbf' size 30M autoextend on next 8M maxsize 32G; <br>
>SQL> alter tablespace GJSY_DATA drop datafile '/opt/oracle/oradata/orcl/add.dbf';    <br>
>SQL> alter tablespace GJSY_TEMP add tempfile '/opt/oracle/oradata/orcl/adt.dbf' size 30M;  <br>
>SQL> alter tablespace GJSY_TEMP drop tempfile '/opt/oracle/oradata/orcl/adt.dbf';

临时表空间文件脱机联机
>SQL> alter database tempfile /opt/oracle/oradata/orcl/gjsy_temp.dbf' offline/online;

默认临时表空间并不能脱机,否则会报错
>SQL> alter tablespace GJSY_TEMP offline;  <br>
#ERROR at line 1: ORA-03217: invalid option for alter of TEMPORARY TABLESPACE

临时表空间移动重命名文件，分三步：1.脱机 2.rename 3.联机
>SQL> alter database rename file '/orcl/temp.dbf' to '/orcl/dwj.dbf';

更改用户的默认临时表空间
>SQL> alter user c##antman temporary tablespace tempdefault;

设置 tempdefault 为数据库默认临时表空间
>SQL> alter database default temporary tablespace tempdefault;

收缩临时表空间
>SQL> alter tablespace GJSY_TEMP shrink space keep 20G;  <br>
>SQL> alter tablespace GJSY_TEMP shrink tempfile '/opt/oracle/oradata/orcl/adt.dbf';

删除临时表空间 GJSY_TEMP 不能删除当前用户的默认临时表空间，否则会报ORA-12906
>SQL> drop tablespace GJSY_TEMP including contents and datafiles cascade constraints;

删除数据表空间 gjsy_data 及其包含的数据对象以及数据文件
>SQL> alter tablespace GJSY_DATA offline;  （可选,临时表空间不用该操作） <br>
>SQL> drop tablespace GJSY_DATA including contents and datafiles cascade constraints;

删除用户(指定关键字 cascade ,可删除用户拥有的所有对象)
>SQL> drop user c##antman cascade;

建立查询删除 directory 文件，并授读写权限
```sql
SQL> create directory dwj as '/root/dwj';
SQL> grant read,write on directory dwj to system;
SQL> select * from dba_directories where directory_name = 'DWJ';
SQL> drop directory dwj;
```

<font color=#FF0000 size=5> <p align="center">UNDO表空间</p></font>

```sql
检查undo的管理方式，有两种管理方式，通过参数undo_management来设置auto和manual
SQL> show parameter undo;

检查UNDO Segment状态
SQL> select usn,xacts,status,rssize/1048576,hwmsize/1048576,shrinks from v$rollstat order by rssize;

查看当前用户回滚段信息
SQL> select s.username, u.name from v$transaction t,v$rollstat r,v$rollname u,v$session s
where s.taddr=t.addr and t.xidusn=r.usn and r.usn=u.usn order by s.username;

查看所有用户回滚段信息
SQL> select segment_name,owner,tablespace_name,status from dba_rollback_segs;

创建UNDO表空间
SQL> create undo tablespace UNDOTBS5 datafile '+RACDAT/santdb/datafile/undotbs.dwj'
size 20G reuse autoextend on next 128m maxsize 30G;

切换UNDO表空间为新的UNDO表空间，动态更改spfile配置文件；
SQL> alter system set undo_tablespace=UNDOTBS5 scope=both;

undo表空间在线离线
SQL> alter tablespace UNDOTBS5 online/offline;

删除undo表空间
SQL> drop tablespace UNDOTBS5 including contents and datafiles;

查看Undo表空间的数据文件
SQL> select * from dba_data_files a where a.TABLESPACE_NAME like '%UNDOTBS%'

增加和删除undo表空间文件
SQL> alter tablespace UNDOTBS5 add datafile '+RACDAT/datafile/undotbs5.dwj' size 31G autoextend on;
SQL> alter tablespace UNDOTBS5 drop datafile '+RACDAT/datafile/undotbs5.dwj';

强制undo保存时间
SQL> alter tablespace undotbs5 retention guarantee;

关闭UNDO的自动调整功能，解决UNDO表空间会在很长时间都一直保持着使用率是接近100%的问题
SQL> alter system set "_undo_autotune"= false;

设置event让SMON不自动OFFLINE回滚段
SQL> alter system set events '10511 trace name context forever, level 1';

修复undo表空间转载
https://blog.csdn.net/tianlesoftware/article/details/6261475
```

<font color=#FF0000 size=5> <p align="center">临时表空间组</p></font>

```
临进表空间组是ORACLE 10g引入的一个新特性，它是一个逻辑概念，不需要显示的创建和删除。只要把一个临时表空间分配到一个组中，临时表空间组就自动创建，所有的临时表空间从临时表空间组中移除就自动删除

一个临时表空间组必须由至少一个临时表空间组成，并且无明确的最大数量限制

如果删除一个临时表空间组的所有成员，该组也自动被删除

临时表空间的名字不能与临时表空间组的名字相同

可以在创建临时表空间是指定表空间组，即隐式创建
SQL> create temporary tablespace GJSY_TEMP2 tempfile '/opt/gjsy_temp2.dbf' size 20G tablespace group grp_temp;

查看临时表空间组
SQL> select * from dba_tablespace_groups;

可以指定已经创建好的临时表空间的临时表空间组
SQL> alter tablespace GJSY_TEMP tablespace group grp_temp;

把临时表空间从临时表空间组中移除
SQL> alter tablespace GJSY_TEMP tablespace group '';

给数据库指定临时表空间或为用户指定临时表空间时，可以使用临时表空间组的名称
>SQL> alter user c##antman temporary tablespace grp_temp;
```

<font color=#FF0000 size=5> <p align="center">创建数据库表</p></font>

查看当前数据库登录用户
>SQL> show user

```sql
create table VEHICLE(
  vehicle_id     NVARCHAR2(20) not null,
  vehicle_type   NUMBER(2) default 1,
  note           NVARCHAR2(100)
);
create table WORKING(
	id             NCHAR(36) not null,
	vehicle_id     NVARCHAR2(20),
	switch_time    DATE
);
--启用和禁用触发器
alter table VEHICLE disable/enable all triggers;
--创建降序表索引，在不读取整个表的情况下，可以使数据库应用程序可以更快地查找数据
create unique index WORKING_INDEX on WORKING(vehicle_id desc);  '可以使用下面的方式在指定表空间'
create unique index WORKING_INDEX on WORKING(vehicle_id desc) local tablespace gps_tbs nologging;
--命令使索引失效
alter index WORKING_INDEX unusable;
--删除表分区(末尾添加 update indexes 可以在删除分区表的时候直接进行索引的重建)
alter table VEHICLE drop partition SYS_P120862;
--重建索引,两种都可以
alter index WORKING_INDEX rebuild (online);
alter index WORKING_INDEX rebuild;
--重建分区索引
alter index WORKING_INDEX rebuild partition p1;
--查询当前用户下有哪些分区表
select table_name, partitioning_type,partition_count from user_part_tables;
--查看非分区索引是否有效
select index_name,status from user_indexes/dba_indexes;
--查看分区索引是否有效
select index_name,status from user_ind_partitions/dba_ind_partitions;
--分区索引重建，只需要重建那个失效的分区
alter index WORKING_INDEX rebuild partition partition_name (online);
alter index WORKING_INDEX rebuild partition partition_name;
--添加主键,只能有一个
alter table VEHICLE add constraint VEHICLE_PK primary key(vehicle_id);
--删除指定表分区
alter table VEHICLE drop partition SYS_P122602;
--删除主键 (如果数据表中有其他外键关联，则无法删除)
alter table VEHICLE drop constraint VEHICLE_PK;
--添加外键
alter table WORKING add constraint WORKING_FK foreign key(vehicle_id) references vehicle(vehicle_id);
--关闭和启动外键约束
alter table WORKING disable/enable constraint WORKING_FK;
--删除外键
alter table WORKING drop constraint WORKING_FK;
--添加检查约束 (如果数据表中数据有超出约束范围的添加失败)
alter table VEHICLE add constraint VEHICLE_CHK check (vehicle_type in ('1','2','3'));
--关闭和启动检查约束
alter table VEHICLE disable/enable constraint VEHICLE_CHK;
--删除约束，只需约束名称即可
alter table VEHICLE drop constraint VEHICLE_CHK;
--约束唯一标识数据库表中的每条记录,可以有多个约束
alter table VEHICLE add constraint DETAIL_UNIQUE unique (note);
--删除 unique 约束
alter table VEHICLE drop constraint DETAIL_UNIQUE;
--添加列的默认值
alter table VEHICLE modify note default 'ecuador';
--删除列的默认值
alter table VEHICLE modify note default NULL;
--杀死阻塞了其他连接的 session; eg: alter system kill session '62,35686';
alter system kill session 'SID,SERIAL#';
--删除asm磁盘组文件
alter diskgroup OCR drop file '+OCR/PRO_BUSI/DATAFILE/xxx.dbf';
--删除数据表 (无法使用flashback恢复),如果其他表的外键引用该表的主键,不用cascade关键字删除会报错
drop table VEHICLE purge;
--如果其他表的外键引用该表的主键，不用cascade关键字删除表就会报错；使用flashback可以恢复，但外键无法恢复
drop table VEHICLE cascade constraint;
--删除索引
drop index WORKING_INDEX;
--删除数据表中的数据，但并不删除表本身
truncate table VEHICLE;
--从回收站删除类型为 table 的 ant_pts
purge table ant_pts;
--从回收站删除类型为 index 的 ant_pts
purge index ant_pts;
--清空当前用户的回收站
purge recyclebin;
--清空所有用户的回收站
purge dba_recyclebin;
```

drop和truncate及delete的区别，truncate不能用于参与了索引视图的表和有foreign key引用的表

drop table | truncate table | delete from
---|---|---
属于DDL(自动提交) | 属于DDL(自动提交) | 属于DML
不可回滚 | 不可回滚 | 可回滚
不可带where | 不可带where | 可带where
表内容和表结构删除 | 表内容删除 | 表结构在，表内容要看where的执行情况
删除速度最快 | 删除速度块 | 删除速度最慢，要逐行删除

<font color=#FF0000 size=5> <p align="center">数据库表操作语句</p></font>

设置"SQL>"终端行宽
>SQL> set linesize 120

查看sql语句执行返回状态值
>SQL> show sqlcode

```sql
alter table c##antman.vehicle rename to newname;                 #修改表名
alter table c##antman.vehicle rename column oldname to newname;  #修改列名
alter table c##antman.vehicle add owner NVARCHAR2(20);           #添加表列
alter table c##antman.vehicle drop column owner;                 #删除表列
alter table VEHICLE modify note NVARCHAR2(500) [default value];  #修改字段类型,默认值参数可选

--创建或替换 view 视图中的 vehicle_view 视图数据
create or replace view vehicle_view as select * from VEHICLE;
--删除 vehicle_view 视图
drop view vehicle_view;
--主外键生效/失效
alter table VEHICLE disable/enable constraint VEHICLE_PK;
--触发器生效/失效
alter table WORKING disable/enable all triggers;
--插入数据 (sysdate获取当前时间，最后事务提交 commit;)
insert into WORKING values('F08C1DDC-E828-11E4-89C7-9D9859F01C00','MAT0532',sysdate);
--插入数据 (指定列)
insert into WORKING (id,vehicle_id) values('uuid','MAT0533');
--把VEHICLE表数据插入到WORKING中
insert into WORKING select * from VEHICLE;
--根据scn号插入指定数据
insert into WORKING select * from WORKING as of scn 1730685;
--更新数据 (最后事务提交 commit;)
update VEHICLE set note = 'text' where vehicle_id = 'MAA3006';
--建立一个快表将working表数据复制一份
create table quici_table as select * from working;
--Create database link
create database link LAB_DB_LINK connect to antman identified by ant
using '(DESCRIPTION =
  (ADDRESS_LIST =
    (ADDRESS = (PROTOCOL = TCP)(HOST = 172.26.130.4)(PORT = 1521))
  )
  (CONNECT_DATA =
    (SERVICE_NAME = antlab)
  )
)';
--查询表的DDL信息
select dbms_metadata.get_ddl('TABLE','VEHICLE') from dual;
--查询表空间的DDL信息
select dbms_metadata.get_ddl('TABLESPACE','tablespace name') from dual;
--查看当前的SCN号
select dbms_flashback.get_system_change_number fscn from dual;
--通过时间来查看历史的scn号码
select name,FIRST_CHANGE#  fscn,NEXT_CHANGE# nscn,FIRST_TIME from v$archived_log;
--通过database link查询数据
select count(*) from VEHICLE@LAB_DB_LINK;
--查询数据
select avg(vehicle_type) as avg,sum(vehicle_type) as sum from VEHICLE where note is not null;
select * from WORKING where switch_time >= to_date('2018-09-28 00:02:00','yyyy-mm-dd hh24:mi:ss');
select * from WORKING where switch_time > sysdate-1/24;    #一小时以前
--单个值进行替换
select replace(v.vehicle_type,1,'text') from VEHICLE;
--多个值值进行替换方式一
select decode(v.vehicle_type,1,'text',2,'bus',3,'cbus') from VEHICLE;
--多个值进行替换方式二
select
sum(case v.vehicle_type when 1 then 1 else 0 end) text,
sum(case v.vehicle_type when 2 then 1 else 0 end) bus,
sum(case when v.vehicle_type <>1 then 1 else 0 end) none
from VEHICLE v;
--单个值进行替换方式二
select case v.vehicle_type when 1 then 'text' else 'other' end from VEHICLE v;
--查询现在往前两年内的数据
select * from WORKING where months_between(sysdate,switch_time)/12<2;
--以首字母大写的方式显示 vehicle_id 列数据
select initcap(vehicle_id) as vehicle_name from WORKING;
--查询语句使用不等于有3种 ！= or <> or ^=
select * from VEHICLE where vehicle_type ^= 3;
--模糊查询 vehicle_id 列 至少含一个字符的数据
select * from VEHICLE where vehicle_id like '%_%';
--查询 VEHICLE 表，优先按照 vehicle_type 降序，再按照 vehicle_id 升序 进行显示
--注意：升序空值在结果的末尾，降序空值在结果的最前面
select vehicle_id,vehicle_type from VEHICLE order by vehicle_type desc,vehicle_id;
--按照日期的年份进行排序显示,还可以使用month，day进行显示；extract 是提取函数
select extract(year from switch_time) as year from WORKING order by year;
--查询 VEHICLE 表中的数据，同时 vehicle_id 包含在 WORKING 表中且等于 'MAT0532'
select * from VEHICLE where vehicle_id in (select vehicle_id from WORKING) and vehicle_id = 'MAT0532';
--按照不同数据组进行分组显示,同时只显示 count(vehicle_type) > 2 的数据;
select vehicle_type,count(vehicle_type) from VEHICLE group by vehicle_type having count(vehicle_type) > 2;
--删除所有数据，只留表结构
delete * from WORKING;
--查询表分区数据
select * from WORKING partition(SYS_P104272);
--删除指定的表分区数据
delete from WORKING  partition(SYS_P104272) where rownum < 1000000;
--按照拼接字符串的方式显示查询数据
select 'vehicle=' || vehicle_id || 'vehicle_type=' || vehicle_type from VEHICLE;
--删除数据，指定行号不包含在查找出来的数据被删除
--空值会对not in造成影响，也就是不等于任何值
delete from WORKING t where rowid not in (select min(rowid) from WORKING group by vehicle_id)
--查找时间区间的数据, between 之前可以加 not 表示不在指定的区间
select * from WORKING switch_time between to_date('2018-08','yyyy-mm') and to_date('2018-09','yyyy-mm');
--会让 MAT0532 对应记录处于锁定状态，其他用户无法修改，修改后必须跟随 commit;
select * from VEHICLE where vehicle_id = 'MAT0532' for update;
--获取指定日期的下一个日期时间
select next_day(sysdate,'Saturday') from dual;
--查询给定日期范围的月数
select months_between(sysdate,switch_time) from WORKING;
--查看用户下所有包含外键的表
select * from USER_CONSTRAINTS t where t.constraint_name like '%_FK';
--指定日期加上指定月数，求出之后的日期
select sysdate,add_months(sysdate,1) from dual;
--获取当前月份的第一天，扩展 'year','day','HH24'...
select trunc(sysdate, 'month') as "first day" from dual;
--获取当前月份的最后一天,trunc函数不显示时间
select trunc(last_day(sysdate)) "last day" from dual;
--获取当前月份的天数
select cast(to_char(last_day(sysdate), 'dd') as int) month_days from dual;
--获取当前月份剩下的天数
select sysdate,last_day(sysdate), last_day(sysdate)-sysdate from dual;
--获取两个日期之间的天数
select round((months_between('01-feb-2014', '01-mar-2012') * 30), 0) days from dual;
select trunc(sysdate) - trunc(w.switch_time) from WORKING w;
--获取直到目前为止今天过去的秒数(从 00：00 开始算)
select (sysdate-trunc(sysdate)) * 24 * 60 * 60 sec_since_morning from dual;
--在 oracle 中生成随机数值
select round(dbms_random.value() * 100) as random_num from dual;
--检查表中是否含有任何的数据
select 1 from install where rownum = 1;
--把数值转换成字符串
select to_char(to_date(1526, 'j'), 'jsp') from dual;
--日期时间间隔操作;当前时间减去7分钟的时间
select sysdate,sysdate - interval '7' MINUTE from dual;
--当前时间减去7小时的时间
select sysdate - interval '7' hour from dual;
--当前时间减去7天的时间
select sysdate - interval '7' day from dual;
--当前时间减去7月的时间
select sysdate,sysdate - interval '7' month from dual;
--当前时间减去7年的时间
select sysdate,sysdate - interval '7' year from dual;
--时间间隔乘以一个数字
select sysdate,sysdate - 8*interval '2' hour from dual;
--日期到字符串转换操作;ddd是365天的第几天，iw-d 是一年中的第几周和一周的第几天
select sysdate,to_char(sysdate,'yyyy-mm-dd hh24:mi:ss') from dual;
select sysdate,to_char(sysdate,'yyyy-mm-dd hh:mi:ss') from dual;
select sysdate,to_char(sysdate,'yyyy-ddd hh:mi:ss') from dual;
select sysdate,to_char(sysdate,'yyyy-mm iw-d hh:mi:ss') from dual;
--字符串到日期转换操作,格式对应即可
select to_date('2003-10-17 21:15:37','yyyy-mm-dd hh24:mi:ss') from dual;
--返回当前时间的秒毫秒，可以指定秒后面的精度(最大=9)
select to_char(current_timestamp(9),'DD-MM-YYYY HH24:MI:ssxff') from dual;
```

高级SQL用法
```sql
--union操作符用于合并多个 select 的结果集.必须拥有相同的列、数据类型、并且语句中列的顺序也要相同
--默认地，union 操作符只会选取不同的值。如果允许重复的值，请使用 union all
--另外，union 结果集中的列名总是等于 union 中第一个 select 语句中的列名
select t.id as first from WORKING t union all select d.id as second from detail d;
--intersect操作符只返回两个查询语句都包含的值
select * from VEHICLE where vehicle_type = 1 intersect select * from VEHICLE where vehicle_type = 1;
--minus操作符只返回第一个查询结果数据但是不在第二个查询结果中的数据
select * from VEHICLE where vehicle_type = 1 minus select * from VEHICLE where vehicle_status = 1;
--auto-increment 会在新记录插入表中时生成一个唯一的数字，必须通过 sequence 对创建该字段
--以 1 起始且以 1 递增。该对象缓存 10 个值以提高性能
create sequence sqe_vehicle
start with 1
increment by 1
cache 10
--插入新记录，必须使用 nextval 函数
insert into VEHICLE values('MAA5011',sqe_vehicle.nextval,'gjsy');
--删除 auto-increment
drop sequence sqe_vehicle;
--给对象名称创建公共同义词(Create the synonym),可以用同义词名称进行查询
create or replace public synonym car for C##ANTMAN.VEHICLE;
--删除对象名称同义词
drop public synonym car;
-- Create database link
create database link DWJ connect to c##antman identified by ant
  using '(DESCRIPTION = (ADDRESS = (PROTOCOL = TCP)(HOST = 192.168.0.1)(PORT = 1521))
  (CONNECT_DATA = (SERVER = DEDICATED)(SERVICE_NAME = orcl.com)))';
-- Drop existing database link
drop database link DWJ;
--oracle 可以使用 NVL() 函数获取字段为空的取值定义
select nvl(note,0) from VEHICLE;
select vehicle_id,nvl(vehicle_type,0) + substr(vehicle_id,4,4) from VEHICLE;
--oracle 计数和计算的内建函数基本类型 1.Aggregate 合计函数 2.Scalar 函数
1. SUM(column_name)                         #返回某列的总和
1. AVG(column_name)	                        #返回某列的平均值
1. COUNT(column_name)	                      #返回某列的行数(不包括NULL值)
1. COUNT(*)	                                #返回表总行数
1. COUNT(distinct column_name)	            #返回相异结果的数目
1. MAX(column_name) / MIN(column_name)	    #返回某列的最高值 / 最低值
2. ROUND(c,decimals)	                      #对某个数值域进行指定小数位数的四舍五入
2. MOD(x,y)	                                #返回除法操作的余数
2. LENGTH(column_name)	                    #返回某个文本域的长度
2. MID(column_name,start[,length])          #从文本字段中提取字符 eg:MID(id,1,3)
--获取表的DDL语句
select dbms_metadata.get_ddl('TABLE',u.table_name) from user_tables u;
--获取表空间的DDL语句
select dbms_metadata.get_ddl('TABLESPACE',u.TABLESPACE_NAME) from user_tables u;
--获取32位 GUID值(该数据类型实际是16位)，然后从 位置3 提取2个值
select dbms_lob.substr(to_blob(sys_guid()),3,2) from dual;
--提取vehicle_id字段部分数据转换成数字
select to_number(substr(vehicle_id,4,3)) from VEHICLE;
--大小写转换
select UPPER/LOWER(vehicle_id) from VEHICLE;
--对字符串进行拼接串联
select concat(vehicle_id,vehicle_type) from VEHICLE;
--多表自然连接,类似 inner join
select count(*) from VEHICLE t natural join WORKING;
select count(*) from VEHICLE t join WORKING using(vehicle_id);
```

在 Oracle 生成随机数据
```sql
select level empl_id,
    mod (rownum, 50000) dept_id,
    trunc (dbms_random.value (1000, 500000), 2) salary,
    decode (round (dbms_random.value (1, 2)),  1, 'm',  2, 'f') gender,
    to_date (
      round (dbms_random.value (1, 28))
      || '-'
      || round (dbms_random.value (1, 12))
      || '-'
      || round (dbms_random.value (1900, 2010)),
      'dd-mm-yyyy')
      dob,
    dbms_random.string ('x', dbms_random.value (20, 50)) address
    from dual
    connect by level < 10000;
```
查询 user_objects 视图中的数据来检查视图的状态，其中对象名称必须是大写的
>SQL> select object_name,status from user_objects where object_name='VEHICLE_VIEW';

查询 WORKING 表创建时间，其中对象名称必须是大写的
>SQL> select created from dba_objects where object_name = 'WORKING';

可以设置 SQL_TRACE 参数跟踪查看 SQL 语句执行具体过程
>SQL> alter session set sql_trace=true;

数据库数据去重有以下那么三种方法
```
1.(*)表示所有记录的每一个字段值完全相同，用关键字 distinct 就可以去掉(关键字会触发排序操作)
select distinct * from VEHICLE;
2.两条记录之间之后只有部分字段的值是有重复的，但是表存在主键或者唯一性ID，这可以使用 group by 分组去重
select vehicle_type from VEHICLE group by vehicle_type;
3.还可以使用临时表，将数据复制到临时表，查找完成重复数据之后再删除临时表
create table TMP(vehicle_id,note) as select vehicle_id,note from VEHICLE
```

数据库连接可分为，内连接(inner join)、外连接(outer join)、全连接(full join)
```sql
1.内连接 <inner join>，这是经常用的查询方式。下面两条语句效果等同
select * from WORKING w inner join vehicle v on v.vehicle_id = w.vehicle_id;
select * from WORKING w,VEHICLE v where w.VEHICLE_ID = v.vehicle_id;

2.外连接 <outer join>，可分为左外连接 <left outer join> 和右外连接 <right outer join>
左外连接就是以 <left join> 前面的表为主表，即使有些记录关联不上，主表的信息能够查询出来的
select * from VEHICLE v left outer join WORKING w on v.vehicle_id = w.vehicle_id;
右外连接就是以 <right join> 后面的表为主表，即使有些记录关联不上，主表的信息能够查询出来
select * from VEHICLE v right outer join WORKING w on v.vehicle_id = w.vehicle_id;
实现外连接的简便用法，效果同上
select * from VEHICLE v, WORKING w where v.vehicle_id(+) = w.vehicle_id;  #右外连接
select * from VEHICLE v, WORKING w where v.vehicle_id = w.vehicle_id(+);  #左外连接

3.全连接的查询结果是左外连接和右外连接查询结果的并集
select * from VEHICLE v full join WORKING w on 1=1;
```

#从pfile文件启动数据库实例 (默认是spfile文件启动)
```
SQL> startup pfile='/opt/oracle/product/OraHome/dbs/initorcl.ora';
startup 启动查找顺序是 spfileSID.ora->spfile.ora->initSID.ora->init.ora(spfile优先于pfile)
pfile和spfile可以互相创建，同时可以指定参数文件的位置
SQL> create spfile[='/opt'] from pfile[='/dbs'];
SQL> create pfile[='/dbs'] from spfile[='/opt'];
```

数据库表数据导出与还原有三种主要的模式(完全、用户、表)
```
1.完全模式,将orcl指定连接的数据库完全导出，system用户必须具有导出导入的权限
[oracle@dwj ~]$ exp system/system@orcl file=~/backup_$(date +%Y-%m-%d).dmp log=~/backup.log full=y
[oracle@dwj ~]$ imp system/system@orcl file=~/backup_$(date +%Y-%m-%d).dmp log=~/backup.log full=y

2.用户模式，fromuser(导入文件中的用户)、touser 参数，必须指定这样才能导入数据;[owner]可选参数
[oracle@dwj ~]$ exp system/system@orcl file=~/backup.dmp log=~/backup.log [owner=system,c##antman]
[oracle@dwj ~]$ imp system/system@orcl file=~/backup.dmp log=~/backup.log fromuser=system touser=system

3.表模式,用户 system 的表 help,work 被导出; owner 导出整个用户或者多个用户,而 tables 表示只导出其中的表
[oracle@dwj ~]$ exp system/system@orcl file=~/backup.dmp log=~/backup.log tables=help,work
[oracle@dwj ~]$ imp system/system@orcl file=~/backup.dmp log=~/backup.log tables=help,work

4.导出表中部分数据，如果表存在，则不会导入数据，参数 [ignore=y] 可以确保数据导入
[oracle@dwj ~]$ exp c##antman/ant@orcl file=~/backup_table.dmp tables=vehicle query="'where vehicle_type<3'"
[oracle@dwj ~]$ imp c##antman/ant@orcl file=~/backup_table.dmp tables=vehicle [ignore=y]

EXP参数里面可能发生冲突的组合：
1.同时指定了 owner 和 tables
2.同时指定了 FULL 和 tables
3.同时指定了多个 owner 和 full
```

expdp --> 数据库表数据导出与还原另外一种导出方式(导入和导出需要相同的表空间和用户名)
```
1.创建dmp文件存储路径
[oracle@dwj ~]$ mkdir ~/dump_dir

2.sqlplus中为dump导入导出新建目录名称(dump_dir)
SQL> create directory dump_dir as '~/dump_dir';

3.设置用户的导入导出目录赋读写权限,dump_dir为上条语句创建的目录名称,antman为数据库的用户名
SQL> grant read,write on directory dump_dir to antman;

4.退出 sqlplus 并运行dump工具导出数据, reuse_dumpfiles=true表示可以覆盖同名文件
[oracle@dwj ~]$ expdp antman/ant directory=dump_dir dumpfile=dump.dmp schemas=antman

5.数据导入命令 [table_exists_action: valid keywords: (skip), append, replace and truncate]
[oracle@dwj ~]$ impdp antman/ant directory=dump_dir dumpfile=dump.dmp [table_exists_action=append] full=y
```

<font color=#FF0000 size=5> <p align="center">RMAN 备份及恢复步骤</p></font>

```
可以用来备份和还原数据库文件、归档日志和控制文件。它也可以用来执行完全或不完全的数据库恢复
RMAN不能用于备份初始化参数文件（备份控制文件时一齐备份）和口令文件
RMAN启动数据库上的Oracle服务器进程来进行备份或还原。备份、还原、恢复是由这些进程驱动的
RMAN可以由OEM的Backup Manager GUI来控制
```

1、切换服务器归档模式，如果已经是归档模式可跳过此步
```
[oracle@diwj ~]$ sqlplus / as sysdba     #以DBA身份连接数据库
SQL> shutdown immediate;                 #立即关闭数据库
SQL> startup mount                       #启动实例并加载数据库，但不打开
SQL> alter database archivelog;          #更改数据库为归档模式
SQL> alter database open;                #打开数据库
SQL> alter system archive log start;     #启用自动归档
SQL> exit                                #退出
```
2、连接启动恢复管理器,其中sys/sys是dba账号
```
[oracle@dwj ~]$ rman target /

其他连接方式：
[oracle@dwj ~]$ rman target=sys/sys@orcl;   #关闭数据库后,同时链接失效,无法再次登录
[oracle@dwj ~]$ rman target '"/ as sysdba"'
[oracle@dwj ~]$ rman target '"/ as sysbackup"'
```
3、查看所有设置
>RMAN> show all;

4、基本设置
```
设置默认的备份设备为磁盘
RMAN> configure default device type to disk;
设置备份的并行级别，通道数
RMAN> configure device type disk parallelism 2;
打开控制文件与服务器参数文件的自动备份
RMAN> configure controlfile autobackup on;
设置备份的文件格式，只适用于磁盘设备
RMAN> configure channel 1 device type disk format 'backup1_%U';
RMAN> configure channel 2 device type disk format 'backup2_%U';
设置控制文件与服务器参数文件自动备份的文件格式
RMAN> configure controlfile autobackup format for device type disk to 'ctl_%F';
```
5、查看数据库物理结构-报表模式(后面命令会使用相关字段数据，eg：File and Tablespace)
>RMAN> report schema;

6、开始备份数据库(根据需求选择其中一种备份方案)
```
备份全库及控制文件、服务器参数文件与所有归档的重做日志，并删除旧的归档日志
RMAN> backup database plus archivelog delete input;
备份指定表空间及归档的重做日志，并删除旧的归档日志
RMAN> backup tablespace GJSY_DATA plus archivelog delete input;
备份归档日志
RMAN> backup archivelog all delete input;
复制数据文件
RMAN> copy datafile 1 to '/opt/oracle/product/OraHome/dbs/system.copy';
```
7、查看备份和文件复本(依据ORACLE_HOME/dbs/backup*,ctl*)
>RMAN> list backup by file;

8、验证备份(依据上一步命令中的结果内容"in backup set #")
>RMAN> validate backupset 3;

9、从自动备份中恢复服务器参数文件
```
RMAN> shutdown normal;                                          #关闭数据库
RMAN> startup mount;                                            #启动实例
方式一：
RMAN> restore spfile to pfile 'initorcl.ora' from autobackup;   #从自动备份中恢复服务器参数文件
SQL> create spfile='spfile.ora' from pfile='initorcl.ora';      #使用生成的pfile文件创建spfile文件
方式二：
RMAN> restore spfile to 'spfile.ora' from autobackup;
方式三：
RMAN> restore spfile to 'spfile.ora' from '/path/back_file';
```
10、从自动备份中恢复控制文件
```
RMAN> shutdown normal;                                   #关闭数据库
RMAN> startup mount;                                     #启动实例
RMAN> restore controlfile to 'ctrl.f' from autobackup;   #从自动备份中恢复控制文件
```
11、恢复和还原整个数据库
```
RMAN> shutdown normal;                                #关闭数据库
[oracle@dwj ~]$ mv gjsy_data.dbf gjsy_data.dbf.bak    #将最新的数据表空间文件备份
[oracle@dwj ~]$ mv sysaux01.dbf sysaux01.dbf.bak
[oracle@dwj ~]$ mv system01.dbf system01.dbf_bak
[oracle@dwj ~]$ mv undotbs01.dbf undotbs01.dbf_bak
[oracle@dwj ~]$ mv users01.dbf users01.dbf_bak

[oracle@dwj ~]$ rman target /       #进入恢复管理器模式
RMAN> startup mount;                #启动实例
RMAN> restore database;             #还原数据库
RMAN> recover database;             #恢复数据库
RMAN> alter database open;          #打开数据库
```
12、恢复和复原表空间
```
RMAN> startup;                                             #启动实例
RMAN> sql 'alter tablespace GJSY_DATA offline immediate';  #将表空间脱机
[oracle@dwj ~]$ mv gjsy_data.dbf gjsy_data.dbf_bak         #将表空间重命名

[oracle@dwj ~]$ rman target /                              #进入恢复管理器模式
RMAN> restore tablespace GJSY_DATA;                        #还原表空间
RMAN> recover tablespace GJSY_DATA;                        #恢复表空间
RMAN> sql 'alter tablespace GJSY_DATA online';             #将表空间联机
```
REPORT 显示基础报告
```
RMAN> report need backup days=3;      #报告最近3天没有被备份的数据文件
RMAN> report unrecoverable;           #报告数据库所有不可恢复的数据文件
RMAN> report obsolete;                #报告多余的备份
```
CROSSCHECK 校验备份信息
```
RMAN> crosscheck backup of controlfile;    #核对控制文件的备份集
RMAN> crosscheck backup of spfile;         #核对SPFILE的备份集
RMAN> crosscheck backup;                   #核对所有备份集
RMAN> crosscheck copy;                     #核对所有映像副本
```
RMAN 删除备份信息(某些情况下并非删除记录，而是打上删除标记)
```
#删除所有以前过期的备份(包括数据文件、归档、控制文件)
RMAN> crosscheck backupset;    #或 crosscheck backup;
RMAN> report obsolete;
RMAN> delete obsolete;

RMAN> delete expired backup;         #删除expired备份
RMAN> delete expired copy;           #删除expired副本
RMAN> delete backupset 19;           #删除特定备份集
RMAN> delete copy;                   #删除所有映像副本
RMAN> delete backup;                 #删除所有备份集
RMAN> delete expired archivelog all; #删除过期的归档日志
```
删除 7 天以前的归档日志
>RMAN> delete force archivelog all completed before 'sysdate-7';

<font color=#FF0000 size=5> <p align="center">恢复误删除的表、表数据</p></font>

1.查询表 user_recyclebin 最近操作过的表名称，然后用闪回(只能用于10G及以上版本)
```
SQL> select * from user_recyclebin;             #查找闪回中的记录
SQL> flashback table WORKING to before drop;    #WORKING是误删除表名
```
2.表数据误删除恢复，直接覆盖原有表
```
#打开Flash存储的权限
SQL> alter table WORKING enable row movement;
#把表还原到指定时间点
SQL> flashback table WORKING to timestamp to_timestamp('2018-10-03 15:40:00','yyyy-mm-dd hh24:mi:ss');
```
3.表数据误删除恢复，查询丢失数据，手动插入
>SQL> select * from WORKING as of timestamp to_timestamp('2018-03-16 11:40:00','YYYY-MM-DD HH24:MI:SS');
