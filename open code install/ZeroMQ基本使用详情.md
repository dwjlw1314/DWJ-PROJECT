<font color=#FF0000 size=5> <p align="center">ZeroMQ 概念</p></font>

ZeroMQ(简称ZMQ)是一个基于消息队列的多线程网络库，其对套接字类型、连接处理、帧、甚至路由的底层细节进行抽象，提供跨越多种传输协议的套接字。ZMQ是介于应用层和传输层之间(按照TCP/IP划分)。ZMQ是以库的形式存在，它封装了网络通信、消息队列、线程调度等功能，向上层提供简洁的API函数来实现高性能网络通信。但是ZeroMQ仅提供非持久性的消息队列

<font color=#FF0000 size=5> <p align="center">ZeroMQ 系统架构</p></font>

ZeroMQ几乎所有的I/O操作都是异步的，主线程不会被阻塞。ZeroMQ会根据用户调用zmq_init函数时传入的接口参数，创建对应数量的I/O Thread。每个I/O Thread都有与之绑定的Poller，Poller采用经典的Reactor模式实现，Poller根据不同操作系统平台使用不同的网络I/O模型(select、poll、epoll、devpoll、kequeue等)。主线程与I/O线程通过Mail Box传递消息来进行通信。Server开始监听或者Client发起连接时，在主线程中创建zmq_listener或zmq_connecter，通过Mail Box发消息的形式将其绑定到I/O线程，I/O线程会把zmq_connecter或zmq_listener添加到Poller中用以侦听读/写事件。Server与Client在第一次通信时，zmq_init用来发送identity，用以进行认证。认证结束后，双方会为此次连接创建Session，以后双方就通过Session进行通信。每个Session都会关联到相应的读/写管道， 主线程收发消息只是分别从管道中读/写数据。Session并不实际跟kernel交换I/O数据，而是通过plugin到Session中的Engine来与kernel交换I/O数据

![image](https://github.com/dwjlw1314/DWJ-PROJECT/raw/master/PictureSource/3.28.1.jpg)

ZeroMQ将消息通信分成4种模型：
1.一对一结对模型(Exclusive-Pair)，可以认为是一个TCP Connection，但是TCP Server只能接受一个连接。数据可以双向流动

2.请求回应模型(Request-Reply)，由Client发起请求，并由Server响应，跟一对一结对模型的区别在于可以有多个Client。该模型主要用于远程调用及任务分配等。Echo服务就是这种经典模型的应用

![image](https://github.com/dwjlw1314/DWJ-PROJECT/raw/master/PictureSource/3.28.2.jpg)

3.发布订阅模型(Publish-Subscribe)。Publish端单向分发数据，且不关心是否把全部信息发送给Subscribe端。如果Publish端开始发布信息时，Subscribe端尚未连接进来，则这些信息会被直接丢弃。Subscribe端只能接收，不能反馈，且在Subscribe端消费速度慢于Publish端的情况下，会在Subscribe端堆积数据。同样，订阅端则只负责接收，而不能反馈。该模型主要用于数据分发。如果发布端和订阅端需要交互(比如要确认订阅者是否已经连接)，则使用额外的socket采用请求回应模型满足这个需求

![image](https://github.com/dwjlw1314/DWJ-PROJECT/raw/master/PictureSource/3.28.3.jpg)

4.管道模型(Push-Pull)，从Push端向Pull端单向的推送数据流。如果有多个Pull端同时连接到Push端，则Push端会在内部做一个负载均衡，采用平均分配的算法，将所有消息均衡发布到Pull端上。与发布订阅模型相比，管道模型在没有消费者的情况下，发布的消息不会被消耗掉；在消费者能力不够的情况下，能够提供多消费者并行消费解决方案。该模型主要用于多任务并行

![image](https://github.com/dwjlw1314/DWJ-PROJECT/raw/master/PictureSource/3.28.4.jpg)

这4种模型总结出了通用的网络通信模型，在实际中可以根据应用需要，组合其中的2种或多种模型来形成自己的解决方案

<font color=#FF0000 size=5> <p align="center">ZeroMQ 通信协议及工作流程</p></font>

ZMQ提供进程内、进程间、机器间、广播等四种通信协议，配置简单，用类似于URL形式的字符串指定即可，格式分别为inproc://、ipc://、tcp://、pgm://。ZeroMQ会自动根据指定的字符串解析出协议、地址、端口号等信息

![image](https://github.com/dwjlw1314/DWJ-PROJECT/raw/master/PictureSource/3.28.5.jpg)

<font color=#FF0000 size=5> <p align="center">ZeroMQ 管道模式应用场景</p></font>

应用ZeroMQ的Push-Pull模型实现游戏服务器的“热插拔”、负载均衡和消息派发。如图部署服务器，Push端充当Gateway，作为一组游戏服务器集群最上层的一个Proxy，起负载均衡的作用，所有Gameserver作为Pull端。当一个请求到达Push端（Gateway）时，Push端根据一定的分配策略将任务派发到Pull端（Gameserver）。以某款游戏为例，刚上线时预计最大同时在线人数是10W，单台Gameserver并发处理能力为1W，需要10台Gameserver，在线人数增加到50W，不需要将Gateway和Gameserver停机，只需要随时在机房新添加40台Gameserver，启动并连接到Gateway即可。ZeroMQ中对Client和Server的启动顺序没有要求，Gameserver之间如果需要通信的话，Gameserver的应用层不需要管理这些细节，ZeroMQ已经做了重连处理

![image](https://github.com/dwjlw1314/DWJ-PROJECT/raw/master/PictureSource/3.28.6.jpg)

<font color=#FF0000 size=5> <p align="center">ZeroMQ 特点</p></font>

```
1、仅仅提供24个API接口，风格类似于BSD Socket
2、处理了网络异常，包括连接异常中断、重连等
3、改变TCP基于字节流收发数据的方式，处理了粘包、半包等问题，以msg为单位收发数据，可以对应用层彻底屏蔽网络通信层
4、对大数据通过SENDMORE/RECVMORE提供分包收发机制
5、通过线程间数据流动来保证同一时刻任何数据都只会被一个线程持有，以此实现多线程的“去锁化”
6、通过高水位HWM来控制流量，用交换SWAP来转储内存数据，弥补HWM丢失数据的缺陷
7、服务器端和客户端的启动没有先后顺序
8、支持多种通信协议，可以灵活地适应多种通信环境，包括进程内、进程间、机器间、广播
9、可以绑定C、C++、Java、.NET、Python等30多种开发语言
```

<font color=#FF0000 size=5> <p align="center">ZeroMQ 源码编译</p></font>

准备相关软件包
```
ZeroMQ: zeromq-4.2.5.tar.gz
```
源码安装ZeroMQ
```
[root@dwj ~]# tar -xvf zeromq-4.2.5.tar.gz
[root@dwj zeromq-4.2.5]# ./configure --prefix=/usr/local/zeromq
[root@dwj zeromq-4.2.5]# make && make install
```

<font color=#FF0000 size=5> <p align="center">ZeroMQ 代码模型</p></font>
