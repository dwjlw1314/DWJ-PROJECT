------ rhel-server-6.4-x86_64 系统 --------

<font color=#FF0000 size=5>一、PostgreSQL简介</font>

PostgreSQL是以加州大学伯克利分校计算机系开发的 POSTGRES 版本 4.2 为基础的对象关系型数据库管理系统（ORDBMS）,简称pgsql,它支持大部分 SQL 标准并且提供了许多其他现代特性：复杂查询 外键 触发器 视图 事务完整性 多版本并发控制 同样，PostgreSQL 可以用许多方法扩展
比如， 通过增加新的：数据类型 函数 操作符 聚集函数 索引方法 过程语言

<font color=#FF0000 size=5>二、 PostgreSQL安装</font>

1.安装依赖包，某些系统只需要部分安装
>[root@dwj ~]# yum -y install perl-ExtUtils-Embed readline-devel zlib-devel pam-devel

>[root@dwj ~]# yum -y install libxml2-devel libxslt-devel openldap-devel python-devel openssl-devel cmake

2.解压编译源码
>[root@dwj ~]# mkdir /usr/local/pgsql

>[root@dwj ~]# tar -zxvf postgresql-9.4.1.tar.gz

>[root@dwj ~]# cd postgresql-9.4.1

>[root@dwj postgresql-9.4.1]# ./configure --prefix=/usr/local/pgsql --with-perl --with-python --with-libxml --with-libxslt

>[root@dwj postgresql-9.4.1]# gmake && gmake install

3.安装PG插件
>[root@dwj postgresql-9.4.1]# cd contrib/

>[root@dwj contrib]# gmake

>[root@dwj contrib]# gmake install

>[root@dwj contrib]# echo "/usr/local/pgsql/lib" >> /etc/ld.so.conf.d/pgsql.conf

>[root@dwj contrib]# ldconfig

4.创建用户postgres
>[root@dwj contrib]# useradd postgres

>[root@dwj contrib]# echo "postgres"|passwd --stdin postgres

5.创建PG数据目录，初始化数据库
>[root@dwj contrib]# mkdir -p /data/pg/data

>[root@dwj contrib]# chown -R postgres:postgres /data/pg

>[postgres@dwj contrib]$ /usr/local/pgsql/bin/initdb --no-locale -U postgres -E utf8 -D /data/pg/data/ -W
  （切换到postgres用户下运行，在初始化的时候，看提示添加超级用户的密码）

```
  参数备注:
  initdb [选项]... [DATADIR]
  -A, --auth=METHOD             本地连接的默认认证方法
  -D, --pgdata=DATADIR          当前数据库簇的位置
  -E, --encoding=ENCODING       为新数据库设置默认编码
  --locale=LOCALE               为新数据库设置默认语言环境
  --lc-collate, --lc-ctype, --lc-messages=LOCALE
  --lc-monetary, --lc-numeric, --lc-time=LOCALE
  为新的数据库簇在各自的目录中分别设定缺省语言环境（默认使用环境变量)
  --no-locale                   等同于 --locale=C
  --pwfile=文件名               对于新的超级用户从文件读取口令
  -T, --text-search-config=CFG  缺省的文本搜索配置
  -U, --username=NAME           数据库超级用户名
  -W, --pwprompt                对于新的超级用户提示输入口令
  -X, --xlogdir=XLOGDIR         当前事务日志目录的位置

  非普通使用选项:
  -d, --debug                   产生大量的除错信息
  -L DIRECTORY                  输入文件的位置
  -n, --noclean                 出错后不清理
  -s, --show                    显示内部设置

   其它选项:
   -?, --help                   显示此帮助, 然后退出
   -V, --version                输出版本信息, 然后退出
  ```
如果没有指定数据目录, 将使用环境变量 PGDATA

6.配置运行环境变量（方便管理） 切换到root用户
>[root@dwj contrib]# vim /etc/profile

添加以下代码：
```config
  PGDATA=/data/pg/data
  PGHOST=127.0.0.1
  PGDATABASE=postgres
  PGUSER=postgres
  PGPORT=5432
  PATH=/usr/local/pgsql/bin:$PATH
  export PATH
  export PGDATA PGHOST PGDATABASE PGUSER PGPORT
```
执行生效
>[root@dwj contrib]# source /etc/profile

7.postgresql服务管理
>[root@dwj contrib]# su postgres

启动：
>[postgres@dwj contrib]$ pg_ctl start -D /data/pg/data

重启：
>[postgres@dwj contrib]$ pg_ctl restart -D /data/pg/data

停止：
>[postgres@dwj contrib]$ pg_ctl stop -D /data/pg/data

强制重启：
>[postgres@dwj contrib]$ pg_ctl restart -D /data/pg/data -m f

强制停止：
>[postgres@dwj contrib]$ pg_ctl stop -D /data/pg/data -m f     #-m f 指定快速关闭

加载配置：
>[postgres@dwj contrib]$ pg_ctl reload -D  /data/pg/data

显示服务状态：
>[postgres@dwj contrib]$ pg_ctl status -D  /data/pg/data

连接数据库：
>[postgres@dwj contrib]$ psql -h 127.0.0.1 -U postgres -p 5432 -d postgres -W

-d 指定数据库 ,-W 输入密码 , -U 指定用户,-p 指定端口,-h 指定IP

8.设置自启动

复制PostgreSQL执行脚本
>[root@dwj contrib]# cp start-scripts/linux /etc/init.d/postgresql   <br>
[root@dwj contrib]# chmod +x /etc/init.d/postgresql

修改/etc/init.d/postgresql,把PGDATA改成PGDATA=/data/pg/data
>[root@dwj contrib]# chkconfig postgresql on

管理PG服务时也可以直接用下面启动脚本
```
启动：service postgresql start
停止：service postgresql stop
重启：service postgresql restart
加载：service postgresql reload
状态：serivce postgresql status
```
9.使用 pgadmin3 管理即可

<font color=#FF0000 size=5> <p align="center">Error</p></font>

错误信息：

	could not connect to server: Connection refused Is the server running on host "localhost"
  and accepting TCP/IP connections on port 5432?

解决方案：

一、在postgresql的安装文件夹/data/pg/data/pg_hba.conf里面
```
找到#IPv4 local connections:
在它上面添加 local pgsql all trust
在它下面的 host all all 127.0.0.1/32 trust
下面添加一行,内容为 host all all 192.168.91.1/24 trust
```
注：/32与/24,用32表示该IP被固定,用24表示前3位固定,该网段计算机就可以访问postgresql数据库

二、/data/pg/data/postgresql.conf文件中

找到#listen_addresses = 'localhost'; 把它改成 listen_addresses = '\*',这样,postgresql就可以监听所有ip地址的连接

三、重启postgresql服务.如果系统启用了防火墙,请先关闭

对了,如果要使用pgadmin连接远程的数据库服务器,须在SSL的选项中选择允许

错误信息： <br>
![image](https://github.com/dwjlw1314/DWJ-PROJECT/raw/master/PictureSource/3.5.1.png)

解决方案：

在postgresql用户下，打开 .bash_profile文件，在最后添加：export PS1='[\u@\h \W]\$' 保存文件退出
![image](https://github.com/dwjlw1314/DWJ-PROJECT/raw/master/PictureSource/3.5.2.png)
