iftop是类似于top的实时流量监控工具，可以用来监控网卡的实时流量（可以指定网段）、反向解析IP、显示端口信息等

主要以Red Hat Enterprise 6.5系统为例，介绍iftop命令：

1.安装iftop,首先安装依赖包flex byacc libpcap ncurses ncurses-devel libpcap-devel
```
[root@dwj opt]# wget http://www.tcpdump.org/release/libpcap-1.8.1.tar.gz
[root@dwj opt]# tar -zxvf libpcap-1.8.1.tar.gz
[root@dwj opt]# cd libpcap-1.8.1
[root@dwj libpcap-1.8.1]# ./configure
[root@dwj libpcap-1.8.1]# make && make install

[root@dwj opt]# wget http://www.ex-parrot.com/pdw/iftop/download/iftop-1.0pre4.tar.gz
[root@dwj opt]# tar -zxvf iftop-1.0pre4.tar.gz
[root@dwj opt]# cd iftop-1.0pre4
[root@dwj iftop-1.0pre4]# ./configure
[root@dwj iftop-1.0pre4]# make && make install
```
直接运行：iftop 效果如下图：
>[root@dwj /]# iftop

![image](https://github.com/dwjlw1314/DWJ-PROJECT/raw/master/PictureSource/4.21.1.png)

iftop界面上显示的是类似刻度尺的刻度范围，为显示流量图形的长条作标尺用的,中间的<= =>这两个左右箭头表示的是流量的方向

字段名 | 含义
---|---
TX | 发送流量
RX | 接收流量
TOTAL | 总流量
Cumm | 运行iftop到目前时间的总流量
peak | 流量峰值
rates | 分别表示过去 2s 10s 40s 的平均流量

2.相关参数及说明

参数名 | 描述
---|---
-i | 设定监测的网卡，如：#iftop -i eth1
-B | 以bytes为单位显示流量(默认是bits)，如：#iftop -B
-n | 使host信息默认直接都显示IP，如：#iftop -n
-N | 使端口信息默认直接都显示端口号，如: #iftop -N
-F | 显示特定网段的进出流量，如：#iftop -F 10.10.1.0/24或# iftop -F 10.10.1.0/255.255.255.0
-h | 帮助，显示参数信息
-p | 使用这个参数后，中间的列表显示的本地主机信息，出现了本机以外的IP信息
-b | 不显示流量图形条
-f | 包统计使用的过滤码；默认：无，只统计IP包
-F | net/mask显示特定IPv4网段的进出流量(Flow)如: #iftop -F 10.10.1.0/24
-P | 使host信息及端口信息默认都显示
-m | 设置界面最上边的刻度的最大值，刻度分五个大段显示，例：#iftop -m 100M
-t | 使用不带窗口菜单的文本(text)接口

进入iftop画面后的一些操作命令(注意大小写)
```
按<q>退出监控
按<h>切换是否显示帮助
按<P>切换暂停/继续显示
按<p>切换是否显示端口信息
按<n>切换显示本机的IP或主机名
按<S>切换是否显示本机的端口信息
按<b>切换是否显示平均流量图形条
按<s>切换是否显示本机的host信息
按<N>切换显示端口号或端口服务名称
按<o>切换是否固定只显示当前的连接
按<T>切换是否显示每个连接的总流量
按<d>切换是否显示远端目标主机的host信息
按<D>切换是否显示远端目标主机的端口信息
按<B>切换计算2秒或10秒或40秒内的平均流量
按<j>或按<k>可以向上或向下滚动屏幕显示的连接记录
按<L切换显示画面上边的刻度;刻度不同，流量图形条会有变化
按<t>切换显示格式为2行/1行/只显示发送流量/只显示接收流量
按<f>可以编辑过滤代码，这是翻译过来的说法，我还没用过这个
按<l>打开屏幕过滤功能，输入要过滤的字符，比如ip,按回车后，屏幕就只显示这个IP相关的流量信息
按<根据左边的本机名或IP排序
按>根据远端目标主机的主机名或IP排序
按1或2或3可以根据右侧显示的三列流量数据进行排序
```
