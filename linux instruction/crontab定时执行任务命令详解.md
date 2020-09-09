一、cron服务 -- anacron命令可以解决cron错过时间后，不执行cron计划任务的问题

cron是一个linux下 的定时执行工具，可以在无需人工干预的情况下运行作业
```
[root@dwj ~]# service crond start      #启动服务
[root@dwj ~]# service crond stop       #关闭服务
[root@dwj ~]# service crond restart    #重启服务
[root@dwj ~]# service crond reload     #重新载入配置
[root@dwj ~]# service crond status     #查看服务状态
```
二、cron在3个地方查找配置文件

/var/spool/cron/ 这个目录下存放的是每个用户的crontab任务，每个任务以创建者的名字命名，比如root建的crontab任务对应的文件就是/var/spool/cron/root。一般一个用户最多只有一个crontab文件

三、/etc/crontab 这个文件负责安排由系统管理员制定的维护系统以及其他任务的crontab
```
SHELL=/bin/bash
PATH=/sbin:/bin:/usr/sbin:/usr/bin
MAILTO=root

# For details see man 4 crontabs

# Example of job definition:
# .---------------- minute (0 - 59)
# |  .------------- hour (0 - 23)
# |  |  .---------- day of month (1 - 31)
# |  |  |  .------- month (1 - 12) OR jan,feb,mar,apr ...
# |  |  |  |  .---- day of week (0 - 6) (Sunday=0 or 7) OR sun,mon,tue,wed,thu,fri,sat
# |  |  |  |  |
# *  *  *  *  * user-name  command to be executed
MAILTO=root：是说，当 /etc/crontab 这个档案中的例行性命令发生错误时，会将错误讯息传给指定用户，因此，我通常改成自己的e-mail
01 * * * * root run-parts /etc/cron.hourly：五个数字后面接的是root，代表的是执行的级别为root身份，当然也可以改为成其他的身份
每个小时的01分，系统会以root身份去/etc/cron.hourly这个目录下执行所有可执行的文件，可以将每天需要执行的命令直接写/etc/cron.daily
```
四、/etc/cron.d/ 这个目录用来存放任何要执行的crontab文件或脚本

五、创建cron脚本(方法一)
```
第一步：写cron脚本文件,命名为crontest.cron
15,30,45,59 * * * * echo "xgmtest....." >> xgmtest.txt 表示，每隔15分钟，执行打印一次命令
第二步：添加定时任务。执行命令 “crontab crontest.cron”搞定
第三步："crontab -l" 查看定时任务是否成功或者检测/var/spool/cron下是否生成对应cron脚本
注意：这操作是直接替换该用户下的crontab，而不是新增
```
六、crontab用法

crontab命令用于安装、删除或者列出用于驱动cron后台进程的表格。用户把需要执行的命令序列放到crontab文件中以获得执行
每个用户都可以有自己的crontab文件。/var/spool/cron下的crontab文件不可以直接创建或者直接修改
>[root@dwj /var/spool/cron]# crontab -e      #创建/var/spool/cron下的crontab文件

>[root@dwj /var/spool/cron]# crontab -e -u oracle   #创建oracle用户的crontab文件

在crontab文件中每行都包括六个域，其中前五个是指定被执行的时间，最后一个是要被执行的命令,格式如下：
```
minute hour day-of-month month-of-year day-of-week commands
合法值 00-59 00-23 01-31 01-12 0-6 (0 is sunday)
除了数字还有几个个特殊的符号就是"*"、"/"和"-"、","
  * 代表所有的取值范围内的数字
  / 代表每个单位的意思,"/5"表示每5个单位
  - 代表从某个数字到某个数字
  , 分开几个离散的数字

  -l 在标准输出上显示当前的crontab
  -r 删除当前的crontab文件
　-e 使用VISUAL或者EDITOR环境变量所指的编辑器编辑当前的crontab文件。当结束编辑离开时，编辑后的文件将自动安装
```
八、例子(适用于/etc/crontab的编辑方式)
每小时执行/etc/cron.hourly内的脚本
>01 * * * * root run-parts /etc/cron.hourly

每天执行/etc/cron.daily内的脚本
>02 4 * * * root run-parts /etc/cron.daily

每星期执行/etc/cron.weekly内的脚本
>22 4 * * 0 root run-parts /etc/cron.weekly

每小时执行/etc/cron.hourly内的脚本
>01 * * * * root run-parts /etc/cron.hourly

每天执行/etc/cron.daily内的脚本
>02 4 * * * root run-parts /etc/cron.daily

每星期执行/etc/cron.weekly内的脚本
>22 4 * * 0 root run-parts /etc/cron.weekly

每月去执行/etc/cron.monthly内的脚本
>42 4 1 * * root run-parts /etc/cron.monthly   <br>
注意: "run-parts"这个参数了，如果去掉这个参数的话，后面就可以写要运行的某个脚本名，而不是文件夹名　

<font color=#FF0000 size=5> <p align="center">at用法</p></font>

at命令的使用顺序如下：先用at命令后接想要程序执行的确定时刻,再输入你想要在以上指定时刻执行的命令

例如：
```
[root@dwj opt]# at 10:59
at> touch /opt/gjsy-death
```
```
at -l       #查看设置的定时任务
atrm num    #删除定时任务
```
