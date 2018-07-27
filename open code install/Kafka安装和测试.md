------ CentOS-7.2-x86_64 系统 --------

<font color=#FF0000 size=5><p align="center">kafka简介</p></font>

kafka是一款分布式消息发布和订阅的系统，具有高性能和高吞吐率

1.消息的发布（publish）称作producer，消息的订阅（subscribe）称作consumer，中间的存储阵列称作broker

2.多个broker协同合作，producer、consumer和broker三者之间通过zookeeper来协调请求和转发

3.producer产生和推送(push)数据到broker，consumer从broker拉取(pull)数据并进行处理

4.broker端不维护数据的消费状态，提升了性能

5.直接使用磁盘进行存储，线性读写，速度快：避免了数据在JVM内存和系统内存之间的复制，减少耗性能的创建对象和垃圾回收

6.Kafka使用scala编写，可以运行在JVM上

![image](https://github.com/dwjlw1314/DWJ-PROJECT/raw/master/PictureSource/3.6.2.png)

<font color=#FF0000 size=5><p align="center">kafka安装及部署</p></font>

<font color=#FF0000>1.环境配置</font>  <br>
1. Kafka版本：kafka_2.10-0.10.1.1.tgz；
2. JDK版本：jdk-7u79-linux-x64.tar.gz；
3. zookeeper版本：zookeeper-3.4.6.tar.gz
4. CentOS7.2 ip1=192.168.1.181；ip2=192.168.1.182；ip3=192.168.1.183

<font color=#FF0000>2.JDK安装过程</font>  <br>
1. [root@dwj ~]# mkdir /usr/jdk   &nbsp;#在/usr目录下创建文件夹jdk        
2. [root@dwj ~]# cp /root/Desktop/kafka/jdk-7u79-linux-x64.tar.gz /usr/jdk/ &nbsp;#复制jkd文件到/usr/jdk目录中
3. [root@dwj jdk]# tar -zxvf jdk-7u79-linux-x64.tar.gz &nbsp;#解压jdk压缩包
4. [root@dwj jdk]# vim /etc/profile <br>
编辑/etc/profile文件，添加以下内容:
```
JAVA_HOME=/usr/jdk/jdk1.7.0_79
CLASSPATH=.:$JAVA_HOME/lib/tools.jar:$JAVA_HOME/lib/dt.jar
PATH=$JAVA_HOME/bin:$PATH
export JAVA_HOME CLASSPATH PATH
```
5. [root@dwj jdk]# source /etc/profile       &nbsp;#使配置生效
6. [root@dwj jdk]# java -version             &nbsp;#验证jdk
```
java version "1.7.0_79"
Java(TM) SE Runtime Environment (build 1.7.0_79-b15)
Java HotSpot(TM) 64-Bit Server VM (build 24.79-b02, mixed mode)
```
<font color=#FF0000>3.安装zookeerper</font>  <br>
1. [root@dwj ~]# mkdir /usr/zookeeper     #在/usr目录下创建文件夹zookeeper
2. [root@dwj ~]# cp /root/Desktop/kafka/zookeeper-3.4.6.tar.gz  /usr/zookeeper/    #复制zookeeper文件到/usr/ zookeeper目录中
3. [root@dwj zookeeper]# tar -zxvf  zookeeper-3.4.6.tar.gz      #解压zookeeper压缩包
4. [root@dwj zookeeper]# vim /etc/profile
编辑/etc/profile文件，添加以下内容:
```
ZOOKEEPER_HOME=/usr/zookeeper/zookeeper-3.4.6
PATH=$JAVA_HOME/bin:$ZOOKEEPER_HOME/bin:$PATH
CLASSPATH=.:$JAVA_HOME/lib/dt.jar:$JAVA_HOME/lib/tools.jar:$ZOOKEEPER_HOME/lib:
export ZOOKEEPER_HOME
```
5. [root@dwj zookeeper]# source /etc/profile      #使配置生效
6. [root@dwj conf]# mv zoo_sample.cfg zoo.cfg     #进入conf/目录下，将zoo_sample.cfg文件名称改为zoo.cfg,
配置内容如下:
```
tickTime：   这个时间是Zookeeper服务器或客户端与服务器之间维持心跳的时间间隔
dataDir：    是Zookeeper保存数据的目录，默认情况下Zookeeper将写数据的日志文件也保存在这个目录里
dataLogDir:  log目录, 同样可以是任意目录. 如果没有设置该参数, 将使用和dataDir相同的设置
clientPort： 客户端连接Zookeeper服务器的端口
```
7. [root@dwj bin]# ./zkServer.sh start    #启动zookeeper服务器

<font color=#FF0000>4.安装kafka</font>  <br>
1. [root@dwj ~]# mkdir /usr/kafka  #在/usr目录下创建文件夹kafka
2. [root@dwj ~]# cp /root/Desktop/kafka/kafka_2.10-0.10.1.1.tgz /usr/kafka/    #复制kafka文件到/usr/kafka目录中
3. [root@dwj kafka]# tar -zxvf  kafka_2.10-0.10.1.1.tgz      #解压kafka压缩包，Kafka目录介绍
```
/bin    操作kafka的可执行脚本，还包含windows下脚本
/config 配置文件所在目录
/libs   依赖库目录
/logs   日志数据目录，目录kafka把server端日志分为5种类型，分为:server,request,state，log-cleaner，controller
```
4. [root@dwj kafka]# vim kafka_2.10-0.10.1.1/config/server.properties  #编辑kafka配置文件
主要是broker.id、log.dirs、zookeeper.connect
5. [root@dwj bin]# ./kafka-server-start.sh /usr/kafka/kafka_2.10-0.10.1.1/config/server.properties &       #启动kafka服务
6. [root@dwj bin]# netstat -nap|egrep "(2181|9092)"      #检查zookeeper和kafka服务器是否启动
7. [root@dwj bin]# ./kafka-console-producer.sh --broker-list 192.168.1.181:9092 --topic test    #生产者发送消息
8. [root@dwj bin]# ./kafka-console-consumer.sh --zookeeper 192.168.1.181:2181 --topic test --from-beginning   #消费者接收消息<br>
	producer，指定的Socket(192.168.1.181+9092),说明生产者的消息要发往kafka，也即是broker   <br>
	consumer, 指定的Socket(192.168.1.181+2181),说明消费者的消息来自zookeeper（协调转发）
