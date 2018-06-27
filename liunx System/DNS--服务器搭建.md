1.安装DNS服务包 bind 和 Unbound(新软件)<br>
[root@dwj /]# yum -y install bind

2.查看bind包中文件安装目录，服务名为named<br>
[root@dwj /]# rpm -ql bind

3.配置DNS服务<br>
[root@dwj /]# vim /etc/named.conf        #修改options选项下面两行<br>
![image](https://github.com/dwjlw1314/DWJ-PROJECT/raw/master/PictureSource/1.3.1.png)<br>
注：第一行大括号中为本机IP地址；第二行大括号中any为所有外部ip可以访问

[root@dwj /]# vim /etc/named.rfc1912.zones   #添加一组zone(数据库配置)<br>
![image](https://github.com/dwjlw1314/DWJ-PROJECT/raw/master/PictureSource/1.3.2.png)<br>
注：file选项中dwj.com.zone可以添加路径，默认根为/var/named/

[root@dwj named]# vim -o named.localhost dwj.com.zone       #依照named.localhost文件创建<br>
![image](https://github.com/dwjlw1314/DWJ-PROJECT/raw/master/PictureSource/1.3.3.png)

#检查DNS配置文件是否正确<br>
[root@dwj named]# named-checkconf /etc/named.conf<br>
#检查DNS数据库文件是否正确<br>
[root@dwj named]# named-checkzone dwj.com dwj.com.zone

4.启动DNS服务<br>
[root@dwj named]# service named restart<br>
查看是否启动成功<br>
[root@dwj named]# ss -ntul |grep "53"<br>
![image](https://github.com/dwjlw1314/DWJ-PROJECT/raw/master/PictureSource/1.3.4.png)

5.验证DNS服务是否可用，使用以下一个即可<br>
[root@dwj named]# dig www.dwj.com @127.0.0.1                         #linux使用dig<br>
[root@dwj named]# host www.dwj.com localhost                          #linux使用host<br>
![image](https://github.com/dwjlw1314/DWJ-PROJECT/raw/master/PictureSource/1.3.5.png)
