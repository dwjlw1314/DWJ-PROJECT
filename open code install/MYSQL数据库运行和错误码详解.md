#启动和停止mySQL服务器
>[root@dwj ~]# /etc/init.d/mysqld start/stop  || 简化启动命令 service mysqld start/stop

#启动mySQL服务器
>[root@dwj ~]# /usr/bin/mysqld_safe &                                                       

#停止mySQL服务器
>[root@dwj ~]# /usr/bin/mysqladmin shutdown

#本地登录mysql数据库
>[root@dwj ~]# mysql -u用户名 -p用户密码

#登录远程mysql数据库
>[root@dwj ~]# mysql -u用户名 –h主机名或者IP地址 -p用户密码

#管理员登录
>[root@dwj ~]# mysql -u root mysql

#增加新用户
>mysql> grant 权限 on 数据库.* to 用户名@登录主机 identified by "密码"

增加一个用户user密码为123456，让其可以在本机上登录，并赋予数据库所有功能的权限。然后键入以下命令：
>mysql> grant all privileges on *.* to user@localhost Identified by "123456";   <br>
如果希望该用户能够在任何机器上登陆mysql，则将localhost改为"%"；如果你不想user有密码，可以把密码填写成空

#更新root密码，最新mysql采用如下SQL
>mysql> UPDATE user SET Password=PASSWORD('newpassword') where USER='root';   <br>
>mysql> UPDATE user SET authentication_string=PASSWORD('newpassword') where USER='root';

#显示数据库列表，缺省有两个数据库：mysql和test
>mysql> show databases;                                                                 

#删除用户
>mysql> delete from user where user='test' and Host='localhost';                        

#刷新权限
>mysql> flush privileges;                                                               

#删除用户数据库
>mysql> drop database zabbix;                                                           

<font color=#FF0000 size=5> <p align="center">ERROR CODE</p></font>

ERROR 1045 (28000): Access denied for user 'root'@'localhost' (using password: YES)
```
原因：数据库中存在空用户所致
解决方法：
1、停用mysql服务：
[root@dwj ~]# service mysql stop
2、输入命令：
[root@dwj ~]# mysqld_safe --user=mysql --skip-grant-tables --skip-networking &
3、登入数据库：
[root@dwj ~]# mysql -u root mysql
4、mysql> use mysql;
5、mysql> select user,host,password from user;
新版本使用如下语句 mysql> select user,host,password_lifetime from user;
结果如下：
+------+-----------+----------+
| user | host      | password |
+------+-----------+----------+
| root | localhost  |         |
| root | dwj        |         |
| root | 127.0.0.1  |         |
|      | localhost  |         |
|      | dwj        |         |
+------+-----------+----------+
6、将上面查询出来的空用户删除：
mysql> delete from user where user='';
7、退出数据库：
mysql> quit
8、启动mysql服务：
[root@dwj ~]# service mysqld start
9、重新用命令：
[root@dwj ~]# mysql -u root -p   //ok
```
ERROR 1820 (HY000): You must reset your password using ALTER USER statement before executing this statement
```
原因：新版本验证加强
解决方法：
1.mysql> set password=password('123456');
    Query OK, 0 rows affected (0.03 sec)
```
