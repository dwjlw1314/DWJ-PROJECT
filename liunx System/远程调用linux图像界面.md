一、软件环境
```
1. SecureCRT 8.1
2. XmanagerPowerSuite-6.0.0029r.exe
3. Xmanager-keygen-master（破解工具）
```

二、安装步骤
1. windows安装Xmanager

下载地址：https://cdn.netsarang.net/45492b25/XmanagerPowerSuite-6.0.0029r.exe
下载完安装好即可

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
