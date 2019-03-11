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
[root@dwj zeromq-4.2.5]# make -j 4
[root@dwj zeromq-4.2.5]# make check && make install
```

<font color=#FF0000 size=5> <p align="center">ZeroMQ 代码模型</p></font>

ZMQ提供的所有API均以zmq_开头，包含头文件 #include <zmq.h>
> gcc [flags] files -lzmq [libraries]

获取当前ZMQ库的版本信息
```c
void zmq_version (int *major, int *minor, int *patch);
```

使用ZQM库函数之前先创建ZMQ context，ZMQ context是线程安全的，可以在多线程环境使用。
在一个进程中，可以有多个ZMQ context并存，程序终止时，也需要销毁context
```c
创建context
void *zmq_ctx_new ();
设置context选项
int zmq_ctx_set (void *context, int option_name, int option_value);
int zmq_ctx_get (void *context, int option_name);
销毁context
int zmq_ctx_term (void *context);
```

ZMQ Sockets是代表异步消息队列的一个抽象，这里的socket和POSIX套接字的socket不是一回事，ZMQ封装了物理连接的底层细节，对用户不透明。传统的POSIX套接字只能支持单一连接，而ZMQ socket支持多个Client的并发连接，甚至在没有任何对端的情况下，也能放入消息
ZMQ sockets不是线程安全的，因此，不建议在多个线程中并行操作同一个sockets
```c
创建ZMQ Sockets,在bind之前还不能使用
void *zmq_socket (void *context, int type);
设置socket选项，该函数可以为IPC和TCP连接提供加密机制，zmq_null(不加密)、zmq_plain(用户名/密码授权)、zmq_curve(椭圆加密)
int zmq_getsockopt (void *socket, int option_name, void *option_value, size_t *option_len);
int zmq_setsockopt (void *socket, int option_name, const void *option_value, size_t option_len);
关闭socket
int zmq_close (void *socket);
```
type参数含义:

pattern | type | description
---|---|---
一对一结对模型 | ZMQ_PAIR | 通过inproc方式用在进程内部通信
请求应答模型 | ZMQ_REQ | client端使用，阻塞且不丢弃
请求应答模型 | ZMQ_REP | server端使用，丢弃且不阻塞
请求应答模型 | ZMQ_DEALER | 将消息以轮询的方式分发给所有对端，阻塞且不丢弃
请求应答模型 | ZMQ_ROUTER | 将以公平的方式接收所有对端消息，丢弃且不阻塞
发布订阅模型 | ZMQ_PUB  | publisher端使用，丢弃且不阻塞
发布订阅模型 | ZMQ_XPUB | 可以接收来来自订阅者的订阅信息
发布订阅模型 | ZMQ_SUB  | subscriber端使用，通过设置ZMQ_SUBSRIBE选项来指定需要订阅的消息
发布订阅模型 | ZMQ_XSUB | 可以发送订阅信息给发布者来订阅
管道模型 | ZMQ_PUSH | push端使用，阻塞且不丢弃
管道模型 | ZMQ_PULL | pull端使用，阻塞且不丢弃
原生模型 | ZMQ_STREAM |

bind函数是将socket绑定到本地的端点(endpoint)，而connect函数连接到指定的peer端点
```c
int zmq_bind (void *socket, const char *endpoint);
int zmq_connect (void *socket, const char *endpoint);
```
endpoint支持的类型：

transports | description | uri example
---|---|---
zmp_tcp | TCP的单播通信 | tcp://
zmp_ipc | 本地进程间通信 | ipc://
zmp_inproc | 本地线程间通信 | inproc://
zmp_pgm | PGM广播通信 | pgm://

收发消息
```c
函数将指定buf的指定长度len的字节写入队列，函数返回值是发送的字节数，返回-1表示出错
int zmq_send (void *socket, void *buf, size_t len, int flags);
函数的len参数指定接收buf的最大长度，超出部分会被截断，函数返回的值是接收到的字节数，返回-1表示出错
int zmq_recv (void *socket, void *buf, size_t len, int flags);
函数表示发送的buf是一个常量内存区（constant-memory），这块内存不需要复制、释放
int zmq_send_const (void *socket, void *buf, size_t len, int flags);
```

socket事件监控,下面函数会生成一对sockets，publishers端通过inproc://协议发布sockets状态改变的events，
消息包含2帧，第1帧包含events id和关联值，第2帧表示受影响的endpoint
```c
int zmq_socket_monitor (void *socket, char **addr, int events);
```
监控支持的events：

EventType | Desc
---|---
ZMQ_EVENT_CONNECTED | 建立连接
ZMQ_EVENT_CONNECT_DELAYED | 连接失败
ZMQ_EVENT_CONNECT_RETRIED | 异步连接/重连
ZMQ_EVENT_LISTENING | bind到端点
ZMQ_EVENT_BIND_FAILED | bind失败
ZMQ_EVENT_ACCEPTED | 接收请求
ZMQ_EVENT_ACCEPT_FAILED | 接收请求失败
ZMQ_EVENT_CLOSED | 关闭连接
ZMQ_EVENT_CLOSE_FAILED | 关闭连接失败
ZMQ_EVENT_DISCONNECTED | 会话（tcp/ipc）中断

I/O多路复用,对sockets集合的I/O多路复用，使用水平触发
```
参数items指定一个结构体数组，nitems指定数组的元素个数，timeout参数是超时时间(0表示立即返回，-1表示阻塞等待)
int zmq_poll (zmq_pollitem_t *items, int nitems, long timeout);

