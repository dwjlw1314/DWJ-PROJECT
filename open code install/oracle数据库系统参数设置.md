<font color=#FF0000 size=5> <p align="center">Oracle系统参数</p></font>

https://blog.csdn.net/dzq_boyka/article/details/80337187 <br>
https://blog.csdn.net/babymouse1212/article/details/71078874

Oracle提供了大量的系统参数，下面是视图查询方法
```SQL
SQL> select count(*) from v$spparameter;
SQL> select count(*) from v$parameter;
SQL> select count(*) from v$parameter2;
SQL> select count(*) from v$system_parameter;
SQL> select count(*) from v$system_parameter2;
```

v$parameter，存放session级的参数，如果没有被"alter session"修改，默认和system级的参数值相同
```
视图关键字段:
NAME             ---参数名
TYPE             ---参数类型
VALUE            ---当前session的参数值
DISPLAY_VALUE    ---用户可读的参数值
ISDEFAULT        ---是否是系统默认值
ISSES_MODIFIABLE ---true表示参数能通过"alter session"被改变，false表示不能改变
ISSYS_MODIFIABLE ---参数是否能被"alter system"改变，改变后：
    FALSE：表示不能改变
    IMMEDIATE：参数可以通过"alter system"改变，立即生效
    DEFERRED：参数可以通过"alter system"改变，在下一个session开始生效
ISINSTANCE_MODIFIABLE ---true表示参数值在每个实例下可以是不同的，false表示所有实例必须具有相同的值
    如果ISSYS_MODIFIABLE为false，则该值总是false
ISMODIFIED       ---表示参数是否在实例启动后被修改
    FALSE：在实例启动后没有被修改
    MODIFIED：参数被使用"alter session"修改
    SYSTEM_MOD：参数被使用"alter system"修改
ISDEPRECATED     ---true表示该参数被弃用，否则false
DESCRIPTION      ---参数的描述信息
UPDATE_COMMENT   ---最近一次修改的注释
HASH             ---参数名的哈希值
```

v$system_parameter，存放实例级别的参数，新的session将从这里继承参数值
```
视图关键字段:
TYPE             ---参数类型:
    1-Boolean
    2-String
    3-Integer
    4-Parameter file
    5-保留
    6-Big integer
其他参数和v$parameter相同
```
v$parameter2,和v$parameter相同，唯一的区别是如果一个参数有多个值，那么在v$parameter2中将有多行，而在v$parameter中则只有一行，在value中使用逗号分隔多个值

v$system_parameter2，类似于v$parameter2

v$spparameter，用于存放服务器参数文件(spfile)的参数信息，如果服务器参数文件没有被用于启动实例，则视图每行的ISSPECIFIED列的值都为false
```
视图关键字段:
SID            ---参数的SID
NAME           ---参数名
VALUE          ---参数值(如果服务器参数文件没有被用于启动实例，则为null)
DISPLAY_VALUE  ---参数值，采用用户友好的格式
ISSPECIFIED    ---true表示参数在服务器参数文件中指定，否则false
ORDINAL        ---参数值的位置(序号),如果服务端配置文件没被用于启动实例，则为0。只有当参数值为一个列表时才使用
UPDATE_COMMENT ---最近一次修改的注释(如果服务器参数文件没有被用于启动实例，则为null)
```

Oracle系统中还有一类参数称之为隐藏参数(hidden parameters)，是系统中使用，但Oracle官方没有公布的参数，这些参数可能是那些还没有成熟或者是系统开发中使用的参数。这些参数在所有Oracle官方提供的文档中都没有介绍，他们的命名有一个共同特征就是都以_作为参数的首字符，和隐藏参数相关的视图有x$ksppi、x$ksppcv和x$ksppsv

