把已经安装了的服务添加为系统服务，可以执行以下命令：
```
chkconfig --add 服务名称                        #添加系统服务
chkconfig --del 服务名称                        #删除系统服务
chkconfig --level 启动级别 服务名 on            #表示开启自启动
chkconfig --level 启动级别 服务名 off           #表示关闭自启动
```
>[root@dwj Desktop]# chkconfig --level 3 mysql on    #让mysql服务在命令行模式，随系统启动

如果要查看哪些服务被添加为系统服务可以使用命令:
>[root@dwj Desktop]# chkconfig --list <br>
>[root@dwj Desktop]# ntsysv

如果要查看哪些程序被添加为自启动，可以使用命令:
>[root@dwj Desktop]# cat /etc/rc.local       #查看这个文件中添加了哪些程序路径

举例：把一个shell脚本添加为系统服务，并跟随系统启动：

在/etc/rc.d/init.d有很多文件，每个文件都是一些shell脚本。我们也可以写一个自己的脚本放在这里,
脚本文件的内容也很简单，类似于这个样子（例如起个名字叫做“dwj”）：

```sh
#chkconfig: 2345 10 90           （2345是启动级别；10是/etc/rc.d/rc5.d/S10network中启动编号，90是K90network关闭编号）
#description:start at boot time  （随便写入描述，只有添加这两行才能被chkconfig识别）
#source function library
. /etc/init.d/functions
start() {
        echo "Starting my process "
        cd /opt
        ./dwj.sh
}
stop() {
        killall dwj.sh
        echo "Stoped"
}
```
写了脚本文件之后事情还没有完，继续完成以下几个步骤：

>[root@dwj Desktop]# chmod +x dwj              #增加执行权限

>[root@dwj Desktop]# chkconfig --add dwj       #把 dwj 添加到系统服务列表

>[root@dwj Desktop]# chkconfig dwj on          #设定dwj的开关（on/off）

>[root@dwj Desktop]# chkconfig --list dwj      #就可以看到已经注册了dwj的服务
