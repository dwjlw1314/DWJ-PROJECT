/etc/init.d/mysqld start/stop  || 简化启动命令 service mysqld start/stop         #启动和停止mySQL服务器

/usr/bin/mysqld_safe &                                                          #启动mySQL服务器

/usr/bin/mysqladmin shutdown                                                    #停止mySQL服务器

mysql -u用户名 -p用户密码                                                        #本地登录mysql数据库

mysql -u用户名 –h主机名或者IP地址 -p用户密码                                      #登录远程mysql数据库

mysql -u root mysql                                                             #管理员登录

grant 权限 on 数据库.* to 用户名@登录主机 identified by "密码"                    #增加新用户

增加一个用户user密码为123456，让其可以在本机上登录，并赋予数据库所有功能的权限。然后键入以下命令：  <br>
grant all privileges on *.* to user@localhost Identified by "123456";  <br>
如果希望该用户能够在任何机器上登陆mysql，则将localhost改为"%"；如果你不想user有密码，可以把密码填写成空

UPDATE user SET Password=PASSWORD('newpassword') where USER='root';             #更新root密码，最新mysql采用如下SQL  <br>
UPDATE user SET authentication_string=PASSWORD('newpassword') where USER='root';

show databases;                                                                 #显示数据库列表，缺省有两个数据库：mysql和test

delete from user where user='test' and Host='localhost';                        #删除用户

flush privileges;                                                               #刷新权限

drop database zabbix;                                                           #删除用户数据库

<font color=#FF0000 size=4> <p align="center">ERROR CODE</p></font>

~~ERROR 1045 (28000): Access denied for user 'root'@'localhost' (using password: YES) <br>
原因：数据库中存在空用户所致  <br>
解决方法： <br>
1、停用mysql服务： <br>
[root@dwj ~]# service mysql stop  <br>
2、输入命令：    <br>
[root@dwj ~]# mysqld_safe --user=mysql --skip-grant-tables --skip-networking &   <br>
3、登入数据库：   <br>
[root@dwj ~]# mysql -u root mysql <br>
4、mysql> use mysql;  <br>
5、mysql> select user,host,password from user; 新版本使用如下语句 mysql> select user,host,password_lifetime from user;

结果如下：
```
+------+-----------+----------+
| user | host      | password |
+------+-----------+----------+
| root | localhost  |         |
| root | dwj        |         |
| root | 127.0.0.1  |         |
|      | localhost  |         |
|      | dwj        |         |
+------+-----------+----------+
5 rows in set (0.00 sec)
```
6、将上面查询出来的空用户删除： <br>
mysql> delete from user where user=''; <br>
7、退出数据库：  <br>
mysql> quit <br>
8、启动mysql服务： <br>
[root@dwj ~]# service mysqld start
9、重新用命令：  <br>
[root@dwj ~]# mysql -u root -p   //ok

~~ERROR 1820 (HY000): You must reset your password using ALTER USER statement before executing this statement  <br>
原因：新版本验证加强 <br>
解决方法： <br>
1.mysql> set password=password('123456'); <br>
    Query OK, 0 rows affected (0.03 sec)
