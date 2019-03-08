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

集群原理
```
rabbitmq是依据erlang的分布式特性（RabbitMQ底层是通过Erlang架构来实现的，所以rabbitmqctl会启动Erlang节点，并基于Erlang节点来使用Erlang系统连接RabbitMQ节点，在连接过程中需要正确的Erlang Cookie和节点名称，Erlang节点通过交换Erlang Cookie以获得认证）来实现的，所以部署rabbitmq分布式集群时要先安装erlang，并把其中一个服务的cookie复制到另外的节点

rabbitmq集群中，各个rabbitmq为对等节点，即每个节点均提供给客户端连接，进行消息的接收和发送。节点分为内存节点和磁盘节点，一般的，均应建立为磁盘节点，为了防止机器重启后的消息消失

RabbitMQ的Cluster集群模式一般分为两种，普通模式和镜像模式。消息队列通过rabbitmq HA镜像队列进行消息队列实体复制

普通模式下，以两个节点(rabbit01、rabbit02)为例来进行说明。对于Queue来说，消息实体只存在于其中一个节点rabbit01(或者rabbit02)，rabbit01和rabbit02两个节点仅有相同的元数据，即队列的结构。当消息进入rabbit01节点的Queue后，consumer从rabbit02节点消费时，RabbitMQ会临时在rabbit01、rabbit02间进行消息传输，把A中的消息实体取出并经过B发送给consumer。所以consumer应尽量连接每一个节点，从中取消息。即对于同一个逻辑队列，要在多个节点建立物理Queue。否则无论consumer连rabbit01或rabbit02，出口总在rabbit01，会产生瓶颈

镜像模式下，将需要消费的队列变为镜像队列，存在于多个节点，这样就可以实现RabbitMQ的HA高可用性。作用就是消息实体会主动在镜像节点之间实现同步，而不是像普通模式那样，在consumer消费数据时临时读取。缺点就是，集群内部的同步通讯会占用大量的网络带宽
```

<font color=#FF0000 size=5> <p align="center">RabbitMQ 安装</p></font>

准备相关软件包
```
make: make-4.1.tar.gz
wxWidgets: wxWidgets-3.0.4.zip
erlang: otp-OTP-21.0.zip
simplejson: simplejson-3.16.0.tar.gz
RabbitMQ: rabbitmq-server-master.zip
```
源码安装make
```
[root@dwj ~]# tar -xvf make-4.1.tar.gz
[root@dwj make-4.1]# ./configure
[root@dwj make-4.1]# make && make install
[root@dwj make-4.1]# make --version
  GNU Make 4.1
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
[root@dwj otp-OTP-21.0]# yum -y install unixODBC unixODBC-devel
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
1.如果出现 SSL connect error 或者 HTTP request failed 错误，升级相关服务(首先配置网络yum源)
>[root@dwj ~]# yum update nss curl libcurl

2.如果出现 elixir: Command not found 错误,安装elixir程序包
```
[root@dwj ~]# git clone https://github.com/elixir-lang/elixir.git
[root@dwj ~]# cd elixir
[root@dwj elixir]# make install
[root@dwj elixir]# elixir --version
  Elixir 1.7.4 (compiled with Erlang/OTP 21)
```
3.修改Makefile文件，添加以下插件信息
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
>[root@dwj rabbitmq]# cp docs/rabbitmq.conf.example etc/rabbitmq/rabbitmq.conf

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
查看rabbitmq安装版本
>[root@dwj rabbitmq]# rabbitmqctl status

查看端口是否被监听
>[root@dwj rabbitmq]# ss -napt|grep 5672  <br>
>[root@dwj rabbitmq]# netstat -nlp | grep beam

验证rabbitmq_management功能，本地浏览器输入：localhost:15672

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
>[root@dwj rabbitmq]# RABBITMQ_NODENAME=test_rabbit_1 RABBITMQ_NODE_PORT=5672 rabbitmq-server -detached

<font color=#FF0000 size=5> <p align="center">RabbitMQ 集群安装</p></font>

1、环境介绍
```
rabbitmq1 192.168.0.111
rabbitmq2 192.168.0.112
```
分别在2台机器上配置/etc/hosts,添加以下内容
```
192.168.0.111 rabbitmq1
192.168.0.112 rabbitmq2
```

2.按照上面单机部署方式在两台服务器安装rabbitmq程序

3.设置erlang
```
找到.erlang cookie文件的位置，一般会存在这两个地址：
第一个地方是$HOME/.erlang.cookie
第二个地方是/var/lib/rabbitmq/
如果使用解压缩方式安装部署的rabbitmq，那么这个文件会在$HOME目录下
如果使用rpm等安装包方式进行安装的，那么这个文件会在/var/lib/rabbitmq目录下

#这里将rabbitmq1的该文件复制到rabbitmq2，这个文件的默认权限是400，因此采用scp的方式只拷贝内容即可
[root@rabbitmq1 ~]# scp /root/.erlang.cookie rabbitmq2:/root/

#查看两台机器的cookie是否一致，设置erlang的目的是要保证集群内的cookie内容一致
[root@rabbitmq1 ~]# cat $HOME/.erlang.cookie
  BSFXFRJMPKEDRGBSDFEY
