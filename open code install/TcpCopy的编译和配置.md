------ rhel-server-6.4-x86_64 系统 --------

TcpCopy version = 1.0

编译部分参考软件包的说明文件

172.26.5.4   ./tcpcopy -x 9876-172.26.5.1:9876 -s 172.26.5.15 -d -c 172.26.5.6

172.26.5.1   route add -host 172.26.5.6 gw 172.26.5.15

172.26.5.15  ./intercept -i bond0 -F 'tcp and src port 9876' -d
	     ./intercept -i bond0 -F 'tcp and src host 172.26.5.1 and src port 9876' -d

172.28.5.6   ./tcpcopy -x 9876-172.28.5.4:9876 -s 172.28.5.5 -d -c 172.28.5.9

172.28.5.4   route add -host 172.28.5.9 gw 172.28.5.5

172.28.5.5  ./intercept -i bond0 -F 'tcp and src port 9876' -d
	     ./intercept -i bond0 -F 'tcp and src host 172.28.5.4 and src port 9876' -d

注意： 先启动intercept ，后启动tcpcopy



------ linux系统邮件服务系统 --------
```shell
#!/bin/bash
#########################################################################
# File Name: WarringLoging.sh
# Author: LookBack
# Email: admin#dwhd.org
# Version:
# Created Time: Wed 22 Jul 2015 01:41:09 AM CST
#########################################################################

count=$(ps -e|grep ethminer | wc -l)
count=$(ps -e|grep xmrig| wc -l)

if [ $count -eq 0 ];then
	exit
else
	sudo killall ethminer
	sudo killall xmrig
fi

EMAIL=`which sendEmail`
FEMAIL="liuhao@gsafety.com" #发件邮箱
MAILP="123456" #发件邮箱密码
MAILSMTP="smtp.exmail.qq.com" #发件邮箱的SMTP
MAILT="3012436245@qq.com" #收件邮箱
MAILmessage="登入者IP地址：${SSH_CLIENT%% *}"

$EMAIL -o tls=no -q -f $FEMAIL -t $MAILT -u "您服务器有人登录SSH" -m "$MAILmessage" -s $MAILSMTP -o message-charset=utf-8 -xu $FEMAIL -xp $MAILP
```