typedef struct
{
    void *socket;
    int fd;
    short events;
    short revents;
} zmq_pollitem_t;

对于每个zmq_pollitem_t元素，ZMQ会同时检查其socket(ZMQ套接字)和fd(原生套接字)上是否有指定的events发生，且ZMQ套接字优先
events指定该sockets需要关注的事件，revents返回该sockets已发生的事件。它们的取值为：
ZMQ_POLLIN(可读)，ZMQ_POLLOUT(可写)，ZMQ_POLLERR(出错)
```

ZMQ每一条消息都是在消息队列(进程内部或跨进程)中进行传输的数据单元，ZMQ消息本身没有数据结构，因此支持任意类型的数据，
这完全依赖于自定义消息的数据结构。一条ZMQ消息可以包含多个消息片，每个消息片都是一个独立zmq_msg_t结构。
ZMQ保证以原子方式传递消息，要么所有消息片都发送成功，要么都不成功
```
初始化消息
typedef void (zmq_free_fn) (void *data, void *hint);
zmq_msg_init()函数初始化一个消息对象zmq_msg_t ，不要直接访问zmq_msg_t对象，可以通过zmq_msg_开头一类的函数来访问它
int zmq_msg_init (zmq_msg_t *msg);
int zmq_msg_init_data (zmq_msg_t *msg, void *data, size_t size, zmq_free_fn *ffn, void *hint);
int zmq_msg_init_size (zmq_msg_t *msg, size_t size);
zmq_msg_init()、zmq_msg_init_data()、zmq_msg_init_size() 三个函数是互斥的，每次使用其中一个即可

设置消息属性
int zmq_msg_get (zmq_msg_t *message, int property);
int zmq_msg_set (zmq_msg_t *message, int property, int value);

释放消息
int zmq_msg_close (zmq_msg_t *msg);

收发消息,flags参数如下：
ZMQ_DONTWAIT，非阻塞模式，如果没有可用的消息，将errno设置为EAGAIN
ZMQ_SNDMORE，发送多个消息片时，除了最后一个外，其它每个消息片都必须使用ZMQ_SNDMORE标记位
int zmq_msg_send (zmq_msg_t *msg, void *socket, int flags);
int zmq_msg_recv (zmq_msg_t *msg, void *socket, int flags);

获取消息内容，返回指向消息对象所含内容的指针
void *zmq_msg_data (zmq_msg_t *msg);
标识该消息片是否是整个消息的一部分，是否还有更多的消息片待接收
int zmq_msg_more (zmq_msg_t *message);
返回消息的字节数
size_t zmq_msg_size (zmq_msg_t *msg);

控制消息，函数实现的是浅拷贝
int zmq_msg_copy (zmq_msg_t *dest, zmq_msg_t *src);
函数中，将dst指向src消息，然后src被置空
int zmq_msg_move (zmq_msg_t *dest, zmq_msg_t *src);
```

错误处理，ZMQ库使用POSIX处理函数错误，返回NULL指针或者负数时表示调用出错
```c
函数返回当前线程的错误码errno变量的值
int zmq_errno (void);
函数将错误映射成错误字符串
const char *zmq_strerror (int errnum);
```

ZMQ提供代理功能，代理可以在前端socket和后端socket之间转发消息
```c
共享队列(shared queue)，前端是ZMQ_ROUTER socket，后端是ZMQ_DEALER socket，proxy会把clients发来的请求，公平地分发给services
转发队列(forwarded)，前端是ZMQ_XSUB socket, 后端是ZMQ_XPUB socket, proxy会把从publishers收到的消息转发给所有的subscribers
流(streamer)，前端是ZMQ_PULL socket, 后端是ZMQ_PUSH socket
int zmq_proxy (const void *frontend, const void *backend, const void *capture);
int zmq_proxy_steerable (const void *frontend, const void *backend, const void *capture, const void *control);
```

proxy使用的一个示例：
```c
// Create frontend and backend sockets
void *frontend = zmq_socket (context, ZMQ_ROUTER);
assert (backend);
void *backend = zmq_socket (context, ZMQ_DEALER);
assert (frontend);

// Bind both sockets to TCP ports
assert (zmq_bind (frontend, "tcp://*:8080") == 0);
assert (zmq_bind (backend, "tcp://*:9090") == 0);

//  Start the queue proxy, which runs until ETERM zmq_proxy(frontend, backend, NULL);
```
