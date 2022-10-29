
<font size=5><p align="center">samba 简介</p></font>
```
Samba 是在 Linux 和 UNIX 系统上实现 SMB 协议的一个免费软件，由服务器及客户端程序构成。

NFS 与 samba 一样，也是在网络中实现文件共享的一种实现，但不幸的是，其不支持 windows 平台，samba 是能够在任何支持 SMB 协议的主机之间共享文件的一种实现，当然也包括 windows。

SMB 是一种在局域网上共享文件和打印机的一种通信协议，它为局域网内的不同计算机之间提供文件及打印机等资源的共享服务。

SMB 协议是 C/S 型协议，客户机通过该协议可以访问服务器上的共享文件系统、打印机及其他资源。
```

### samba监听端口

|<center> TCP </center>|<center>UDP</center>|
|:----------:|:--------:|
| 139 & 445|	137 & 138 |

* tcp 端口相对应的服务是 smbd 服务，其作用是提供对服务器中文件、打印资源的共享访问
* udp 端口相对应的服务是 nmbd 服务，其作用是提供基于 NetBIOS 主机名称的解析

### samba进程

|<center>进程</center>|<center>对应</center>|
|:----------:|:--------:|
|  nmbd	     |对应 netbios|
|  smbd	     |对应 cifs 协议|
|  winbindd + ldap	|对应 Windows AD 活动目录|

### samba安全级别有三个，分别是 user，server，domain

|<center>安全级别</center>|<center>作用</center>|
|:----------:|:--------:|
|   user	   | 基于本地的验证 |
|   server	 | 由另一台指定的服务器对用户身份进行认证 |
|   domain	 | 由域控进行身份验证 |

### 常用配置文件参数
|<center>参数</center>|<center>作用</center>|
|:----------:|:--------:|
|workgroup	 | 表示设置工作组名称 |
|server string | 	表示描述 samba 服务器 |
|security |	表示设置安全级别，其值可为 share、user、server、domain |
|passdb backend	 |  表示设置共享帐户文件的类型，其值可为 tdbsam(tdb数据库文件)、ldapsam(LDAP目录认证)、smbpasswd(兼容旧版本 samba 密码文件) |
|comment	 |  表示设置对应共享目录的注释，说明信息，即文件共享名 |
|browseable	 |  表示设置共享是否可见 |
|writable	 |  表示设置目录是否可写 |
|path	 |  表示共享目录的路径 |
|guest ok	 |  表示设置是否所有人均可访问共享目录 |
|public	 |  表示设置是否允许匿名用户访问 |
|write list	 |  表示设置允许写的用户和组，组要用 @ 表示，例如 write list = root,@root |
|valid users	 |  设置可以访问的用户和组，例如 valid users = root,@root |
|hosts deny	 |  设置拒绝哪台主机访问，例如 hosts deny = 192.168.10.100 |
|hosts allow	 |  设置允许哪台主机访问，例如 hosts allow = 192.168.10.200 |
|printable	 |  表示设置是否为打印机 |

### 搭建共享服务器
1. 安装samba
>[root@dwj /]# sudo apt-get install samba

2. 查看安装好的版本
>[root@dwj /]# samba -V

3. 创建访问共享文件夹的用户名 (略)

4. 使用系统已经存在的用户名来访问共享文件夹，直接设置访问密码
>[root@dwj /]# sudo smbpasswd -a dwj

5. 设置好密码后，设置共享文件夹的用户权限：
>[root@dwj /]# sudo chown dwj /home/share  #/home/share:共享文件夹路径

6. samba后默认是添加到防火墙服务内的(非必须）
>[root@dwj /]# sudo firewall-cmd --add-service=samba

7. 修改samba配置文件, 文件末尾添加
>[root@dwj /]# sudo vi /etc/samba/smb.conf
```shell
[share]
path = /home/share
valid users = dwj
available = yes
browseable = yes
writable = yes
public = no
```

8. 重启samba
>[root@dwj /]# sudo service smbd restart
