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
//以 sys 用户登陆的话 必须要加上 as sysdba 子句
[oracle@dwj ~]$ sqlplus / as sysdba          #以操作系统权限认证的sys管理员登陆
[oracle@dwj ~]$ sqlplus / as sysbackup       #以操作系统权限认证的sysbackup管理员登陆
[oracle@dwj ~]$ sqlplus /nolog               #不在terminal当中暴露密码的登陆方式
SQL> conn / as sysdba   
SQL> conn sys/password as sysdba
[oracle@dwj ~]$ sqlplus scott/tiger          #非管理员用户登陆
[oracle@dwj ~]$ sqlplus                      #不显露密码的登陆方式
Enter user-name: system
Enter password:
[oracle@dwj ~]$ sqlplus scott/tiger@orcl     #非管理员用户使用tns别名登陆
[oracle@dwj ~]$ sqlplus system/pwd@orcl      #管理员用户使用tns别名登陆
通过网络连接，这是需要数据库服务器的 listener 处于监听状态。此时建立一个连接的大致步骤如下:
a. 查询sqlnet.ora，看看名称的解析方式，默认是TNSNAME
b. 查询 tnsnames.ora 文件，从里边找 orcl 的记录，并且找到数据库服务器的主机名或者IP，端口和service_name
c. 如果服务器 listener 进程没有问题的话，建立与 listener 进程的连接
d. 根据不同的服务器模式如专用服务器模式或者共享服务器模式，listener采取连接动作。默认是专用服务器模式
```
检查系统当前视图相关参数(v$parameter视图中查询参数的时候其实都是通过x$ksppi和x$ksppcv这两个内部视图中得到的)
```sql
SQL> select count(*) from v$instance;              #所有参数字段
SQL> select count(*) from v$system_parameter;      #所有参数字段
SQL> select count(*) from v$spparameter;           #所有参数字段
SQL> select count(*) from v$parameter;             #所有参数字段
SQL> select status from v$instance;                #查看oracle启动状态
SQL> select * from nls_database_parameters;        #查看数据库服务器字符集
SQL> select name from v$tempfile;                  #查看当前用户临时表空间位置
SQL> select count(*) from v$process;               #查询数据库当前进程的连接数
SQL> select count(*) from v$session;               #查看数据库当前会话的连接数
SQL> select name from v$controlfile;               #查看控制文件
SQL> select member from v$logfile;                 #查看日志文件
SQL> select flashback_on from v$database;          #查看闪回是否开启
SQL> select created,log_mode from v$database;      #查看数据库的创建日期和归档方式
SQL> select SQL_TEXT,SQL_ID,SERVICE from v$sql;    #查看数据库执行语句文本内容
SQL> select MESSAGE from v$session_longops;        #捕捉运行很久的SQL
SQL> select * from v$locked_object;                #查看未提交的事务
SQL> select * from v$transaction;                  #查看未提交的事务
SQL> select * from v$version;                      #查看数据库的版本
```
查看所有环境设置：
> SQL> show all

检查数据库全部或者部分参数(parameter_name可选模糊匹配)
> SQL> show parameter [parameter_name];        #eg:show parameter service_names

查看数据库服务器字符集
> SQL> select userenv('language') from dual;

查询数据库允许的最大连接数(processes)或会话数(sessions)
>SQL> select value from v$parameter where name = 'processes';

查看数据库对象, owner参数需要大写
>SQL> select object_name, object_type, status from all_objects where owner='SYSTEM';

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

查询某用户下的所有表空间详细信息(desc 适用所有视图参数eg:Tablespaces;Views;Sequences;Indexes;...)
```sql
在dba权限下查询 user 用户下的所有表的信息
SQL> select count(*) from dba_tables where owner='user';
SQL> select count(*) from all_tables where owner='user';
表名以及各详细内容等相应字段查询
SQL> desc dba_tables / all_tables

在当前用户权限下查询该用户所有表的信息
SQL> select count(*) from user_tables;
表名以及各详细内容等相应字段查询;已经不包括owner字段
SQL> desc user_tables
```
查询数据库下的所有用户视图详细信息
```sql
在dba权限下查询用户的所有信息
SQL> select count(*) from dba_users;
SQL> select count(*) from all_users;
用户视图所有字段查询
SQL> desc dba_users / all_users

