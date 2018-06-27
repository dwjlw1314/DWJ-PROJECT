日志对任何一个OS、应用软件、服务进程而言都是必不可少的模块。日志文件对于系统和网络安全起到中大作用，同时具有审计、跟踪、排错功能。可以通过日志文件监测系统与网络安全隐患，以及监测黑客入侵攻击路线。

1.who 命令查询utmp文件并报告当前登录的每个用户,Who的缺省输出包括用户名、终端类型、登录日期及远程主机。使用该命令,系统管理员可以查看当前系统存在哪些不法用户，从而对其进行审计和处理。命令显示如下所示：
```
[root@dwj /]# who
root   tty/1    2016-07-11 19:22 (:0)
root   pts/0    2016-07-11 22:50 (:0.0)
root   pts/1    2016-07-11 18:50 (:0.0)
```
如果指明了wtmp文件名，则who命令查询所有以前的记录,命令who /var/log/wtmp将报告自从wtmp文件创建或删改以来的每一次登录。运行该命令如下所示：
```
[root@dwj /]# who /var/log/wtmp
root   tty/1    2016-07-11 19:22 (:0)
root   pts/0    2016-07-11 22:50 (:0.0)
root   tty/1    2016-07-06 18:22 (:0)
root   tty/1    2016-07-06 19:22 (:0)
root   pts/1    2016-07-11 18:50 (:0.0)
root   pts/0    2016-08-11 22:50 (:0.0)
root   tty/1    2016-08-07 18:22 (:0)
root   tty/1    2016-08-06 19:22 (:0)
root   pts/1    2016-07-26 18:50 (:0.0)
```
2.users 命令用单独的一行打印出当前登录的用户，每个显示的用户名对应一个登录会话。如果一个用户有不止一个登录会话，那他的用户名将显示相同的次数,运行该命令将如下所示:
```
[root@dwj eclipse]# users
root  root
```
3.last 命令往回搜索wtmp来显示自从文件第一次创建以来登录过的用户。系统管理员可以周期性地对这些用户的登录情况进行审计和考核，从而发现起中存在的问题,确定不法用户,并进行处理，可以通过指明用户来显示其登录信息即可，运行命令如下所示：
![image](https://github.com/dwjlw1314/DWJ-PROJECT/raw/master/PictureSource/4.19.1.png)

4.lastlog 命令在每次有用户登录时被查询。可以使用lastlog命令检查某特定用户上次登录的时间，格式化登录日志/var/log/lastlog的内容。它根据UID排序显示登录名、端口号（tty）和上次登录时间。如果一个用户从未登录过，lastlog显示**Never  logged**。注意需要以root身份运行该命令
```
[root@dwj eclipse]# lastlog -u root
Username    Port   From    Latest
root                       **Never logged in**
[root@dwj eclipse]# lastlog
root                       **Never logged in**
bin                        **Never logged in**
```
5.lastb 命令查看错误登录日志，查看的是/var/log/btmp文件
```
[root@dwj eclipse]# lastb
root     ssh:notty    192.168.8.123    Tue Jun 12 11:49 - 11:49  (00:00)    
root     ssh:notty    192.168.8.123    Tue Jun 12 11:38 - 11:38  (00:00)    
btmp begins Tue Jun 12 11:38:07 2018
```

>[root@dwj log]# vim /etc/rsyslog.conf     #rsyslog v5 configuration file

等级名称 |  说明
---|---
none   | 没有等级
debug  | 一般的调试信息说明
info   | 基本的通知信息
notice | 普通信息，但是有一定的重要性
warning| 警告信息，但是不影响到服务或系统运行
err    | 错误信息
crit   | 临界状态信息
alert  | 警告状态信息
emerg  | 疼痛等级，系统已经无法使用了

>[root@dwj log]# vim /etc/logrotate.conf   #日志轮替配置文件

参数 | 参数说明
---|---
daily   |  日志轮替周期是每天
weekly  |  日志轮替周期是每周
monthly |  日志轮替周期是每月
rotate  |  保留的日志文件个数，0指的没有备份
compress|  日志轮替时旧日志进行压缩
create mode owner group | 建立新日志，同时指定权限，用户和所属组
mail address | 当日止轮替时，输出内容发送到指定的邮箱
missingok |  如果日志不存在，则忽略该日志的警告信息
notifempty|  如果日志为空文件，则不进行日志轮替
minsize |  日志轮替的最小值，也就是日志到达这个最小值才轮替，否则就算时间到了也不轮替
size    |  日志只有大于指定大小才进行轮替，不按照时间进行
dateext |  使用日期作为轮替文件的后缀

>[root@dwj log]# logrotate -v /etc/logrotate.conf  #显示哪些日志需要轮替

>[root@dwj log]# logrotate -f /etc/logrotate.conf  #强制执行日志轮替
