<font color=#FF0000 size=5> <p align="center">性能相关查询</p></font>



<font color=#FF0000 size=5> <p align="center">常用语句</p></font>

Optionally initialize the database and enable automatic start:
>[root@dwj ~]# sudo /usr/pgsql-13/bin/postgresql-13-setup initdb
>[root@dwj ~]# sudo systemctl enable postgresql-13
>[root@dwj ~]# sudo systemctl start postgresql-13

创建用户
>postgres=# CREATE USER postgres WITH PASSWORD 'postgres';

创建数据库实例
>postgres=# CREATE DATABASE db_safety_eye;

授权用户到实例上
>postgres=# GRANT ALL PRIVILEGES ON DATABASE db_safety_eye TO postgres;

修改用户密码
>postgres=# ALTER USER postgres PASSWORD 'postgres';

修改数据库实例编码
>postgres=# update pg_database set encoding = pg_char_to_encoding('UTF8') where datname = 'db_safety_eye';

查看数据库数据目录
>postgres=# show data_directory;

查看数据库版本
>postgres=# show server_version;
>postgres=#  select version();

查看哪些用户在链接数据库
>postgres=# select * from pg_stat_activity;

杀死指定进程(也可选其他参数)
>postgres=# select pg_terminate_backend(pid) FROM pg_stat_activity WHERE pid = '10069'
```
pg_cancel_backend       只是取消当前某一个进程的查询操作，但不能释放数据库连接
pg_terminate_backend    可以在pg的后台杀死这个进程，从而释放出宝贵的连接资源
```