在当前用户权限下查询该用户所有的信息
SQL> select count(*) from user_users;
用户视图所有字段查询
SQL> desc user_users
```
查看和启动数据库的监听端口 1521 ,其中有 Listener Parameter File 路径
>[oracle@dwj Desktop]$ lsnrctl status/start

修改数据库系统参数 <sevice_name> 的值
>SQL> alter system set service_names=newvalue;

修改数据库用户密码，显示成功后执行 commit;
>SQL> alter user sys identified by new-password;

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

查看归档模式
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

第5步：给用户授予权限(以下是不同权限的设置方式，取其一即可，角色描述如下)
SQL> grant select any dictionary to c##antman;
SQL> grant connect,resource,dba to c##antman;

撤销权限(可选部分)
语法：revoke role-type from username;
SQL> revoke connect,resource from c##antman;
```
oracle为兼容以前版本，提供三种标准角色(role):connect/resource和dba
```
1.connect role(连接角色),临时用户,特指不需要建表的用户,通常只是使用oracle简单权限,这种权限只对其他用户的表有访问权限，
包括select/insert/update和delete等,该角色用户还能够创建表、视图、序列(sequence)、簇(cluster)、同义词(synonym)、
会话(session)和其他数据的链(link)

2.resource role(资源角色),更可靠和正式的数据库用户可以授予resource role.提供给用户另外的权限以创建他们自己的表、序列、
过程(procedure)、触发器(trigger)、索引(index)和簇(cluster)

3.dba role(管理员角色),dba role拥有所有的系统权限,包括无限制的空间限额和给其他用户授予各种权限的能力,system由dba用户拥有

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

查询当前用户权限和角色
SQL> select * from session_privs;
SQL> select * from user_role_privs;
```
ORACLE 12C版本中 cdb 和 pdb 关系图

