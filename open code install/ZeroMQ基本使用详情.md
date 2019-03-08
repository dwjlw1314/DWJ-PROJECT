<font color=#FF0000 size=5> <p align="center">ZeroMQ 概念</p></font>

ZeroMQ(简称ZMQ)是一个基于消息队列的多线程网络库，其对套接字类型、连接处理、帧、甚至路由的底层细节进行抽象，提供跨越多种传输协议的套接字。ZMQ是介于应用层和传输层之间(按照TCP/IP划分)，是一个可伸缩层，可并行运行。ZMQ不是单独的服务，而是一个嵌入式库，它封装了网络通信、消息队列、线程调度等功能，向上层提供简洁的API，应用程序通过加载库文件，调用API函数来实现高性能网络通信

ZeroMQ将消息通信分成4种模型：
1.一对一结对模型(Exclusive-Pair)，可以认为是一个TCP Connection，但是TCP Server只能接受一个连接。数据可以双向流动

2.请求回应模型(Request-Reply)，由Client发起请求，并由Server响应，跟一对一结对模型的区别在于可以有多个Client

3.发布订阅模型(Publish-Subscribe)。Publish端单向分发数据，且不关心是否把全部信息发送给Subscribe端。如果Publish端开始发布信息时，Subscribe端尚未连接进来，则这些信息会被直接丢弃。Subscribe端只能接收，不能反馈，且在Subscribe端消费速度慢于Publish端的情况下，会在Subscribe端堆积数据。同样，订阅端则只负责接收，而不能反馈。如果发布端和订阅端需要交互(比如要确认订阅者是否已经连接)，则使用额外的socket采用请求回应模型满足这个需求

4.管道模型(Push-Pull)，从Push端向Pull端单向的推送数据流。如果有多个Pull端同时连接到Push端，则Push端会在内部做一个负载均衡，采用平均分配的算法，将所有消息均衡发布到Pull端上。与发布订阅模型相比，管道模型在没有消费者的情况下，发布的消息不会被消耗掉；在消费者能力不够的情况下，能够提供多消费者并行消费解决方案。该模型主要用于多任务并行

这4种模型总结出了通用的网络通信模型，在实际中可以根据应用需要，组合其中的2种或多种模型来形成自己的解决方案

ZMQ提供进程内（inproc://）、进程间（ipc://）、机器间（tcp://）、广播（pgm://）等四种通信协议

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

主线程与I/O线程：

I/O线程，ZMQ根据用户调用zmq_init函数时传入的参数，创建对应数量的I/O线程。每个I/O线程都有与之绑定的Poller，Poller采用经典的Reactor模式实现。Poller根据不同操作系统平台使用不同的网络I/O模型(select、poll、epoll、devpoll、kequeue等)，所有的I/O操作都是异步的，线程不会被阻塞。主线程与I/O线程通过Mail Box传递消息来进行通信

![image](https://github.com/dwjlw1314/DWJ-PROJECT/raw/master/PictureSource/3.28.1.jpg)

```
Server，在主线程创建zmq_listener，通过Mail Box发消息的形式将其绑定到I/O线程，I/O线程把zmq_listener添加到Poller中用以侦听读事件
Client，在主线程中创建zmq_connecter，通过Mail Box发消息的形式将其绑定到I/O线程，I/O线程把zmq_connecter添加到Poller中侦听写事件
Client与Server第一次通信时，会创建zmq_init来进行身份认证。双方会为此次连接创建Session，以后双方就通过Session进行通信
每个Session都会关联到相应的读/写管道， 主线程收发消息只是分别从管道中读/写数据。Session并不实际跟kernel交换I/O数据，而是通过plugin到Session中的Engine来与kernel交换I/O数据
```
