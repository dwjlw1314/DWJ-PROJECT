<font color=#FF0000 size=5> <p align="center">RabbitMQ 概念</p></font>

中间件消息队列产品有 ActiveMQ、RabbitMQ、Kafka、ZeroMQ、RocketMQ

RabbitMQ 特点
```
RabbitMQ 是一个由 Erlang 语言开发的 AMQP 的开源实现

AMQP ：Advanced Message Queue，高级消息队列协议
它是应用层协议的一个开放标准，基于此协议的客户端与消息中间件可传递消息，并不受产品、开发语言等条件的限制

RabbitMQ 最初起源于金融系统，用于在分布式系统中存储转发消息，在易用性、扩展性、高可用性等方面表现不俗。具体特点包括：

1.可靠性(Reliability)
RabbitMQ 使用一些机制来保证可靠性，如持久化、传输确认、发布确认

2.灵活的路由(Flexible Routing)
在消息进入队列之前，通过 Exchange 来路由消息。对于典型的路由功能，RabbitMQ 已经提供了一些内置的 Exchange 来实现。针对更复杂的路由功能，可以将多个 Exchange 绑定在一起，也通过插件机制实现自己的 Exchange

3.消息集群(Clustering)
多个 RabbitMQ 服务器可以组成一个集群，形成一个逻辑 Broker

4.高可用(Highly Available Queues)
队列可以在集群中的机器上进行镜像，使得在部分节点出问题的情况下队列仍然可用

5.多种协议(Multi-protocol)
RabbitMQ 支持多种消息队列协议，比如 STOMP、MQTT 等等

6.多语言客户端(Many Clients)
RabbitMQ 几乎支持所有常用语言，比如 Java、.NET、Ruby 等等

7.管理界面(Management UI)
RabbitMQ 提供了一个易用的用户界面，使得用户可以监控和管理消息 Broker 的许多方面

8.跟踪机制(Tracing)
如果消息异常，RabbitMQ 提供了消息跟踪机制，使用者可以找出发生了什么

9.插件机制(Plugin System)
RabbitMQ 提供了许多插件，来从多方面进行扩展，也可以编写自己的插件
```
RabbitMQ 中的概念模型

抽象的消息模型:

![image](https://github.com/dwjlw1314/DWJ-PROJECT/raw/master/PictureSource/3.21.1.jpg)

消费者(consumer)订阅某个队列。生产者(producer)创建消息，然后发布到队列(queue)中，最后将消息发送到监听的消费者

RabbitMQ 基本概念

![image](https://github.com/dwjlw1314/DWJ-PROJECT/raw/master/PictureSource/3.21.2.jpg)
```
1.Message
消息是不具名的，它由消息头和消息体组成。消息体是不透明的，而消息头则由一系列的可选属性组成，这些属性包括routing-key(路由键)、priority(相对于其他消息的优先权)、delivery-mode(指出该消息可能需要持久性存储)等

2.Publisher
消息的生产者，也是一个向交换器发布消息的客户端应用程序

3.Exchange
交换器，用来接收生产者发送的消息并将这些消息路由给服务器中的队列

4.Binding
绑定，用于消息队列和交换器之间的关联。一个绑定就是基于路由键将交换器和消息队列连接起来的路由规则，所以可以将交换器理解成一个由绑定构成的路由表

5.Queue
消息队列，用来保存消息直到发送给消费者。它是消息的容器，也是消息的终点。一个消息可投入一个或多个队列。消息一直在队列里面，等待消费者连接到这个队列将其取走

6.Connection
网络连接，比如一个TCP连接

7.Channel
信道，多路复用连接中的一条独立的双向数据流通道。信道是建立在真实的TCP连接内地虚拟连接，AMQP 命令都是通过信道发出去的，不管是发布消息、订阅队列还是接收消息，这些动作都是通过信道完成。因为对于操作系统来说建立和销毁 TCP 都是非常昂贵的开销，所以引入了信道的概念，以复用一条 TCP 连接

8.Consumer
消息的消费者，表示一个从消息队列中取得消息的客户端应用程序

9.Virtual Host
虚拟主机，表示一批交换器、消息队列和相关对象。虚拟主机是共享相同的身份认证和加密环境的独立服务器域。每个 vhost 本质上就是一个 mini 版的 RabbitMQ 服务器，拥有自己的队列、交换器、绑定和权限机制。vhost 是 AMQP 概念的基础，必须在连接时指定，RabbitMQ 默认的 vhost 是 /

10.Broker
表示消息队列服务器实体
```

AMQP 消息路由

![image](https://github.com/dwjlw1314/DWJ-PROJECT/raw/master/PictureSource/3.21.3.jpg)

Exchange 类型，目前共四种类型：direct、fanout、topic、headers。headers 交换器和 direct 交换器完全一致，但性能差很多
```
1.direct
消息中的路由键(routing key)和 Binding 中的 binding key 一致， 交换器就将消息发到对应的队列中。路由键与队列名是完全匹配、单播的模式

2.fanout
每个发到 fanout 类型交换器的消息都会分到所有绑定的队列上去。fanout 交换器不处理路由键，只是简单的将队列绑定到交换器上，每个发送到交换器的消息都会被转发到与该交换器绑定的所有队列上。很像子网广播，每台子网内的主机都获得了一份复制的消息。fanout 类型转发消息是最快的

3.topic
topic 交换器通过模式匹配分配消息的路由键属性，将路由键和某个模式进行匹配，此时队列需要绑定到一个模式上。它将路由键和绑定键的字符串切分成单词，这些单词之间用点隔开。它同样也会识别通配符“#”意思是匹配0个或多个单词
```

<font color=#FF0000 size=5> <p align="center">RabbitMQ 安装</p></font>

准备相关软件包
```
make-4.1.tar.gz
wxWidgets-3.0.4.zip
erlang: otp-OTP-21.0.zip
simplejson: simplejson-3.16.0.tar.gz
RabbitMQ: rabbitmq-server-master.zip
```
源码安装make
```
[root@dwj ~]# tar -xvf make-4.1.tar.gz
[root@dwj make-4.1]# ./configure
[root@dwj make-4.1]# make && make install
```
源码安装wxWidgets
```
[root@dwj ~]# unzip wxWidgets-3.0.4.zip
[root@dwj ~]# cd wxWidgets-3.0.4
[root@dwj wxWidgets-3.0.4]# ./configure --with-opengl --enable-debug --enable-unicode
[root@dwj wxWidgets-3.0.4]# make && make install
```
源码安装erlang(需要自行解决一些依赖包的安装)
```
[root@dwj ~]# unzip otp-OTP-21.0.zip
[root@dwj ~]# cd otp-OTP-21.0
[root@dwj otp-OTP-21.0]# ./otp_build autoconf
[root@dwj otp-OTP-21.0]# ./configure
[root@dwj otp-OTP-21.0]# make -j4 && make install
```
源码安装simplejson
```
[root@dwj ~]# tar -xvf simplejson-3.16.0.tar.gz
[root@dwj ~]# cd simplejson-3.16.0
[root@dwj simplejson-3.16.0]# python setup.py install
```
源码安装rabbitmq-server
```
[root@dwj ~]# unzip rabbitmq-server-master.zip
[root@dwj ~]# mv rabbitmq-server-master /usr/local/rabbitmq
[root@dwj ~]# cd /usr/local/rabbitmq
[root@dwj rabbitmq]# make
```
1.如果出现 SSL connect error 或者 HTTP request failed 错误，升级nss服务(首先配置网络yum源)
>[root@dwj ~]# yum update nss

2.如果出现 elixir: Command not found 错误,安装elixir程序包
```
[root@dwj ~]# git clone https://github.com/elixir-lang/elixir.git
[root@dwj ~]# cd elixir
[root@dwj elixir]# make install
```
修改Makefile文件，添加以下插件信息
>[root@dwj rabbitmq]# vim Makefile

```
BUILD_DEPS = rabbitmq_cli syslog rabbitmq_management rabbitmq_amqp1_0 rabbitmq_tracing rabbitmq_recent_history_exchange rabbitmq_event_exchange
```
再次运行make命令，下载和构建相应插件
>[root@dwj rabbitmq]# make

生成相应版本的plugins
>[root@dwj rabbitmq]# make run-broker

编译完成后修改/etc/profile文件，末尾添加可执行程序路径环境变量:
>[root@dwj ~]# vim /etc/profile   <br>
export PATH=$PATH:/usr/local/rabbitmq/scripts

执行source /etc/profile让文件配置生效
>[root@dwj ~]# source /etc/profile

设置rabbitmq服务默认读取配置文件的路径，方式一：
>[root@dwj rabbitmq]# mkdir /etc/rabbitmq  <br>
>[root@dwj rabbitmq]# cp docs/rabbitmq.conf.example /etc/rabbitmq/rabbitmq.conf

通过环境变量设置rabbitmq服务指定配置文件路径,末尾添加环境变量，方式二：
>[root@dwj rabbitmq]# vim ~/.bash_profile

```
RABBITMQ_HOME=/usr/local/rabbitmq/
export RABBITMQ_HOME
```
执行source ~/.bash_profile让文件配置生效
>[root@dwj ~]# source ~/.bash_profile

指定目录创建配置文件
>[root@dwj rabbitmq]# mkdir $RABBITMQ_HOME/etc  <br>
>[root@dwj rabbitmq]# cp docs/rabbitmq.conf.example etc/rabbitmq.conf

如果配置文件采用方式二，则需要创建插件声明目录
>[root@dwj rabbitmq]# mkdir /etc/rabbitmq

启用management管理插件
>[root@dwj rabbitmq]# rabbitmq-plugins enable rabbitmq_management

配置完成后运行服务，如果有如下界面表示安装成功
>[root@dwj rabbitmq]# rabbitmq-server

```
  ##  ##
  ##  ##      RabbitMQ 0.0.0. Copyright (C) 2007-2018 Pivotal Software, Inc.
  ##########  Licensed under the MPL.  See http://www.rabbitmq.com/
  ######  ##
  ##########  Logs: /var/log/rabbitmq/rabbit@dwj.log
                    /var/log/rabbitmq/rabbit@dwj_upgrade.log

              Starting broker...
 completed with 3 plugins.
```
查看端口是否被监听
>[root@dwj rabbitmq]# ss -napt|grep 5672  <br>
>[root@dwj rabbitmq]# netstat -nlp | grep beam
















RabbitMQ 是用 Erlang 语言写的，在Erlang 中有两个概念：节点和应用程序。节点就是 Erlang 虚拟机的每个实例，而多个 Erlang 应用程序可以运行在同一个节点之上。节点之间可以进行本地通信(不管他们是不是运行在同一台服务器之上)。比如一个运行在节点A上的应用程序可以调用节点B上应用程序的方法，就好像调用本地函数一样。如果应用程序由于某些原因奔溃，Erlang 节点会自动尝试重启应用程序

后端运行模式
>[root@dwj rabbitmq]# rabbitmq-server –detached

关闭整个 RabbitMQ 节点
>[root@dwj rabbitmq]# rabbitmqctl stop

关闭不同的节点，包括远程节点，只需要传入参数 -n ; dwj是主机名
>[root@dwj rabbitmq]# rabbitmqctl -n rabbit@dwj stop

启动和关闭 RabbitMQ 应用程序,同时保持 Erlang 节点运行则可以用 stop_app
>[root@dwj rabbitmq]# rabbitmqctl start_app/stop_app

重置 RabbitMQ 节点,该命令将清除所有的队列
>[root@dwj rabbitmq]# rabbitmqctl reset

查看已声明的队列
>[root@dwj rabbitmq]# rabbitmqctl list_queues

查看交换器，附加参数列出了交换器的名称、类型、是否持久化、是否自动删除
>[root@dwj rabbitmq]# rabbitmqctl list_exchanges name type durable auto_delete

查看绑定
>[root@dwj rabbitmq]# rabbitmqctl list_bindings

查看监听用户
>[root@dwj rabbitmq]# rabbitmqctl list_users

如果是在一台机器上同时启动多个 RabbitMQ 节点来组建集群的话，会因为节点名称和端口冲突导致启动失败。所以在每次调用 rabbitmq-server 命令前，设置环境变量 RABBITMQ_NODENAME 和 RABBITMQ_NODE_PORT 来明确指定唯一的节点名称和端口


RABBITMQ_NODENAME=test_rabbit_1 RABBITMQ_NODE_PORT=5672 ./sbin/rabbitmq-server -detached













运行脚本顺序
MQ-install.sh  ->  MQ-policy.sh  ->  MQ-jiqun.sh;

参考：

http://blog.csdn.net/rainday0310/article/details/22082503
http://www.cnblogs.com/caca/p/rabbitmq_demo.html
