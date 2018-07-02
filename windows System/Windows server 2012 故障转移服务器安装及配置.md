一、准备阶段
```
群集节点A：
计算机名：WIN-A
Public IP地址：192.168.100.1
Heart  IP地址：10.10.10.10
群集节点B：
计算机名：WIN-B
Public IP地址：192.168.100.2
Heart  IP地址：10.10.10.20
存储服务器：
计算机名：FileServer
IP地址：192.168.100.10

10.10.10.30 集群存储
192.168.100.3 集群虚拟IP
```
二、前提条件
```
节点A,B加入AD域并用域账户登录
节点A，B均不能作为域控制器
节点A，B都可以访问存储
节点A，B的补丁最好一致
关闭防火墙
```
三、配置存储服务器 <br>
1.进入服务器管理器，点击File and Storage Services
![image](https://github.com/dwjlw1314/DWJ-PROJECT/raw/master/PictureSource/nlb_Failover/2.7.1.png)

2.点击iSCSI，点击To install iSCSI Target Server,start the Add Roles and Features Wizard
![image](https://github.com/dwjlw1314/DWJ-PROJECT/raw/master/PictureSource/nlb_Failover/2.7.2.png)

3.点击下一步（Next）
![image](https://github.com/dwjlw1314/DWJ-PROJECT/raw/master/PictureSource/nlb_Failover/2.7.3.png)

4.点击下一步（Next）
![image](https://github.com/dwjlw1314/DWJ-PROJECT/raw/master/PictureSource/nlb_Failover/2.7.4.png)

5.点击安装（Install）
![image](https://github.com/dwjlw1314/DWJ-PROJECT/raw/master/PictureSource/nlb_Failover/2.7.5.png)

6.安装完成。点击TASKS，再点击New iSCSI Virtual Disk
![image](https://github.com/dwjlw1314/DWJ-PROJECT/raw/master/PictureSource/nlb_Failover/2.7.6.png)

7.选择创建位置，下一步
![image](https://github.com/dwjlw1314/DWJ-PROJECT/raw/master/PictureSource/nlb_Failover/2.7.7.png)

8.输入iSCSI磁盘名称
![image](https://github.com/dwjlw1314/DWJ-PROJECT/raw/master/PictureSource/nlb_Failover/2.7.8.png)

9.输入预分配磁盘大小，下一步
![image](https://github.com/dwjlw1314/DWJ-PROJECT/raw/master/PictureSource/nlb_Failover/2.7.9.png)

10.新建iSCSI目标，下一步
![image](https://github.com/dwjlw1314/DWJ-PROJECT/raw/master/PictureSource/nlb_Failover/2.7.10.png)

11.输入目标名称
![image](https://github.com/dwjlw1314/DWJ-PROJECT/raw/master/PictureSource/nlb_Failover/2.7.11.png)

12.添加访问规则
![image](https://github.com/dwjlw1314/DWJ-PROJECT/raw/master/PictureSource/nlb_Failover/2.7.12.png)

13.点击下一步
![image](https://github.com/dwjlw1314/DWJ-PROJECT/raw/master/PictureSource/nlb_Failover/2.7.13.png)

14.点击下一步
![image](https://github.com/dwjlw1314/DWJ-PROJECT/raw/master/PictureSource/nlb_Failover/2.7.14.png)

15.点击创建
![image](https://github.com/dwjlw1314/DWJ-PROJECT/raw/master/PictureSource/nlb_Failover/2.7.15.png)

16.安装完成，关闭。（以上为仲裁盘创建，重复6-16步操作，创建一个共享磁盘Sharedisk）
![image](https://github.com/dwjlw1314/DWJ-PROJECT/raw/master/PictureSource/nlb_Failover/2.7.16.png)

17.配置节点A，使其能够访问iSCSI目标。点击工具，选择iSCSI Initistor
![image](https://github.com/dwjlw1314/DWJ-PROJECT/raw/master/PictureSource/nlb_Failover/2.7.17.png)

18.点击Yes
![image](https://github.com/dwjlw1314/DWJ-PROJECT/raw/master/PictureSource/nlb_Failover/2.7.18.png)

19.点击Discovery,再点击Discover Portal
![image](https://github.com/dwjlw1314/DWJ-PROJECT/raw/master/PictureSource/nlb_Failover/2.7.19.png)

20.输入存储服务器IP地址，点击OK
![image](https://github.com/dwjlw1314/DWJ-PROJECT/raw/master/PictureSource/nlb_Failover/2.7.20.png)

21.点击Targets，选择一个目标点击连接
![image](https://github.com/dwjlw1314/DWJ-PROJECT/raw/master/PictureSource/nlb_Failover/2.7.21.png)

22.默认点击OK（需将两个目标都连接上
![image](https://github.com/dwjlw1314/DWJ-PROJECT/raw/master/PictureSource/nlb_Failover/2.7.22.png)

23.打开计算机管理器，右键磁盘1选择联机
![image](https://github.com/dwjlw1314/DWJ-PROJECT/raw/master/PictureSource/nlb_Failover/2.7.23.png)

24.右键磁盘1选择Initialize Disk
![image](https://github.com/dwjlw1314/DWJ-PROJECT/raw/master/PictureSource/nlb_Failover/2.7.24.png)

25.右击右侧方框区域选择创建新简单卷
![image](https://github.com/dwjlw1314/DWJ-PROJECT/raw/master/PictureSource/nlb_Failover/2.7.25.png)

26.创建完两个磁盘后如图所示。（节点B只需要将两个磁盘联机，然后修改盘符，两节点所设置的磁盘盘符需一致
![image](https://github.com/dwjlw1314/DWJ-PROJECT/raw/master/PictureSource/nlb_Failover/2.7.26.png)

安装和配置故障转移群集 <br>
27.添加角色和功能
![image](https://github.com/dwjlw1314/DWJ-PROJECT/raw/master/PictureSource/nlb_Failover/2.7.27.png)

28.点击下一步
![image](https://github.com/dwjlw1314/DWJ-PROJECT/raw/master/PictureSource/nlb_Failover/2.7.28.png)

29.点击下一步
![image](https://github.com/dwjlw1314/DWJ-PROJECT/raw/master/PictureSource/nlb_Failover/2.7.29.png)

30.默认设置，点击下一步
![image](https://github.com/dwjlw1314/DWJ-PROJECT/raw/master/PictureSource/nlb_Failover/2.7.30.png)

31.点击下一步
![image](https://github.com/dwjlw1314/DWJ-PROJECT/raw/master/PictureSource/nlb_Failover/2.7.31.png)

32.选择故障转移群集，下一步
![image](https://github.com/dwjlw1314/DWJ-PROJECT/raw/master/PictureSource/nlb_Failover/2.7.32.png)

33.点击安装
![image](https://github.com/dwjlw1314/DWJ-PROJECT/raw/master/PictureSource/nlb_Failover/2.7.33.png)

34.安装完成，关闭。（A，B节点均要安装故障转移群集功能）
![image](https://github.com/dwjlw1314/DWJ-PROJECT/raw/master/PictureSource/nlb_Failover/2.7.34.png)

35.选择Tools---Failover Cluster Manager
![image](https://github.com/dwjlw1314/DWJ-PROJECT/raw/master/PictureSource/nlb_Failover/2.7.35.png)

36.点击创建群集
![image](https://github.com/dwjlw1314/DWJ-PROJECT/raw/master/PictureSource/nlb_Failover/2.7.36.png)

37.点击下一步
![image](https://github.com/dwjlw1314/DWJ-PROJECT/raw/master/PictureSource/nlb_Failover/2.7.37.png)

38.点击Browse
![image](https://github.com/dwjlw1314/DWJ-PROJECT/raw/master/PictureSource/nlb_Failover/2.7.38.png)

39.点击Advanced
![image](https://github.com/dwjlw1314/DWJ-PROJECT/raw/master/PictureSource/nlb_Failover/2.7.39.png)

40.点击Find Now，选择两台节点服务器点击OK
![image](https://github.com/dwjlw1314/DWJ-PROJECT/raw/master/PictureSource/nlb_Failover/2.7.40.png)

41.点击OK
![image](https://github.com/dwjlw1314/DWJ-PROJECT/raw/master/PictureSource/nlb_Failover/2.7.41.png)

42.点击下一步
![image](https://github.com/dwjlw1314/DWJ-PROJECT/raw/master/PictureSource/nlb_Failover/2.7.42.png)

43.默认设置，点击下一步
![image](https://github.com/dwjlw1314/DWJ-PROJECT/raw/master/PictureSource/nlb_Failover/2.7.43.png)

44.点击下一步
![image](https://github.com/dwjlw1314/DWJ-PROJECT/raw/master/PictureSource/nlb_Failover/2.7.44.png)

45.默认设置，点击下一步
![image](https://github.com/dwjlw1314/DWJ-PROJECT/raw/master/PictureSource/nlb_Failover/2.7.45.png)

46.点击下一步
![image](https://github.com/dwjlw1314/DWJ-PROJECT/raw/master/PictureSource/nlb_Failover/2.7.46.png)

47.验证完成，点击关闭
![image](https://github.com/dwjlw1314/DWJ-PROJECT/raw/master/PictureSource/nlb_Failover/2.7.47.png)

48.输入群集名称及群集IP。（只选择Public网段设置即可，心跳IP不用管）
![image](https://github.com/dwjlw1314/DWJ-PROJECT/raw/master/PictureSource/nlb_Failover/2.7.48.png)

49.安装完成，点击下一步
![image](https://github.com/dwjlw1314/DWJ-PROJECT/raw/master/PictureSource/nlb_Failover/2.7.49.png)

50.创建完成关闭即可
![image](https://github.com/dwjlw1314/DWJ-PROJECT/raw/master/PictureSource/nlb_Failover/2.7.50.png)

如需自启动程序，可添加如下脚本和相应批处理来解决: (autostart.vbs)
```vbs
Option Explicit
Dim wmi,proc,procs,proname,flag,WshShell
Do
proname="运行文件名"
Set wmi = getobject("winmgmts:{impersonationlevel = impersonate}!\\.\root\cimv2")
Set procs=wmi.execquery("select * from win32_process")
flag=True
For Each proc in procs
If StrComp(proc.name,proname) = 0  Then  
flag=False
Exit For
End If
Next
Set wmi=nothing
If flag Then
Set WshShell = Wscript.CreateObject("Wscript.Shell")
WshShell.Run("要执行的文件路径")
End If
wscript.sleep 10000
loop
```
