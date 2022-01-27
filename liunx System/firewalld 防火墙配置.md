注意：firewalld服务有两份规则策略配置记录，配置永久生效的策略记录时，需要执行"reload"参数后才能立即生效：
```
Permanent:永久生效的
RunTime:现在正在生效的
```

1.查看当前区域
>[root@dwj ~]# chkconfig iptables off

2.查看防火墙状态，需要注意的是，设置端口规则时防火墙必须是启动状态
>[root@dwj ~]# systemctl status firewalld 　　　　//返回信息提示防火墙未启动
```
firewalld.service - firewalld - dynamic firewall daemon
 Loaded: loaded (/usr/lib/systemd/system/firewalld.service; disabled; vendor preset: enabled)
 Active: inactive (dead)　　　　　　//dead代表关闭状态
   Docs: man:firewalld(1)
```

>[root@dwj ~]# systemctl start firewalld 　　　　//启动防火墙

>[root@dwj ~]# systemctl status firewalld
```
firewalld.service - firewalld - dynamic firewall daemon
 Loaded: loaded (/usr/lib/systemd/system/firewalld.service; disabled; vendor preset: enabled)
 Active: active (running) since Tue 2018-10-09 19:38:36 CST; 2s ago
   Docs: man:firewalld(1)
Main PID: 9269 (firewalld)
 CGroup: /system.slice/firewalld.service
         └─9269 /usr/bin/python -Es /usr/sbin/firewalld --nofork --nopid
```

3.配置需要开发的端口，--permanent的作用是使设置永久生效，不加的话机器重启之后失效
>[root@dwj ~]# firewall-cmd --zone=public --add-port=5432/tcp --permanent   //success

4.执行命令使端口生效
>[root@dwj ~]# firewall-cmd --reload    //success

5、批量开放或限制端口
>[root@dwj ~]# firewall-cmd --zone=public --add-port=100-500/tcp --permanent   //success

6.查看端口是否生效
>[root@dwj ~]# firewall-cmd --zone=public --query-port=5432/tcp    //yes

7.查看所有开放端口
>[root@dwj ~]# firewall-cmd --list-port

8.删除一个端口
>[root@dwj ~]# firewall-cmd --zone=public --remove-port=8080/tcp --permanent

9.查看版本
>[root@dwj ~]# firewall-cmd --version

10.查看状态
>[root@dwj ~]# firewall-cmd --state

11.查看所在区域
>[root@dwj ~]# firewall-cmd --get-active-zones

12.限制IP地址访问
>[root@dwj ~]# firewall-cmd --permanent --add-rich-rule="rule family="ipv4" source address="192.168.0.200" port protocol="tcp" port="80" reject"

13.解除IP地址限制
>[root@dwj ~]# firewall-cmd --permanent --add-rich-rule="rule family="ipv4" source address="192.168.0.200" port protocol="tcp" port="80" accept"

14.查看ip规则，确认是否生效
>[root@dwj ~]# firewall-cmd --zone=public --list-rich-rules

15.编辑规则文件，删掉原来的设置规则，重新载入一下防火墙即可
>[root@dwj ~]# vim /etc/firewalld/zones/public.xml

16.整段的IP地址，具体的设置规则可参考下表:
![image](https://github.com/dwjlw1314/DWJ-PROJECT/raw/master/PictureSource/1.20.1.jpg)
