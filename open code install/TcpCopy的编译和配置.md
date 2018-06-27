------ rhel-server-6.4-x86_64 系统 --------

TcpCopy version =1.0

编译部分参考开元包说明文件

172.26.5.4   ./tcpcopy -x 9876-172.26.5.1:9876 -s 172.26.5.15 -d -c 172.26.5.6

172.26.5.1   route add -host 172.26.5.6 gw 172.26.5.15

172.26.5.15  ./intercept -i bond0 -F 'tcp and src port 9876' -d
	     ./intercept -i bond0 -F 'tcp and src host 172.26.5.1 and src port 9876' -d
____
172.28.5.6   ./tcpcopy -x 9876-172.28.5.4:9876 -s 172.28.5.5 -d -c 172.28.5.9

172.28.5.4   route add -host 172.28.5.9 gw 172.28.5.5

172.28.5.5  ./intercept -i bond0 -F 'tcp and src port 9876' -d
	     ./intercept -i bond0 -F 'tcp and src host 172.28.5.4 and src port 9876' -d

注意： 先启动intercept ，后启动tcpcopy