x$ksppi是v$parameter、v$parameter2、v$system_parameter和v$system_parameter2的基础表，保存参数信息
```
视图关键字段:
ADDR      ---内存地址
INDX      ---序号
INST_ID   ---实例编号
KSPPINM   ---参数名称
KSPPITY   ---参数类型：
    1-Boolean
    2-String
    3-Integer
    4-Parameter file
KSPPDESC  ---参数描述信息
KSPPIFLG  ---标志,用来说明isses_modifiable或者issys_modifiable
```

x$ksppcv,保存当前session的参数值，和x$ksppi用indx关联
```
视图关键字段:
KSPPSTVL   ---参数的当前值
KSPPSTDF   ---参数的默认值
KSPPSTVF   ---标志字段，用来说明('Modified'、'System Modified'或is_adjusted)
KSPPSTCMNT ---注释
```
x$ksppsv,保存系统参数值，和x$ksppi用indx关联，字段和x$ksppcv基本一致

查询隐藏参数和当前session的参数值和默认值
```SQL
select ksppinm "Parameter Name", ksppstvl "Value", ksppstdf "Default"
  from x$ksppi x, x$ksppcv y where x.indx = y.indx
  and ksppinm like '/_%trace%' escape '/';
```
查看隐藏参数，并显示当前session和实例的参数值
```
select a.ksppinm  Parameter,a.ksppdesc Description,
  b.ksppstvl "Session Value",c.ksppstvl "Instance Value"
  from x$ksppi a, x$ksppcv b, x$ksppsv c where a.indx = b.indx  
  and a.indx = c.indx and a.ksppinm like '\_%' escape '\';
```

Oracle中存在一些新版本中废弃的参数，可以在视图V$OBSOLETE_PARAMETER中查找到，该视图值包含两个字段，
name(参数名)和ISSPECIFIED(true表示参数在参数文件中指定，false表示没有。一般情况下该值都应该为false)
>SQL> select name, isspecified from v$obsolete_parameter; #查询隐藏参数和ISSPECIFIED值

v$parameter_valid_values,包含了参数的有效值列表
```
视图关键字段:
1）NUM        ---参数编号
2）NAME       ---参数名
3）ORDINAL    ---同一个参数的有效值序列号(从1开始)
4）VALUE      ---参数值
5）ISDEFAULT  ---该值是否为参数的默认值
```
查看参数optimizer_features_enable的所有有效值
>SQL> select * from v$parameter_valid_values where name = 'optimizer_features_enable';

参数修改4个典型参数做实验：
>SQL> select name,ISSES_MODIFIABLE,ISSYS_MODIFIABLE from v$parameter where name in ('workarea_size_policy','audit_file_dest','sga_target','sga_max_size');

```
NAME                        ISSES_MODIFIABLE            ISSYS_MODIFIABLE
--------------------------- --------------------------- ----------------------------
sga_max_size                   FALSE                    FALSE
sga_target                     FALSE                    IMMEDIATE
audit_file_dest                FALSE                    DEFERRED
workarea_size_policy           TRUE                     IMMEDIATE
```
1.workarea_size_policy可以alter session修改

查看原来的配置：
>SQL> show parameter workarea_size_policy

```
NAME                                 TYPE        VALUE
------------------------------------ ----------- -----------------
workarea_size_policy                 string      AUTO
```
在session级别修改：
>SQL> alter session set workarea_size_policy=MANUAL;  <br>
Session altered.

在本session查看，可以发现修改已经生效：
>SQL> show parameter workarea_size_policy

```
NAME                                 TYPE        VALUE
------------------------------------ ----------- -------------------
workarea_size_policy                 string      MANUAL
```
2.sga_target在用alter system修改后立即生效

查看原来的配置：
>SQL> show parameter sga_target

```
NAME                                 TYPE        VALUE
------------------------------------ ----------- --------------
sga_target                           big integer 30G
```
>SQL> alter system set sga_target=32G;  <br>
System altered.

用alter system修改后立即生效：
>SQL> show parameter sga_target