```

4.使用 -detached 参数运行各节点
>[root@rabbitmq1 ~]# rabbitmq-server -detached

5.目前服务是以单独的RabbitMQ存在的，可以正常提供服务了，接下来把所有节点组成集群
```
[root@rabbitmq1 ~]# rabbitmqctl stop_app
#以磁盘节点模式加入集群，这里集群的名字一定不能写错
[root@rabbitmq1 ~]# rabbitmqctl join_cluster rabbit@rabbitmq2
#如果要使用内存节点，则可以使用以下命令加入集群
[root@rabbitmq1 ~]# rabbitmqctl join_cluster --ram rabbit@rabbitmq2
[root@rabbitmq1 ~]# rabbitmqctl start_app
```
在任意节点上来查看是否集群配置成功
>[root@rabbitmq1 ~]# rabbitmqctl cluster_status

6.由于guest这个用户,只能在本地访问,所以新增一个远程用户并赋予权限:
```
#添加用户并设置密码
[root@rabbitmq1 ~]# rabbitmqctl add_user admin admin
#添加权限(使admin用户对虚拟主机“/” 具有所有权限)
[root@rabbitmq1 ~]# rabbitmqctl set_permissions -p "/" admin ".*" ".*" ".*"
#修改用户角色(加入administrator用户组)
[root@rabbitmq1 ~]# rabbitmqctl set_user_tags admin administrator
```

6.设置镜像队列策略,可以使用原有默认的虚拟主机'/'
```
新建virtual_host
[root@rabbitmq1 ~]# rabbitmqctl add_vhost gjsy
撤销virtual_host
[root@rabbitmq1 ~]# rabbitmqctl delete_vhost gjsy
同时给对应用户“admin”和“guest”均加上权限，可以在页面直接设置
给virtual_host设置策略
[root@rabbitmq1 ~]# rabbitmqctl set_policy -p gjsy ha-all "^" '{"ha-mode":"all"}'
"gjsy" vhost名称，"^"匹配所有的队列，ha-all 策略名ha-all, '{"ha-mode":"all"}' 策略模式即复制到所有节点，包含新增节点
```

7.配置文件字段描述

Key | Documentation
---|---
tcp_listeners | 用于监听 AMQP连接的端口列表(无SSL). 可以包含整数 (即"监听所有接口")或者元组如 {"127.0.0.1", 5672} 用于监听一个或多个接口，Default: [5672]
num_tcp_acceptors | 接受TCP侦听器连接的Erlang进程数，Default: 10
vm_memory_high_watermark | 流程控制触发的内存阀值．相看memory-based flow control 文档，Default: 0.4
vm_memory_high_watermark_paging_ratio | 高水位限制的分数，当达到阀值时，队列中消息消息会转移到磁盘上以释放内存. 参考memory-based flow control 文档，Default: 0.5
disk_free_limit | RabbitMQ存储数据分区的可用磁盘空间限制．当可用空间值低于阀值时，流程控制将被触发.此值可根据RAM的总大小来相对设置 (如.{mem_relative, 1.0}).此值也可以设为整数(单位为bytes)或者使用数字单位(如．"50MB").默认情况下，可用磁盘空间必须超过50MB.参考 Disk Alarms 文档，Default: 50000000
log_levels | 控制日志的粒度.其值是日志事件类别(category)和日志级别(level)成对的列表，level 可以是 'none' (不记录日志事件), 'error' (只记录错误), 'warning' (只记录错误和警告), 'info' (记录错误，警告和信息), or 'debug' (记录错误，警告，信息以及调试信息)。目前定义了４种日志类别. 它们是：<br>channel - 针对所有与AMQP channels相关的事件<br>connection - 针对所有与网络连接相关的事件<br>federation - 针对所有与federation相关的事件<br>mirroring - 针对所有与 mirrored queues相关的事件<br>Default: [{connection, info}]
frame_max | 与客户端协商的允许最大frame大小. 设置为0表示无限制。设置较大的值可以提高吞吐量;设置一个较小的值可能会提高延迟，Default: 131072
channel_max | 与客户端协商的允许最大chanel大小. 设置为0表示无限制．该数值越大，则broker使用的内存就越高，Default: 0
channel_operation_timeout | Channel 操作超时时间(毫秒为单位)(内部使用，因为消息协议的区别和限制，不暴露给客户端)，Default: 5000
heartbeat | 表示心跳延迟(单位为秒) ，服务器将在connection.tune frame中发送.如果设置为0, 心跳将被禁用. 客户端可以不用遵循服务器的建议，Default: 60 (3.5.5之前的版本是580)
default_vhost | 当RabbitMQ从头开始创建数据库时创建的虚拟主机. amq.rabbitmq.log交换器会存在于这个虚拟主机中，Default: <<"/">>
default_user | RabbitMQ从头开始创建数据库时，创建的用户名，Default: <<"guest">>
default_pass | 默认用户的密码，Default: <<"guest">>
default_user_tags | 默认用户的Tags，Default: [administrator]
default_permissions | 创建用户时分配给它的默认Permissions，Default: .*
loopback_users | 只能通过环回接口(即localhost)连接broker的用户列表，如果你希望默认的guest用户能远程连接,你必须将其修改为[]，Default: [<<"guest">>]
collect_statistics | 统计收集模式。主要与管理插件相关。选项：<br>none (不发出统计事件)<br>coarse (发出每个队列 /每个通道 /每个连接的统计事件)<br>fine (也发出每个消息统计事件)<br>Default: none
collect_statistics_interval | 统计收集时间间隔(毫秒为单位)。主要针对于 management plugin，Default: 5000

8.参考：

http://blog.csdn.net/rainday0310/article/details/22082503
http://www.cnblogs.com/caca/p/rabbitmq_demo.html
