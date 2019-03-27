#win10 系统激活命令

    C:\WINDOWS\system32>slmgr -upk (清空原有密钥)
    C:\WINDOWS\system32>slmgr -ipk VK7JG-NPHTM-C97JM-9MPGT-3V66T
    C:\WINDOWS\system32>slmgr /skms kms.xspace.in
    C:\WINDOWS\system32>slmgr -ato
    激活以后，打开运行，然后这里我们输入命令slmgr.vbs -xpr查看激活的状态
```
win+R可以运行命令：
credwiz                   #备份或还原储存的用户名和密码
cleanmgr                  #磁盘清理
compmgmt.msc              #计算机管理
charmap                   #启动字符映射表
calc                      #启动计算器
chkdsk.exe                #Chkdsk磁盘检查
dvdplay                   #DVD播放器
diskmgmt.msc              #磁盘管理实用程序
devmgmt.msc               #设备管理器
dcomcnfg                  #打开系统组件服务
dxdiag                    #检查DirectX信息
explorer                  #打开文件资源管理器
eventvwr                  #事件查看器
eudcedit                  #造字程序
fsmgmt.msc                #共享文件夹管理器
gpedit.msc                #组策略
iexpress                  #木马捆绑工具，系统自带
logoff                    #注销命令
lusrmgr.msc               #本机用户和组
mplayer2 / dvdplay        #简易widnows media player
mspaint                   #画图板
mstsc                     #远程桌面连接
magnify                   #放大镜实用程序
msconfig.exe              #系统配置实用程序
mmc                       #打开控制台
mobsync                   #同步命令
ncpa.cpl                  #网络管理界面
notepad                   #打开记事本
narrator                  #屏幕“讲述人”
netstat -an               #(cmd)网络接口检查命令
nslookup                  #IP地址侦测器/网络管理的工具向导
osk                       #打开屏幕键盘
odbcad32                  #ODBC数据源管理器
PowerShell                #提供强大远程处理能力
psr                       #问题步骤记录器
perfmon.msc               #计算机性能监测程序
regsvr32 /u *.dll         #停止dll文件运行
regedt32 / regedit        #注册表编辑器
rsop.msc  / gpedit.msc    #组策略结果集
sysdm.cpl                 #高级系统属性
systeminfo                #查看操作系统、产品ID、处理器、内存、BIOS、虚拟内存、补丁安装和网卡连接等详细的报告
> systeminfo > c:＼sysInfo.txt #结果从定向到指定文件
start c:＼sysInfo.txt     #打开c:＼sysInfo.txt文件查看
slui                      #查看windows系统激活信息
sdclt                     #备份状态与配置，查看系统是否已备份
services.msc              #本地服务设置
sigverif                  #文件签名验证程序
shrpubw                   #创建共享文件夹
secpol.msc                #本地安全策略
syskey                    #系统加密，一旦加密就不能解开，保护windows xp系统的双重密码
taskmgr                   #任务管理器
utilman                   #辅助工具管理器
wf.msc                    #高级安全Windows防火墙
wmimgmt.msc               #打开windows管理体系结构(WMI)
wscript                   #windows脚本宿主设置
write                     #写字板
wiaacmgr                  #扫描仪和照相机向导
winver                    #检查Windows版本
lpksetup                  #系统安装语言包向导
```
关机命令
```
shutdown -i               #打开设置自动关机对话框
shutdown -a               #取消自动关机
shutdown -f               #强行关闭应用程序
shutdown -r               #注销当前用户
shutdown -c               #消息内容：输入关机对话框中的消息
```
cmd界面可运行的命令
```
nbtstat -A ip             #局域网通过ip地址查看主机名
net start [service]       #不带参数,则显示当前正在运行的服务列表;带参数则开启本地的服务
arp.exe                   #显示和更改计算机的ip与硬件物理地址的对应列表
at.exe                    #计划运行任务
> at 11:25 taskkill /im qq.exe /f  #在11:25 杀掉qq.exe进程
attrib.exe                #显示和更改文件和文件夹属性
cacls.exe                 #显示和编辑acl(文件访问属性)
change.exe                #与终端服务器相关的查询
comp.exe                  #比较两个文件和文件集的内容
convert.exe               #转换文件系统到ntfs
defrag                    #磁盘碎片整理
sc                        #用于与服务控制管理器和服务进行通信的命令行程序
wmic                      #扩展WMI(Windows管理工具)
```