![image](https://github.com/dwjlw1314/DWJ-PROJECT/raw/master/PictureSource/3.22.1.jpg)

ORA-65096: invalid common user or role name错误
```
显示可用的pdb
SQL> select PDB from v$services;

显示当前是 COMMON_USERS 还是 LOCAL_USERS (12C以上版本)
SQL> show con_name

如果是 CDB$ROOT 模式，需要使用如下方式创建用户
SQL> create user c##ant identified by ant;

切换 ORCLPDB 模式,就可以使用旧模式创建用户
SQL> alter session set container=ORCLPDB;

新建用户出现 ORA-01109: database not open错误
SQL> alter pluggable database ORCLPDB open;
```
查看数据表空间和临时表空间存储位置和名称
>SQL> select tablespace_name from dba_tablespaces;   #包括数据和临时表空间  <br>
>SQL> select file_name,tablespace_name,BYTES from dba_data_files;  <br>
>SQL> select file_name,tablespace_name,BYTES from dba_temp_files;

查看数据表和临时表空间的使用情况
>SQL> select tablespace_name,bytes/1024 from dba_free_space;   <br>
>SQL> select tablespace_name,free_space/1024 from dba_temp_free_space;

增加数据表空间和临时表空间原有数据文件大小
>SQL> alter database datafile '/opt/oracle/oradata/orcl/gjsy_data.dbf' resize 100M;  <br>
>SQL> alter database tempfile '/opt/oracle/oradata/orcl/gjsy_temp.dbf' resize 100M;

给数据表空间和临时表空间增加和删除数据文件
>SQL> alter tablespace GJSY_DATA add datafile '/opt/oracle/oradata/orcl/add.dbf' size 30M;  <br>
>SQL> alter tablespace GJSY_DATA drop datafile '/opt/oracle/oradata/orcl/add.dbf';    <br>
>SQL> alter tablespace GJSY_TEMP add tempfile '/opt/oracle/oradata/orcl/adt.dbf' size 30M;  <br>
>SQL> alter tablespace GJSY_TEMP drop tempfile '/opt/oracle/oradata/orcl/adt.dbf';

更改用户的默认临时表空间
>SQL> alter user c##antman temporary tablespace tempdefault;

设置 tempdefault 为数据库默认临时表空间
>SQL> alter database default temporary tablespace tempdefault;

删除表空间 gjsy_tmp 及其包含的数据对象以及数据文件
>SQL> alter tablespace gjsy_tmp offline;  （可选） <br>
>SQL> drop tablespace gjsy_tmp including contents and datafiles;

删除用户(指定关键字 cascade ,可删除用户拥有的所有对象)
>SQL> drop user c##antman cascade;

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
--添加主键
alter table VEHICLE add constraint VEHICLE_PK primary key(vehicle_id);
--删除主键 (如果数据表中有其他外键关联，则无法删除)
alter table VEHICLE drop constraint VEHICLE_PK;
--添加外键
alter table WORKING add constraint WORKING_FK foreign key(vehicle_id) references vehicle(vehicle_id);
--删除外键
alter table WORKING drop constraint WORKING_FK;
--添加检查约束 (如果数据表中数据有超出约束范围的添加失败)
alter table VEHICLE add constraint VEHICLE_CHK check (vehicle_type in ('1','2','3'));
--删除约束，只需约束名称即可
alter table VEHICLE drop constraint VEHICLE_CHK;
--删除数据表
drop table VEHICLE purge;
```

<font color=#FF0000 size=5> <p align="center">数据库表操作语句</p></font>

设置 SQL> 终端行宽
SQL> set linesize 120
```sql
alter table c##antman.vehicle rename to newname;                 #修改表名
alter table c##antman.vehicle rename column oldname to newname;  #修改列名
alter table c##antman.vehicle add owner NVARCHAR2(20);           #添加表列
alter table c##antman.vehicle drop column owner;                 #删除表列
alter table VEHICLE modify note NVARCHAR2(500) [default value];  #修改字段类型,参数可选

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
--更新数据 (最后事务提交 commit;)
update VEHICLE set note = 'text' where vehicle_id = 'MAA3006';
--建立一个快表将working表数据复制一份
create table quici_table as select * from working;
--查询数据
select avg(vehicle_type) as avg,sum(vehicle_type) as sum from VEHICLE where note is not null;
select * from WORKING where switch_time >= to_date('2018-09-28 00:02:00','yyyy-mm-dd hh24:mi:ss');
select * from WORKING where switch_time > sysdate-1/24;    #一小时以前
--查询现在往前两年内的数据
select * from WORKING where months_between(sysdate,switch_time)/12<2;
--以首字母大写的方式显示 vehicle_id 列数据
select initcap(vehicle_id) as vehicle_name from WORKING;
--模糊查询 vehicle_id 列 至少含一个字符的数据
select * from VEHICLE where vehicle_id like '%_%';
--查询 VEHICLE 表，优先按照 vehicle_type 降序，再按照 vehicle_id 升序 进行显示
select vehicle_id,vehicle_type from VEHICLE order by vehicle_type desc,vehicle_id;
--按照日期的年份进行排序显示,还可以使用month，day进行显示；extract 是提取函数
select extract(year from switch_time) as year from WORKING order by year;
--查询 VEHICLE 表中的数据，同时 vehicle_id 包含在 WORKING 表中且等于 'MAT0532'
select * from VEHICLE where vehicle_id in (select vehicle_id from WORKING) and vehicle_id = 'MAT0532';
--按照不同数据组进行分组显示,同时只显示 count(vehicle_type) > 2 的数据;
select vehicle_type,count(vehicle_type) from VEHICLE group by vehicle_type having count(vehicle_type) > 2;
--删除数据，行号不在查找出来的数据
delete from WORKING t where rowid not in (select min(rowid) from WORKING group by vehicle_id)
--会让更新进程处于等待状态，必须跟随 commit;
select * from VEHICLE for update;
--获取当前月份的第一天，扩展 'year'
select trunc(sysdate, 'month') "first day"  from dual;
--获取当前月份的最后一天,trunc函数不显示时间
select trunc(last_day(sysdate)) "last day" from dual;
--获取当前月份的天数
select cast(to_char(last_day(sysdate), 'dd') as int) month_days from dual;
--获取当前月份剩下的天数
select sysdate,last_day(sysdate), last_day(sysdate)-sysdate from dual;
--获取两个日期之间的天数
select round((months_between('01-feb-2014', '01-mar-2012') * 30), 0) days from dual;
select trunc(sysdate) - trunc(w.switch_time) from WORKING w;
--获取直到目前为止今天过去的秒数（从 00：00 开始算）
select (sysdate-trunc(sysdate)) * 24 * 60 * 60 sec_since_morning from dual;
--在 Oracle 中生成随机数值
select round(dbms_random.value () * 100) as random_num from dual;
--检查表中是否含有任何的数据
select 1 from install where rownum = 1;
--把数值转换成文字
select to_char (to_date (1526, 'j'), 'jsp') from dual;
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
查询 user_objects 视图中的数据来检查视图的状态。其中对象名称必须是大写的
>SQL> select object_name,status from user_objects where object_name='VEHICLE_VIEW';

可以设置 SQL_TRACE 参数跟踪查看 SQL 语句执行具体过程
>SQL> alter session set sql_trace=true;

数据库去重有以下那么三种方法
```
1.两条记录或者多条记录的每一个字段值完全相同，这种情况去重复最简单，用关键字 distinct 就可以去掉
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
select * from VEHICLE v, WORKING w where v.vehicle_id(+) = w.vehicle_id;

3.全连接的查询结果是左外连接和右外连接查询结果的并集
select * from VEHICLE v full join WORKING w on 1=1;
```

#从pfile文件启动数据库实例 (默认是spfile文件启动)
```
SQL> startup pfile='/opt/oracle/product/OraHome/dbs/initorcl.ora';
startup 启动查找顺序是 spfileSID.ora->spfile.ora->initSID.ora->init.ora(spfile优先于pfile)
pfile和spfile可以互相创建，同时可以指定参数文件的位置
SQL> create spfile[='xxxxx'] from pfile[='xxxx'];
SQL> create pfile[='xxxxx'] from spfile[='xxxx'];
```

oracle数据库表数据导出与还原有三种主要的模式(完全、用户、表)
```
1.完全模式,将orcl数据库完全导出，system用户必须具有特殊的权限
[oracle@dwj ~]$ exp system/system@orcl file=~/backup_$(date +%Y-%m-%d).dmp log=~/backup.log full=y
[oracle@dwj ~]$ imp system/system@orcl file=~/backup_$(date +%Y-%m-%d).dmp log=~/backup.log full=y

2.用户模式，导入必须指定 fromuser、touser 参数，这样才能导入数据;[owner]可选参数
[oracle@dwj ~]$ exp system/system@orcl file=~/backup.dmp log=~/backup.log [owner=system,c##antman]
[oracle@dwj ~]$ imp system/system@orcl file=~/backup.dmp log=~/backup.log fromuser=system touser=system

3.表模式,用户 system 的表 help,work 被导出; owner 导出整个用户或者多个用户,而 tables 表示只导出其中的表
[oracle@dwj ~]$ exp system/system@orcl file=~/backup.dmp log=~/backup.log tables=help,work
[oracle@dwj ~]$ imp system/system@orcl file=~/backup.dmp log=~/backup.log tables=help,work

导出表中部分数据，如果表存在，则不会导入数据，参数 [ignore=y] 可以确保数据导入
exp c##antman/ant@orcl file=~/backup_table.dmp tables=vehicle query="'where vehicle_type<3'" [ignore=y]

EXP参数里面可能发生冲突的组合：
1.同时指定了 owner 和 tables
2.同时指定了 FULL 和 tables
3.同时指定了多个 owner 和 full
```

<font color=#FF0000 size=5> <p align="center">Oracle RMAN 备份及恢复步骤</p></font>

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
[oracle@dwj ~]$ rman target=sys/sys@orcl;        #关闭数据库后,同时链接失效,无法再次登录
[oracle@dwj ~]$ rman target '"/ as sysdba"'
[oracle@dwj ~]$ rman target '"/ as sysbackup"'
```
3、基本设置
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
4、查看所有设置
>RMAN> show all;

5、查看数据库物理结构-报表模式(后面第7，9条命令会使用相关字段数据，eg：File and Tablespace)
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

RMAN> delete expired backup;   #删除expired备份
RMAN> delete expired copy;     #删除expired副本
RMAN> delete backupset 19;     #删除特定备份集
RMAN> delete copy;             #删除所有映像副本
RMAN> delete backup;           #删除所有备份集
```
删除 7 天以前的归档日志
>RMAN> delete force archivelog all completed before 'sysdate-7';

<font color=#FF0000 size=5> <p align="center">Oracle 恢复误删除的表</p></font>

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
