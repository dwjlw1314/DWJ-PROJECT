一、准备阶段
```
两个节点，集群IP地址：192.168.100.3
节点A：计算机名：WIN-A  IP地址： 192.168.100.1
节点B：计算机名：WIN-B  IP地址： 192.168.100.2
```
一、安装功能

1.进入服务器管理器---管理---添加规则和功能
![image](https://github.com/dwjlw1314/DWJ-PROJECT/raw/master/PictureSource/nlb_Failover/2.6.1.png)

2.点击下一步（Next）
![image](https://github.com/dwjlw1314/DWJ-PROJECT/raw/master/PictureSource/nlb_Failover/2.6.2.png)

3.点击下一步（Next）
![image](https://github.com/dwjlw1314/DWJ-PROJECT/raw/master/PictureSource/nlb_Failover/2.6.3.png)

4.点击下一步（Next）
![image](https://github.com/dwjlw1314/DWJ-PROJECT/raw/master/PictureSource/nlb_Failover/2.6.4.png)

5.点击下一步（Next）
![image](https://github.com/dwjlw1314/DWJ-PROJECT/raw/master/PictureSource/nlb_Failover/2.6.5.png)

6.选择网络负载均衡（Network Load Balancing）
![image](https://github.com/dwjlw1314/DWJ-PROJECT/raw/master/PictureSource/nlb_Failover/2.6.6.png)

7.点击安装（Install）
![image](https://github.com/dwjlw1314/DWJ-PROJECT/raw/master/PictureSource/nlb_Failover/2.6.7.png)

8.安装完成
![image](https://github.com/dwjlw1314/DWJ-PROJECT/raw/master/PictureSource/nlb_Failover/2.6.8.png)
注：两台节点服务器均需安装网络负载均衡功能

三、配置功能

首先配置A节点 <br>
9.服务器管理器---工具---网络负载均衡管
![image](https://github.com/dwjlw1314/DWJ-PROJECT/raw/master/PictureSource/nlb_Failover/2.6.9.png)

10.右击Network Load Balancing Cluster选择New Cluster
![image](https://github.com/dwjlw1314/DWJ-PROJECT/raw/master/PictureSource/nlb_Failover/2.6.10.png)

11.输入WIN-A服务器的IP地址（即192.168.100.1），点击连接，选择网卡，下一步
![image](https://github.com/dwjlw1314/DWJ-PROJECT/raw/master/PictureSource/nlb_Failover/2.6.11.png)

12.默认选择，下一步。（顶端可以设置优先级）
![image](https://github.com/dwjlw1314/DWJ-PROJECT/raw/master/PictureSource/nlb_Failover/2.6.12.png)

13.点击Add，输入集群IP和子网掩码，点击OK，然后下一步
![image](https://github.com/dwjlw1314/DWJ-PROJECT/raw/master/PictureSource/nlb_Failover/2.6.13.png)

14.选择多播（Multicast），下一步
![image](https://github.com/dwjlw1314/DWJ-PROJECT/raw/master/PictureSource/nlb_Failover/2.6.14.png)

15.确定端口规则，点击结束
![image](https://github.com/dwjlw1314/DWJ-PROJECT/raw/master/PictureSource/nlb_Failover/2.6.15.png)

16.等待数秒，A节点显示已聚合
![image](https://github.com/dwjlw1314/DWJ-PROJECT/raw/master/PictureSource/nlb_Failover/2.6.16.png)

然后配置B节点 <br>
17.在B节点上打开网络负载均衡管理器，右击Network Load Balancing Cluster选择Connect to Existing
![image](https://github.com/dwjlw1314/DWJ-PROJECT/raw/master/PictureSource/nlb_Failover/2.6.17.png)

18.输入集群IP（即192.168.100.3），点击连接，选择群集，点击结束
![image](https://github.com/dwjlw1314/DWJ-PROJECT/raw/master/PictureSource/nlb_Failover/2.6.18.png)

19.右键选择Add Host To Cluster
![image](https://github.com/dwjlw1314/DWJ-PROJECT/raw/master/PictureSource/nlb_Failover/2.6.19.png)

20.输入WIN-B节点的IP地址，点击连接，选择网卡，下一步
![image](https://github.com/dwjlw1314/DWJ-PROJECT/raw/master/PictureSource/nlb_Failover/2.6.20.png)

21.默认设置，下一步
![image](https://github.com/dwjlw1314/DWJ-PROJECT/raw/master/PictureSource/nlb_Failover/2.6.21.png)

22.默认设置，下一步
![image](https://github.com/dwjlw1314/DWJ-PROJECT/raw/master/PictureSource/nlb_Failover/2.6.22.png)

23.等待数秒后，节点B同样显示已聚合。至此负载均衡配置完毕
![image](https://github.com/dwjlw1314/DWJ-PROJECT/raw/master/PictureSource/nlb_Failover/2.6.23.png)
