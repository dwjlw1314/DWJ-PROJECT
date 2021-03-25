<font color=#FF0000 size=5> <p align="center">数据库名</p></font>

数据库名就是一个数据库的标识，就像人的身份证号一样。他用参数DB_NAME表示，如果一台机器上装了多个数据库，那么每一个数据库都有一个数据库名。在数据库安装或创建完成之后，参数DB_NAME被写入参数文件之中。格式如下：
DB_NAME=orcl

在创建数据库时就应考虑好数据库名，并且在创建完数据库之后，数据库名不宜修改，即使要修改也会很麻烦。因为，数据库名还被写入控制文件中，控制文件是以 二进制型式存储的，用户无法修改控制文件的内容。假设用户修改了参数文件中的数据库名，即修改DB_NAME的值。但是在Oracle启动时，由于参数文 件中的DB_NAME与控制文件中的数据库名不一致，导致数据库启动失败，将返回ORA-01103错误

数据库名的作用是在安装数据库、创建新的数据库、创建数据库控制文件、修改数据结构、备份与恢复数据库时使用到

有很多Oracle安装文件目录是与数据库名相关的，如：
```
winnt: d:\oracle\product\10.1.0\oradata\DB_NAME\...
Unix: /home/app/oracle/product/10.1.0/oradata/DB_NAME/...

pfile:
winnt: d:\oracle\product\10.1.0\admin\DB_NAME\pfile\ini.ora
Unix: /home/app/oracle/product/10.1.0/admin/DB_NAME/pfile/init$ORACLE_SID.ora

跟踪文件目录：
winnt: /home/app/oracle/product/10.1.0/admin/DB_NAME/bdump/...

create database命令中的数据库名也要与参数文件中DB_NAME参数的值一致，否则将产生错误。同样，修改数据库结构的语句alter database，当然也要指出要修改的数据库的名称

如果控制文件损坏或丢失，数据库将不能加载，这时要重新创建控制文件，方法是以nomount方式启动实例，然后以create controlfile命令创建控制文件，这个命令中也是指DB_NAME

还有在备份或恢复数据库时，都需要用到数据库名

查询当前数据名
方法一: select name from v$database;
方法二：show parameter db
方法三：查看参数文件
```

如何在已创建数据库之后，修改数据库名。步骤如下：
```
1.关闭数据库
2.修改数据库参数文件中的DB_NAME参数的值为新的数据库名
3.以NOMOUNT方式启动实例，修建控制文件(有关创建控制文件的命令语法，请参考oracle文档)
```

<font color=#FF0000 size=5> <p align="center">数据库实例名</p></font>

数据库实例名是用于和操作系统进行联系的标识，就是说数据库和操作系统之间的交互用的是数据库实例名。实例名也被写入参数文件中，该参数为instance_name，在winnt平台中，实例名同时也被写入注册表

在一般情况下，数据库名和实例名是一对一的关系，也可以不同，但如果在oracle并行服务器架构(即oracle实时应用集群)中，数据库名和实例名是一对多的关系

查询当前数据库实例名
```
方法一：select instance_name from v$instance;
方法二：show parameter instance
方法三：在参数文件中查询
```
数据库实例名与ORACLE_SID，虽然都是oracle实例，但两者是有区别的。instance_name是oracle数据库参数。而ORACLE_SID是操作系统的环境变量。ORACLD_SID用于与操作系统交互，也就是说，从操作系统的角度访问实例名，必须通过ORACLE_SID。在winnt平台，ORACLE_SID还需存在于注册表中。

且ORACLE_SID必须与instance_name的值一致，否则，你将会收到一个错误 <br>
在unix平台，是“ORACLE not available”,在winnt平台，是“TNS:协议适配器错误”

数据库实例名与网络连接

数据库实例名除了与操作系统交互外，还用于网络连接的oracle服务器标识。当你配置oracle主机连接串的时候，就需要指定实例名。当然8i以后版本的网络组件要求使用的是服务名SERVICE_NAME

<font color=#FF0000 size=5> <p align="center">数据库域名</p></font>

在分布式数据库系统中，不同版本的数据库服务器之间，不论运行的操作系统是unix或是windows，各服务器之间都可以通过数据库链路进行远程复制，数据库域名主要用于oracle分布式环境中的复制。举例说明如：

