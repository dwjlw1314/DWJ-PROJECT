Windows系统默认目录
```
C:\Windows\Cursors                                             #系统自带鼠标指针
C:\Windows\Web\Wallpaper                                       #系统自带壁纸
C:\Windows\System32\drivers\etc                                #系统主机、服务和网络配置文件
C:\Windows\system32\oobe\info\background.bmp                   #自定义系统登录界面背景
C:\ProgramData\Microsoft\User Account Pictures                 #自带用户类型图像[来宾用户和当前用户的默认头像]
C:\Users\Administrator\AppData\Local\IconCache.db              #系统图标缓存
C:\Users\hsql\AppData\Roaming\Microsoft\Windows\Themes         #自定义桌面壁纸
C:\Users\hsql\AppData\Roaming\Microsoft\Windows\SendTo         #文件-右键菜单-发送到 图标位置
C:\ProgramData\Microsoft\Windows\Start Menu\Programs           #开始菜单-所有程序 图标存放位置

右键 锁定到任务栏 后图标存放位置 和 右键 附到开始菜单 后图标存放位置
C:\Users\hsql\AppData\Roaming\Microsoft\Internet Explorer\Quick Launch\User Pinned\TaskBar
C:\Users\hsql\AppData\Roaming\Microsoft\Internet Explorer\Quick Launch\User Pinned\StartMenu
```

Windows系统目录简写
```
%SYSTEMDRIVE%               #Windows系统所在磁盘分区，也就是Windows系统所安装到的盘符根目录
%HOMEDRIVE%                 #Windows系统所在磁盘分区，通常就是C盘的根目录
%SYSTEMROOT%                #Windows系统所在的目录，通常就是C:\Windows
%WINDIR%                    #%SYSTEMROOT%的功能相同，指向Windows所在目录
%ProgramFiles%              #Program Files的路径，通常情况下是C:\Program Files
%CommonProgramFiles%        #公用文件(Common Files)目录，通常是C:\Program Files\Common Files
%USERPROFILE%               #当前帐户的用户目录，通常是C:\Users\当前用户名
%HOMEPATH%                  #功能和%USERPROFILE%相同
%ALLUSERSPROFILE%           #所有用户的用户目录，通常是C:\ProgramData
%APPDATA%                   #当前用户的Application Data目录，通常是C:\Users\hsql\AppData\Roaming
%TEMP%                      #当前用户的临时文件目录，通常是C:\Users\当前用户名\AppData\Local\Temp
%ComSpec%                   #命令提示符，通常是C:\WINDOWS\System32\cmd.exe
```

Windows注册表路径
```
IE浏览器 右键菜单：
HKCU\Software\Microsoft\Internet Explorer\MenuExt
桌面 右键菜单：
HKLM\SOFTWARE\Classes\Directory\Background\shell
HKLM\SOFTWARE\Classes\Directory\Background\shellex
开机自动启动程序的位置：
HKLM\Software\Microsoft\Windows\CurrentVersion\Run
系统环境变量：
HKLM\SYSTEM\CurrentControlSet\Control\Session Manager\Environment
```
