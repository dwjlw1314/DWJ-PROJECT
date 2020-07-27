一、软件环境
```
1. SecureCRT 8.1
2. XmanagerPowerSuite-6.0.0029r.exe
3. Xmanager-keygen-master（破解工具）
```

二、安装步骤
1. windows安装Xmanager

下载地址：https://cdn.netsarang.net/45492b25/XmanagerPowerSuite-6.0.0029r.exe
下载完安装好即可，这里选择带r的可添加注册码版本，官网链接是没有的

2. windows安装SecureCRT

启动SecureCRT，进入Options->Session Options->Remote/X11 选中Forword X11 Packets ->OK

![image](https://github.com/dwjlw1314/DWJ-PROJECT/raw/master/PictureSource/1.18.1.jpg)

>[root@localhost dwj]# vim /etc/ssh/sshd_config  #修改 X11Forwarding yes

![image](https://github.com/dwjlw1314/DWJ-PROJECT/raw/master/PictureSource/1.18.2.jpg)

重启sshd服务
>[root@localhost dwj]# service sshd restart

在windows上运行Xmanager - Passive(xshell无需运行该命令)

之后使用SecureCRT以root用户远程登陆执行命令如下：
>[root@localhost dwj]# xhost +  <br>
返回信息为： access control disabled,clients can connect from any host

查看远程目标机器显示效果,如果能出现信息说明已经成功了
>[root@localhost dwj]# xdpyinfo

查看远程目标机器name of display的字符串
>[root@localhost dwj]# echo $DISPLAY  #显示localhost:10.0

如果 xhost + 不能执行，可以输入以下命令:
>[root@localhost dwj]# export DISPLAY=:0.0  # 0.0具体根据上一条命令修改

安装xclock程序，运行验证环境是否配置完成
>[root@localhost dwj]# sudo apt-get install xclock  <br>
>[root@localhost dwj]# ./xclock

<font color=#FF00>其他用户切换后无法调用图形界面</font>
>[root@localhost dwj]# xdpyinfo |head -1
```
//显示内容如下，使用 10.0
name of display:    localhost:10.0
```

查看root用户下MAGIC-COOKIE
>[root@localhost dwj]# xauth list |grep unix:10
```
localhost.localdomain/unix:10  MIT-MAGIC-COOKIE-1  0d7b632fb5ed3462d6056aac6fbb73bc
```

切换到oracle用户下
>[root@localhost dwj]# su - oracle

查看oracle用户下MAGIC-COOKIE, 显示内容动态变化
>[root@localhost dwj]# xauth list
```
localhost.localdomain/unix:4  MIT-MAGIC-COOKIE-1  157020453af1eeda62cb895ad91bcdbe
```

添加root用户下的认证文件
>[root@localhost dwj]# xauth add localhost.localdomain/unix:10  MIT-MAGIC-COOKIE-1  b9b7993fca0ed9f32db786139359def8

导入环境路径
>[root@localhost dwj]# export PATH=/usr/sbin:$PATH:$ORACLE_HOME/bin

导入DISPLAPY路径，其中10.0 是 xdpyinfo 中获取的
>[root@localhost dwj]# export DISPLAY=:10.0

运行图形界面测试程序
>[root@localhost dwj]# ./xclock