全国交通运政系统的分布式数据库，其中：
```
福建节点： fj.jtyz
福建厦门节点： xm.fj.jtyz
江西： jx.jtyz
江西上饶：sr.jx.jtyz
这就是数据库域名
```
数据库域名在存在于参数文件中，他的参数是db_domain
```
方法一：select value from v$parameter where name = 'db_domain';
方法二：show parameter domain
方法三：在参数文件中查询。
```

全局数据库名=数据库名+数据库域名，如前述福建节点的全局数据库名是：oradb.fj.jtyz

<font color=#FF0000 size=5> <p align="center">数据库服务名</p></font>

从oracle9i版本开始，引入了一个新的参数，即数据库服务名。参数名是SERVICE_NAME，如果数据库有域名，则数据库服务名就是全局数据库名；否则，数据库服务名与数据库名相同。

查询数据库服务名
```
方法一：select value from v$parameter where name = 'service_name';
方法二：show parameter service_name
方法三：在参数文件中查询
```
数据库服务名与网络连接，从oracle8i开如的oracle网络组件，数据库与客户端的连接主机串使用数据库服务名。之前用的是ORACLE_SID,即数据库实例名

<font color=#FF0000 size=5> <p align="center">Schema</p></font>

创建Schema，尽管DBMS在定义schema方面有所不同，但是其有一个共同点，就是每一个都支持CREATE SCHEMA语句

在MySQL中，CREATE SCHEMA创建了一个数据库，这是因为CREATE SCHEMA是CREATE DATABASE的同义词。换句话说，你可以使用CREATE SCHEMA或者CREATE DATABASE来创建一个数据库

在Oracle中，CREATE SCHEMA语句实际上并不创建一个模式，这是因为在创建用户时，用户就已经创建了一个模式，也就是说在ORACLE中CREATE USER就创建了一个schema，CREATE SCHEMA语句允许你将schema同表和视图关联起来，并在这些对象上授权，从而不必在多个事务中发出多个SQL语句

在SQL Server中，CREATE SCHEMA将按照名称创建一个模式，与MySQL不同，CREATE SCHEMA语句创建了一个单独定义到数据库的模式。和ORACLE也不同，CREATE SCHEMA语句实际创建了一个模式，一旦创建了schema，就可以往模式中添加用户和对象

<font color=#FF0000 size=5> <p align="center">Oracle参数文件</p></font>

有两种创建数据库的方式，一种是以命令行脚本方式，即手动方式创建；另一种是利用Oracle提供的数据库配置向导来创建。本篇主要介绍在Unix和Windows下以命令行脚本方式创建Oracle数据库

一个完整的数据库系统，应包括一个物理结构、一个逻辑结构、一个内存结构和一个进程结构，如果要创建一个新的数据库，则这些结构都必须完整的建立起来

