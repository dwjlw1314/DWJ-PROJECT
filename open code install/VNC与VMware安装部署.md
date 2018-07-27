------ rhel-server-6.4-x86_64 系统 --------

vnc:虚拟网络计算机的缩写，是一种linux系统下常用的图形化远程管理工具

前提： VNC软件包，VMware服务端
```
eg：vnc-server-4.1.2-14.el5_5.4.x86_64;
    VNCViewerPlu安装包，windows显示客户端
    VMware-Workstation-Full-11.0.0-2305329.x86_64.bundle
```
安装过程：

1.首先，检查系统是否安装过vnc服务
>[root@diwj Desktop]# rpm –q vnc-server <br>
#若出现package vnc-server is not installed,说明没有安装；

2.root用户执行如下操作
>[root@diwj Desktop]# rpm –ivh vnc-server-4.1.2-14.el5_5.4.x86_64.rpm  <br>
>-i：安装rpm包；-vh：显示安装进度

3.启动vncserver，如图提示启动成功
>[root@diwj Desktop]# vncserver <br>
![image](https://github.com/dwjlw1314/DWJ-PROJECT/raw/master/PictureSource/3.3.1.png)

4.开启图形化功能，编辑xstartup，去掉图中#，添加一行
>[root@diwj Desktop]# vim /root/.vnc/xstartup
></root/>：代表当前用户目录
![image](https://github.com/dwjlw1314/DWJ-PROJECT/raw/master/PictureSource/3.3.2.png)

5.配置vnc服务自启动
>[root@diwj Desktop]# setup  <br>
选中system services -> vncserver保存退出

6.端口自启动，去掉方框中的#号，将2:myusername改为1:root
>[root@diwj Desktop]# vim /etc/sysconfig/vncservers

注意事项： <br>
如果/root/.vnc目录下生成 dwj:2.log 日志，说明出现问题，需要杀掉进程Xvnc，删除/tmp/.X11-unix/X1

                  VMware-Workstation
运行方式：APPlications-system tools-vmware-workstation
安装完成后日志目录在 /tmp/vmware-root/
程序安装路径在 /usr/lib/vmware/
