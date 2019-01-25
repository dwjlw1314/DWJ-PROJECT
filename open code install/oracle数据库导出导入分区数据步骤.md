假设备份告警数据：表结构名字为 Device_alert,BUSINESS_ALERT

<font color=#FF0000 size=5> <p align="center">备份步骤</p></font>

一、以oracle用户，登录数据库服务器
>[oracle@dwj ~]$ id      #uid=500(oracle) gid=500(oinstall) groups=500(oinstall)

二、创建 /home/oracle/ant_ptsf.sql 脚本，通过脚本创建表分区信息
>[oracle@dwj ~]$ vim ant_ptsf.sql      #内容如下：

```sql
drop table ant_pts;
create table ant_pts (
 T_NAME    VARCHAR2(30),
 P_NAME    VARCHAR2(30),
 P_HVALUE      DATE);

declare
 t_name        varchar2(30);
 p_name        varchar2(30);
 p_value       long;
cursor c1 is select partition_name,table_name,high_value from user_tab_partitions where table_name in ('BUSINESS_ALERT','DEVICE_ALERT');

begin
  open c1;
  loop
    fetch c1 into p_name, t_name, p_value;
    exit when c1%NOTFOUND;
    insert into ant_pts values(t_name, p_name, to_date(dbms_lob.substr(to_clob(p_value),10,11),'yyyy-mm-dd'));
  end loop;
  close c1;
end;
/
commit;
```
SQL语句必须用分号(;)或者斜杠(/)来终止，PL/SQL 块必须用斜杠(/)来终止

使用需要导出数据的用户登录oracle
>[oracle@dwj ~]$ sqlplus c##antman/ant

运行 ant_ptsf.sql 脚本
>SQL> @ant_ptsf.sql;

查看表创建详情
>SQL> desc ant_pts;

三、创建数据导出参数文件
>[oracle@dwj ~]$ vim alertdata.par      #备份2018年1月份数据的参数文件，参数文件内容如下：

```
directory=dump_dir
dumpfile=alertdata.dmp
logfile=alertdata.log
tables= BUSINESS_ALERT,DEVICE_ALERT
include=table_data:"in (select p_name from ant_pts where t_name in ('BUSINESS_ALERT','DEVICE_ALERT') and p_hvalue >= to_date('2018-01-01','yyyy-mm-dd') and  p_hvalue < to_date('2018-02-01','yyyy-mm-dd'))"
```
登录oracle
>[oracle@dwj ~]$ sqlplus / as sysdba

创建dmp文件存储路径,需要使用绝对路径
>[oracle@dwj ~]$ mkdir /home/oracle/dump_dir

sqlplus中为dump导入导出新建目录名称(dump_dir)
>SQL> create directory dump_dir as '/home/oracle/dump_dir';

设置用户的导入导出目录赋读写权限,dump_dir为上条语句创建的目录名称,antman为导出数据用户
>SQL> grant read,write on directory dump_dir to c##antman;

四、从数据库备份数据,执行成功后,在目录/home/oracle/dir生成备份数据文件alertdata.dmp
>[oracle@dwj ~]$ expdp c##antman/ant parfile=alertdata.par

五、删除数据库中已备份的数据

使用需要导出数据的用户登录oracle
>[oracle@dwj ~]$ sqlplus c##antman/ant

创建 del_ptsf.sql 脚本，内容如下：
```sql
set pages 0
set linesize 120
spool del.sql
select 'alter table '||t_name ||' drop partition '||p_name ||';' from ant_pts
where t_name in ('BUSINESS_ALERT','DEVICE_ALERT')
and p_hvalue >= to_date('2018-01-01','yyyy-mm-dd')
and  p_hvalue < to_date('2018-03-01','yyyy-mm-dd')
order by t_name;
spool off
```

运行 del_ptsf.sql 脚本生成删除分区脚本 del.sql
>SQL> @del_ptsf.sql;

编辑脚本del.sql文件,删除文件开始和结束部分的SQL语句及注释说明文字,运行脚本删除分区数据
>SQL> @del.sql;

六、非参数文件方式导入备份的数据
>[oracle@dwj dir]$ impdp c##antman/ant directory=dir dumpfile=alertdata.dmp

七、参数文件方式导入备份的数据
>[oracle@dwj dir]$ impdp antman/ant parfile=imp_alertdata.par

创建参数文件
>[oracle@dwj dir]$ vim imp_alertdata.par

```
directory=dir
dumpfile=alertdata.dmp
logfile=alertdata.log
schemas=antman,ant
```

<font color=#FF0000 size=5> <p align="center">终止imp/exp和expdp/impdp进程运行的方法</p></font>

1.通过dba_datapump_jobs来查询数据泵进程
>sql> select * from dba_datapump_jobs;

2.进入到对应的job中
>[oracle@dwj dir]$ expdp antman/ant attach=SYS_EXPORT_FULL_01

```
Export> stop_job=immediate
Are you sure you wish to stop this job ([yes]/no): yes
[oracle@dwj dir]$
```

3.交互模式
```
CONTINUE_CLIENT    返回到记录模式。假如处于空闲状态, 将重新启动作业
START_JOB          启动恢复当前作业
STATUS             在默认值(0)将显示可用时的新状态的情况下,要监视的频率 (以秒计) 作业状态
STATUS=[interval]
STOP_JOB           顺序关闭执行的作业并退出客户机
STOP_JOB=IMMEDIATE 将立即关闭数据泵作业
```
