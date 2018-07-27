一. 需要在系统中安装nicstat程序，测试版本---nicstat-1.95.tar.gz

二. 在解压包目录nicstat-1.95内,有个nicstat.sh脚本

1.查看网卡速度(-l)：

![image](https://github.com/dwjlw1314/DWJ-PROJECT/raw/master/PictureSource/3.15.1.jpg)

2.间隔3秒,查看2次结果(留意%Util和Sat):

![image](https://github.com/dwjlw1314/DWJ-PROJECT/raw/master/PictureSource/3.15.2.jpg)

字段名 | 含义
---|---
Time  | 表示当前采样的响应时间
Int   | 网卡名称
rKB/s | 每秒接收到千字节数
wKB/s | 每秒写的千字节数
rPk/s | 每秒接收到的数据包数目
wPk/s | 每秒写的数据包数目
rAvs  | 接收到的数据包平均大小
wAvs  | 传输的数据包平均大小
%Util | 网卡利用率(百分比)
Sat   | 网卡每秒的错误数.网卡是否接近饱满的一个指标

3.查看扩展信息(-x 和 -s):

![image](https://github.com/dwjlw1314/DWJ-PROJECT/raw/master/PictureSource/3.15.3.jpg)

4.查看tcp相关信息(-t):

![image](https://github.com/dwjlw1314/DWJ-PROJECT/raw/master/PictureSource/3.15.4.jpg)

字段名 | 含义
---|---
InKB  | 表示每秒接收到的千字节
OutKB | 表示每秒传输的千字节
InSeg | 表示每秒接收到的TCP数据段(TCP Segments)
OutSeg | 表示每秒传输的TCP数据段(TCP Segments)
Reset  | 表示TCP连接从ESTABLISHED或CLOSE-WAIT状态直接转变为CLOSED状态的次数
AttF   | 表示TCP连接从SYN-SENT或SYN-RCVD状态直接转变为CLOSED状态的次数,再加上TCP连接从SYN-RCVD状态直接转变为LISTEN状态的次数
%ReTX  | 表示TCP数据段(TCP Segments)重传的百分比.即传输的TCP数据段包含有一个或多个之前传输的八位字节
InConn | 表示TCP连接从LISTEN状态直接转变为SYN-RCVD状态的次数
OutCon | 表示TCP连接从CLOSED状态直接转变为SYN-SENT状态的次数
Drops  | 表示从完成连接(completed connection)的队列和未完成连接(incomplete connection)的队列中丢弃的连接次数