参数文件决定着数据库的总体结构，说明如下：
```
数据库标识类参数

DB_NAME: 数据库名，此参数在创建数据前决定，数据库创建后修改时，必须建控制文件
DB_DOMAIN: 数据库域名，用于区别同名数据库。数据库名与域名一起构成了全局数据库名
INSTANCE_NAME: 数据库实例名，可以与数据库相同
SERVICE_NAMES: 数据库服务名，与全局数据库名相同如果没有域名，则服务名就是数据库名

b.日志管理类参数

LOG_ARCHIVE_START: 是否启动自动归档进程ARCH
LOG_ARCHIVE_DEST: 归档日志文件存储目录
LOG_ARCHIVE_FORMAT: 归档日志文件的默认文件存储格式
LOG_ARCHIVE_DUPLEX_DEST: 归档日志文件镜像存储目录（Oracle8以上）
LOG_ARCHIVE_DEST_n: 归档日志文件存储目录（Oracle8i以上）
LOG_ARCHIVE_DEST_STATE_n: 设置参数LOG_ARCHIVE_DEST_n失效或生效
LOG_ARCHIVE_MAX_PROCESSES: 设置自动归档进程的个数
LOG_ARCHIVE_MIN_SUCCEED_DEST: 设置最少的成功归档日志存储目录的个数
LOG_CHECKPOINT_INTERVAL: 根据日志数量设置检验点频率
LOG_CHECKPOINT_TIMEOUT: 根据时间间隔设置检验点频率

c.内存管理参数

DB_BLOCK_SIZE: 标准数据块大小
DB_nK_CACHE_SIZE: 非标准数据块数据缓冲区大小
SHARED_POOL_SIZE: 共享池大小控制参数，单位为字节
DB_CACHE_SIZE: 标准数据块数据缓冲区大小
DB_BLOCK_BUFFERS: 数据缓冲区大小，9i之后已放弃使用
LOG_BUFFER: 日志缓冲区大小
SORT_AREA_SIZE: 排序区大小
LARGE_POOL_SIZE: 大池大小
JAVA_POOL_SIZE:Java池大小

d.最大许可用户数量限制参数

LICENSE_MAX_SESSIONS:数据库可以连接的最大会话数
LICENSE_MAX_USERS:数据库支持的最大用户数
LICENSE_MAX_WARNING:数据库最大警告会数（会话数据达到这个值时，产生新会话时就会产生警告信息）

e.系统跟踪信息管理参数

USER_DUMP_DEST:用户跟踪文件生成的设置
BACKGROUND_DUMP_DEST:后台进程跟踪文件生成的位置
MAX_DUMPFILE_SIZE:跟踪文件的最大尺寸

f.系统性能优化与动态统计参数

SQL_TRACE:设置SQL跟踪
TIMED_STATICS:设置动态统计
AUDIT_TRAIL:启动数据库审计功能

g.其他系统参数

CONTROL_FILES:控制文件名及路径
Undo_MANAGMENT:Undo空间管理方式
ROLLBACK_SEGMENTS:为这个例程分配的回退段名
OPEN_CURSORS:一个用户一次可以打开的游标的最大值
PROCESSES:最大进程数，包括后台进程与服务器进程
IFILE:另一个参数文件的名字
DB_RECOVERY_FILE_DEST:自动数据库备份目录
DB_RECOVERY_FILE_SIZE:数据库备份文件大小

2）参数文件样式

db_name=myoracle
instance_name=myoracle
db_domain=fangys.xiya.com
service_names=myoracle.fangys.xiya.com
control_files=(/home/app/oracle/product/10.1.0/oradata/myoracle/control01.ctl,
                /home/app/oracle/product/10.1.0/oradata/myoracle/control02.ctl,
                /home/app/oracle/product/10.1.0/oradata/myoracle/control03.ctl)

db_block_size=8192
user_dump_dest=/home/app/oracle/product/10.1.0/admin/myoracle/udump
background_dump_dest=/home/app/oracle/product/10.1.0/admin/myoracle/bdump
core_dump_dest=/home/app/oracle/product/10.1.0/admin/myoracle/cdump
db_recovery_file_dest=/home/app/oracle/product/10.1.0/flash_recover_area
db_recovery_file_size=100G
```

在数据库创建结束后，数据库自动处于OPEN状态下，这时所有V$××××类数据字典都可以查询。而其它数据字典，如DBA_DATA_FILES、DBA_TABLESPACES等都不存在，必须通过下列骤为系统创建数据字典
```
1)加载常用的数据字典包
sql>@/home/app/oracle/product/10.1.0/db_1/rdbms/catalog

2)加载PL/SQL程序包
sql>@/home/app/oracle/product/10.1.0/db_1/rdbms/admin/catproc

3)加载数据复制支持软件包
sql>@/home/app/oracle/product/10.1.0/db_1/rdbms/admin/catrep

4)加载Java程序包
sql>@/home/app/oracle/product/10.1.0/db_1/javavm/install/initjvm

5)加载系统环境文件
sql>connect system/pass
sql>@/home/app/oracle/product/10.1.0/db_1/sqlplus/admin/pupbld
```

<font color=#FF0000 size=5> <p align="center">数据库字符集</p></font>

```
在整个运行环节中，字符集在3个环节中发挥作用：
1.软件在操作系统上运作时的对用户的显示，此时采用操作系统定义的字符集进行显示。sqlplus是运行在操作系统上的软件，当然要使用系统所指定的字符集对外显示内容
2.数据向oracle服务端传送前的通告。也就是sqlplus告诉服务器现在使用的字符集是什么
3.数据流到达服务器后，按照服务器所使用的字符集自动翻译客户端的数据，然后存储进系统
```