```
NAME                                 TYPE        VALUE
------------------------------------ ----------- ------------------
sga_target                           big integer 32G
```
3.audit_file_dest在用alter system修改后，知道下个session才生效

查看原来的配置：
>SQL> show parameter audit_file_dest

```
NAME                          TYPE        VALUE
----------------------------- ----------- -----------------------
audit_file_dest               string      /u01/app/oracle/admin/santdb/adump
```
注意：后面必须得加关键字deferred，否则会报错
>SQL> alter system set audit_file_dest='/u01/app/oracle/adump' deferred;  <br>
System altered.

在本session里查询还是原值，没有改变：
>SQL> show parameter audit_file

```
NAME                          TYPE        VALUE
----------------------------- ----------- -------------------------
audit_file_dest               string      /u01/app/oracle/admin/santdb/adump
```
重新开个session，在查询发现已经改为新值了：
>SQL> show parameter audit_file_dest

```
NAME                          TYPE        VALUE
----------------------------- ----------- -----------------------------
audit_file_dest               string      /u01/app/oracle/adump
```
4.sga_max_size在用alter system修改后必须重启实例才能生效

查看原来的配置：
>SQL> show parameter sga_max_size

NAME                                 TYPE        VALUE
------------------------------------ ----------- -----------------
sga_max_size                         big integer 16G

注意：后面必须得加scope=spfile，否则会报错
>SQL> alter system set sga_max_size=32G scope=spfile;  <br>
System altered.

如果数据库没重启，无论如何还是原来的配置：
>SQL> show parameter sga_max_size

```
NAME                                 TYPE        VALUE
------------------------------------ ----------- ------------
sga_max_size                         big integer 30G
```
重启数据库：
```
SQL> shutdown immediate;
Database closed.
Database dismounted.
ORACLE instance shut down.
SQL> startup
ORACLE instance started.

Total System Global Area 1468006400 bytes
Fixed Size                  1303076 bytes
Variable Size             612371932 bytes
Database Buffers          847249408 bytes
Redo Buffers                7081984 bytes
Database mounted.
Database opened.
```
再重新查询，就可以看到用的是新值了：
>SQL> show parameter sga_max_size

```
NAME                                 TYPE        VALUE
------------------------------------ ----------- -------------
sga_max_size                         big integer 32G
```
5.db_files值在用alter system修改后必须重启实例才能生效

查看原来的配置：
>SQL> show parameter db_files;

NAME                                 TYPE        VALUE
------------------------------------ ----------- -----------------
db_files                             integer     200

注意：后面必须得加scope=spfile，否则会报错
>SQL> alter system set db_files=2000 scope=spfile;  <br>
System altered.

重启数据库（RAC集群需要在所有节点进行修改重启）：
```
SQL> shutdown immediate;
Database closed.
Database dismounted.
ORACLE instance shut down.
SQL> startup
ORACLE instance started.

Total System Global Area 1468006400 bytes
Fixed Size                  1303076 bytes
Variable Size             612371932 bytes
Database Buffers          847249408 bytes
Redo Buffers                7081984 bytes
Database mounted.
Database opened.
```

6.更改归档日志路径(db_recovery_file_dest)

查看原来的配置：
>SQL> show parameter db_recovery_file_dest;

NAME                                 TYPE        VALUE
------------------------------------ ----------- -----------------
db_recovery_file_dest                string     +ORADATA

注意：如果报错，后面添加 scope=spfile
>SQL> alter system set db_recovery_file_dest='+ORALOG';  <br>
System altered.

RAC集群所有节点自动生效,无需重启DB实例

Oracle系统体系结构组成包含5个重要部分
```
1. 连接数据库实例服务
2. 服务器进程
3. 文件系统管理
4. 内存区域管理（SGA系统全局区）
5. 后台进程
```

Oracle启动时，自动分配SGA内存，创建oracle内存结构，再启动若干常驻内存的操作系统进程，组成oracle进程结构，内存区域和后台进程构成了一个oracle实例
