<font color=#FF0000 size=5> <p align="center">性能相关查询</p></font>



<font color=#FF0000 size=5> <p align="center">常用语句</p></font>

查看哪些用户在链接数据库
>postgres=# select * from pg_stat_activity;

杀死指定进程(也可选其他参数)
>postgres=# select pg_terminate_backend(pid) FROM pg_stat_activity WHERE pid = '10069'
```
pg_cancel_backend       只是取消当前某一个进程的查询操作，但不能释放数据库连接
pg_terminate_backend    可以在pg的后台杀死这个进程，从而释放出宝贵的连接资源
```