```
在客户端sqlplus和服务端传送数据，数据会按照服务端字符集进行翻译，这个过程是自动完成的不需要人工干预。客户端要想正确显示从服务端读取到的数据，也需要按照本地的字符集设置进行翻译，这个过程也是自动的

sqlplus作为运行在操作系统上的软件，必然使用操作系统所使用的字符集设置。sqlplus设置的字符集，作用只有一个，那就是通告服务器端，为相互之间的字符集翻译做准备

客户端的字符集设置是在NLS_LANG环境变量中设置的，客户读端的字符集可以和oracle客户端设置得不一样（本来人家就是自动翻译的），但是客户端字符集一定要和操作系统设置的字符集相匹配！

考虑一下，sqlplus使用的是操作系统的字符集定义显示和发送数据（假设是TYPE_A），却告诉oracle服务器自己使用的字符集是TYPE_B，oracle服务器会将客户端发送过来的TYPE_A数据当作TYPE_B字符集格式用自身的TYPE_C字符集进行翻译，然后再存储进系统，这就形成了乱码。反向的过程类似，Oracle服务器发出的数据格式是TYPE_C，但是客户端软件认为自己使用的编码是TYPE_B并进行了翻译，交给操作系统用TYPE_A字符集进行翻译显示，最终导致了无法正确显示

一个现实的例子：RHEL5.8操作系统安装了中文支持包以后，所有的语言环境都被设置成了zh_CN.UTF-8（通过LANG环境变量可知，也可通过locale命令查到）,数据库服务器所使用的字符集为ZHS16GBK,很明显，这两者不一致，sqlplus在没有设置NLS_LANG环境变量时，与数据库保持一致，此时，从服务端取得的数据不需要翻译，被sqlplus读取并用zh_CN.UTF-8的字符编码映射关系进行翻译显示，所有的汉字变成了?

根据上面的分析，要解决这一问题，要把sqlplus的字符集设置成和操作系统一致即可，操作系统设置的是zh_CN.UTF-8，但在.bash_profile里面还不能直接将NLS_LANG设置为zh_CN.UTF-8，因为这个zh_CN.UTF8是字符集的localeID而不是字符集的名称，真正的名称叫SIMPLIFIEDCHINESE_CHINA.AL32UTF8
如果设置成zh_CN.UTF8，oracle会报ORA-12705: Cannotaccess NLS data files or invalid environmentspecified错误
在.bash_profile里面加入NLS_LANG="SIMPLIFIEDCHINESE_CHINA.AL32UTF8"; export NLS_LANG 问题就解决了
```

不同平台的一些细节：
```
Windows（ 如简体系统为：ZHS16GBK，繁体系统为：MSWIN950 ）

1、设置session变量

# 常用中文字符集
set NLS_LANG=SIMPLIFIED CHINESE_CHINA.ZHS16GBK
# 常用unicode字符集
set NLS_LANG=american_america.AL32UTF8

2、可以通过修改注册表键值永久设置
HKEY_LOCAL_MACHINE/SOFTWARE/ORACLE/HOMExx/NLS_LANG

3、设置环境变量
NLS_LANG=SIMPLIFIED CHINESE_CHINA.ZHS16GBK
NLS_LANG=american_america.AL32UTF8

Unix/Linus:

1、设置session变量

# 常用unicode字符集
export NLS_LANG=american_america.AL32UTF8
# 常用中文字符集
export NLS_LANG="Simplified Chinese_china".ZHS16GBK

2、编辑 bash_profile文件进行永久设置
# vim .bash_profile
NLS_LANG="Simplified Chinese_china".ZHS16GBK
export NLS_LANG
```

1、查看sqlplus客户编码：
>[oracle@dwj ~]$ echo $NLS_LANG

2、查看系统编码：
>[oracle@dwj ~]$ locale

3、查看数据库字符集，执行如下查询:
>SQL> select userenv('language') from dual;

Oracle字符集转换的基本原则：
```
设置客户端的NLS_LANG为客户端操作系统的字符集
如果数据库字符集等于NLS_LANG，数据库和客户端传输字符时不作任何转换
如果它们俩不等，则需要在不同字符集间转换，只有客户端操作系统字符集是数据库字符集子集的基础上才能正确转换，否则会出现乱码
```

参考：https://www.cnblogs.com/bingo1717/p/7803359.html

<font color=#FF0000 size=5> <p align="center">游标</p></font>

隐式游标，隐式游标的好处是不需要手动关闭，方便
```sql
for currow in (
   select t.col1, t.col2
   from tableName t
   where ...
) loop
    if currow.col1 = 0 then
       return;    -- 中止sp，返回
   end if;
end loop;
```
显式游标
```sql
declare
isok integer;
v_event_id number(10);
v_isagain number(2);
v_rate number(2);

v_sender char(11) := '13800138000';

cursor cursorVar is select event_id, isagain, rate from call_event where sender = v_sender; -- 声明游标

begin
    open cursorVar;    -- 打开游标
    loop
         fetch cursorVar into v_event_id, v_isagain, v_rate;       -- 取值
         exit when cursorVar%notfound;                             -- 当没有记录时退出循环
         dbms_output.put_line(v_event_id || ', ' || v_isagain || ', ' || v_rate);
    end loop;

    close cursorVar;   -- 关闭游标

    --游标的属性有：%FOUND,%NOTFOUNRD,%ISOPEN,%ROWCOUNT;
    --%FOUND:已检索到记录时，返回true
    --%NOTFOUNRD：检索不到记录时，返回true
    --%ISOPEN:游标已打开时返回true
    --%ROWCOUNT:代表检索的记录数，从1开始
end;
```
带参数游标
```sql
declare
isok integer;
v_event_id number(10);
v_isagain number(2);
v_rate number(2);

v_sender char(11) := '13800138000';

cursor cursorVar(p_sender varchar2) is select event_id, isagain, rate from call_event where sender = p_sender; -- 声明游标

begin
    open cursorVar(v_sender);    -- 打开游标，在括号里传参
    loop
         fetch cursorVar into v_event_id, v_isagain, v_rate;       -- 取值
         exit when cursorVar%notfound;                             --当没有记录时退出循环
         dbms_output.put_line(v_event_id || ', ' || v_isagain || ', ' || v_rate);
    end loop;

    close cursorVar;   -- 关闭游标
end;
```

<font color=#FF0000 size=5> <p align="center">物化视图</p></font>

创建物化视图语句(默认情况下，如果没指定刷新方法和刷新模式，则Oracle默认为FORCE和DEMAND)
> create materialized view mv_name as select * from table_name

创建定时刷新的物化视图(指定物化视图每天刷新一次,没有指定刷新时间)
> create materialized view mv_name refresh force on demand start with sysdate next sysdate+1

创建指定时间刷新的物化视图
> create materialized view mv_name refresh force on demand start with sysdate next to_date( concat( to_char( sysdate+1,'dd-mm-yyyy'),' 22:00:00'),'dd-mm-yyyy hh24:mi:ss')

物化视图的删除
> drop materialized view mv_name

物化视图的特点
```
1.物化视图在某种意义上说就是一个物理表(而且不仅仅是一个物理表)，这通过其可以被user_tables查询出来，而得到佐证
2.物化视图也是一种段(segment)，所以其有自己的物理存储属性
3.物化视图会占用数据库磁盘空间
```

物化视图数据随着基表更新方式
```
oracle提供手工刷新和自动刷新，默认为手工刷新。也就是说，通过手工执行某个Oracle提供的系统级存储过程或包，来保证物化视图与基表数据一致性
自动刷新，其实也就是Oracle会建立一个job，通过这个job来调用相同的存储过程或包，加以实现
刷新模式有两种：ON DEMAND和ON COMMIT，前者不刷新(手工或自动)就不更新物化视图，而后者不刷新也会更新物化视图，只要基表发生了COMMIT
刷新方法有四种：FAST、COMPLETE、FORCE和NEVER
FAST 刷新采用增量刷新，只刷新自上次刷新以后进行的修改
COMPLETE 刷新对整个物化视图进行完全的刷新
FORCE Oracle在刷新时会去判断是否可以进行快速刷新，如果可以则采用FAST方式，否则采用COMPLETE的方式
NEVER 不进行任何刷新
```

Oracle 表分区思维导图

![image](https://github.com/dwjlw1314/DWJ-PROJECT/raw/master/PictureSource/3.30.1.png)

LOB类型相关描述
```
oracle中内置的LOB数据类型包括BLOB、CLOB、NCLOB、BFILE（外部存储）的大型化和非结构化数据，如文本、图像、视屏、空间数据存储

CLOB 数据类型
它存储单字节和多字节字符数据。支持固定宽度和可变宽度的字符集。CLOB对象可以存储最多 (4 gigabytes-1) * (database block size) 大小的字符

NCLOB 数据类型
它存储UNICODE类型的数据，支持固定宽度和可变宽度的字符集，NCLOB对象可以存储最多(4 gigabytes-1) * (database block size)大小的文本数据

BLOB 数据类型
它存储非结构化的二进制数据大对象，它可以被认为是没有字符集语义的比特流，一般是图像、声音、视频等文件。BLOB对象最多存储(4 gigabytes-1) * (database block size)的二进制数据

BFILE 数据类型
二进制文件，存储在数据库外的系统文件，只读的，数据库会将该文件当二进制文件处理
```
