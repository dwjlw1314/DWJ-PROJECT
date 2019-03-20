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

ZMQ提供进程内、进程间、机器间、广播等四种通信协议，配置简单，用类似于URL形式的字符串指定即可，格式分别为inproc://、ipc://、tcp://、pgm://。pgm和epgm传输方式只能被ZMQ_PUB和ZMQ_SUB两种socket使用

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
ZMQ_IO_THREADS: 如果仅使用Inproc传输消息，则设置为零，否则至少将其设置为1个。必须在context创建任何套接字之前设置
int zmq_ctx_set (void *context, int option_name, int option_value);
ZMQ_BLOCKY: 如果context终止时阻塞，则返回1；不阻塞则返回0
ZMQ_IO_THREADS: 参数返回context的ZMQ线程池的大小
ZMQ_MAX_SOCKETS: 参数返回允许创建socket数量的最大值
ZMQ_MAX_MSGSZ: 参数返回context允许的最大消息大小，默认值为int_max
ZMQ_SOCKET_LIMIT: 参数返回接受的最大套接字数
int zmq_ctx_get (void *context, int option_name);
销毁context
int zmq_ctx_destroy (void *context);
int zmq_ctx_shutdown(void *context);
环境上下文的终止过程会按下列步骤进行：
● context创建的socket的任何阻塞调用都会立刻返回错误代码ETERM。所有对基于context的更深层次的操作都会失败并返回错误代码ETERM
● 中断所有的阻塞调用后，函数会进入阻塞状态，直到满足下列条件：所有基于context创建的scoekt都已经被zmq_close()函数关闭
● context上的每一个socket来说，所有被应用进程使用zmq_send()发送的消息必须被真实的发送到了网络上，或者socket设置ZMQ_LINGER 的超时时间已到
int zmq_ctx_term(void *context);
```

ZMQ Sockets是代表异步消息队列的一个抽象，这里的socket和POSIX套接字的socket不是一回事，ZMQ封装了物理连接的底层细节，对用户不透明。传统的POSIX套接字只能支持单一连接，而ZMQ socket支持多个Client的并发连接，甚至在没有任何对端的情况下，也能放入消息
ZMQ sockets不是线程安全的，因此，不建议在多个线程中并行操作同一个sockets
```c
创建ZMQ Sockets,在bind之前还不能使用
void *zmq_socket (void *context, int type);
设置socket选项，该函数可以为IPC和TCP连接提供加密机制，zmq_null(不加密)、zmq_plain(用户名/密码授权)、zmq_curve(椭圆加密)
int zmq_getsockopt (void *socket, int option_name, void *option_value, size_t *option_len);
int zmq_setsockopt (void *socket, int option_name, const void *option_value, size_t option_len);
```
type参数含义:

pattern | type | Compatible sockets | description
---|---|---|---
一对一结对模型 | ZMQ_PAIR | ZMQ_PAIR | 通过inproc方式用在进程内部通信，不会自动进行重连
请求应答模型 | ZMQ_REQ | ZMQ_REP,ZMQ_ROUTER | client端使用，阻塞且不丢弃
请求应答模型 | ZMQ_REP | ZMQ_REQ,ZMQ_DEALER | server端使用，丢弃且不阻塞
请求应答模型 | ZMQ_DEALER | ZMQ_ROUTER,ZMQ_REP,ZMQ_DEALER | 将消息以轮询的方式分发给所有对端，阻塞且不丢弃
请求应答模型 | ZMQ_ROUTER | ZMQ_DEALER,ZMQ_REQ,ZMQ_ROUTER | 将以公平的方式接收所有对端消息，丢弃且不阻塞
发布订阅模型 | ZMQ_PUB | ZMQ_SUB,ZMQ_XSUB | publisher端使用，丢弃且不阻塞
发布订阅模型 | ZMQ_XPUB | ZMQ_SUB,ZMQ_XSUB | 可以接收来来自订阅者的订阅信息
发布订阅模型 | ZMQ_SUB | ZMQ_PUB,ZMQ_XPUB | subscriber端使用，通过设置ZMQ_SUBSRIBE选项来指定需要订阅的消息
发布订阅模型 | ZMQ_XSUB | ZMQ_PUB,ZMQ_XPUB | 可以发送订阅信息给发布者来订阅
管道模型 | ZMQ_PUSH | ZMQ_PULL | push端使用，阻塞且不丢弃
管道模型 | ZMQ_PULL | ZMQ_PUSH | pull端使用，阻塞且不丢弃
原生模型 | ZMQ_STREAM | \ | \

bind函数是将socket绑定到本地的端点(endpoint)。而connect函数连接到指定的peer端点，socket会进入普通ready状态。成功调用并不意味着连接已经真实的建立，但是使用inproc://传输的时候必须在调用zmq_connect()之前执行zmq_bind()
```c
int zmq_bind (void *socket, const char *endpoint);
函数会将socket参数与绑定的endpoint指定的终结点解除绑定
int zmq_unbind (void *socket, const char *endpoint);
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

socket事件监控,函数会产生一个PAIR类型的socket，用来把socket状态改变通过inproc://传输方式发送到endpoint终结点上，
消息包含2帧，第1帧包含事件id和事件值，组织方式是16 bit的ID和32 bit的事件值。第2帧表示受影响的endpoint
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
ZMQ_EVENT_DISCONNECTED | 会话(tcp/ipc)中断

zmq_socket_monitor使用的一个示例：
```c
// Read one event off the monitor socket; return value and address
// by reference, if not null, and event number by value. Returns -1 in case of error.
static int get_monitor_event (void *monitor, int *value, char **address)
{
	// First frame in message contains event number and value
	zmq_msg_t msg;
	zmq_msg_init (&msg);
	if (zmq_msg_recv (&msg, monitor, 0) == -1)
		return -1; // Interrupted, presumably
	assert (zmq_msg_more (&msg));

	uint8_t *data = (uint8_t *)zmq_msg_data (&msg);
	uint16_t event = *(uint16_t *) (data);
	if (value) *value = *(uint32_t *) (data + 2);

	// Second frame in message contains event address
	zmq_msg_init (&msg);
	if (zmq_msg_recv (&msg, monitor, 0) == -1)
		return -1; // Interrupted, presumably
	assert (!zmq_msg_more (&msg));

	if (address) {
		uint8_t *data = (uint8_t *) zmq_msg_data (&msg);
		size_t size = zmq_msg_size (&msg);
		*address = (char *) malloc (size + 1);
		memcpy (*address, data, size);
		(*address)[size] = 0;
	}
	return event;
}

int main (void)
{
	void *ctx = zmq_ctx_new ();
	// We'll monitor these two sockets
	void *client = zmq_socket (ctx, ZMQ_DEALER);
	void *server = zmq_socket (ctx, ZMQ_DEALER);

	// Socket monitoring only works over inproc://
	int rc = zmq_socket_monitor (client, "tcp://127.0.0.1:9999", 0);
	assert (rc == -1 || zmq_errno () == EPROTONOSUPPORT);

	// Monitor all events on client and server sockets
	rc = zmq_socket_monitor (client, "inproc://monitor-client", ZMQ_EVENT_ALL);
	rc = zmq_socket_monitor (server, "inproc://monitor-server", ZMQ_EVENT_ALL);

	// Create two sockets for collecting monitor events
	void *client_mon = zmq_socket (ctx, ZMQ_PAIR);
	void *server_mon = zmq_socket (ctx, ZMQ_PAIR);

	// Connect these to the inproc endpoints so they'll get events
	rc = zmq_connect (client_mon, "inproc://monitor-client");
	rc = zmq_connect (server_mon, "inproc://monitor-server");

	// Now do a basic ping test
	rc = zmq_bind (server, "tcp://127.0.0.1:9998");
	rc = zmq_connect (client, "tcp://127.0.0.1:9998");
	bounce (client, server);

	// Close client and server
	close_zero_linger (client);
	close_zero_linger (server);

	// Now collect and check events from both sockets
  int event = get_monitor_event (client_mon, NULL, NULL);
  if (event == ZMQ_EVENT_CONNECT_DELAYED)
  event = get_monitor_event (client_mon, NULL, NULL);
  assert (event == ZMQ_EVENT_CONNECTED);
  event = get_monitor_event (client_mon, NULL, NULL);
  assert (event == ZMQ_EVENT_MONITOR_STOPPED);

  // This is the flow of server events
  event = get_monitor_event (server_mon, NULL, NULL);
  assert (event == ZMQ_EVENT_LISTENING);
  event = get_monitor_event (server_mon, NULL, NULL);
  assert (event == ZMQ_EVENT_ACCEPTED);
  event = get_monitor_event (server_mon, NULL, NULL);
  assert (event == ZMQ_EVENT_CLOSED);
  event = get_monitor_event (server_mon, NULL, NULL);
  assert (event == ZMQ_EVENT_MONITOR_STOPPED);

	// Close down the sockets
	close_zero_linger (client_mon);
	close_zero_linger (server_mon);
	zmq_ctx_term (ctx);

	return 0;
}
```*

socket关闭函数，程序中通过zmq_recv()函数接收但没有被应用程序使用的消息都将会被丢弃。已经使用zmq_send()发送的消息但是还没有被设备发送到peer的处理方式，将由socket的ZMQ_LINGER值决定。默认是不丢弃，但当调用zmq_ctx_term()函数时会导致应用程序阻塞
```c
关闭参数指定的socket，zmq_connect()函数无法重用套接字
int zmq_close (void *socket);
断开socket参数与endpoint参数指定的终结点的连接，zmq_connect()函数可以重用套接字
int zmq_disconnect (void *socket, const char *addr);
```

I/O多路复用,对sockets集合的I/O多路复用，使用水平触发
```c
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
```*

ZMQ每一条消息都是在消息队列(进程内部或跨进程)中进行传输的数据单元，ZMQ消息本身没有数据结构，因此支持任意类型的数据，
这完全依赖于自定义消息的数据结构。一条ZMQ消息可以包含多个消息片，每个消息片都是一个独立zmq_msg_t结构。
ZMQ保证以原子方式传递消息，要么所有消息片都发送成功，要么都不成功
```
初始化消息
zmq_msg_init()函数初始化成一个空的消息对象，可以通过zmq_msg_开头一类的函数来访问该对象
int zmq_msg_init (zmq_msg_t *msg);
从一个指定的存储空间中初始化一个ZMQ消息对象的数据；用data和size参数对msg指定的ZMQ消息对象的内容进行初始化。ZMQ不会执行拷贝操作，并且ZMQ会取得指定数据的拥有权。还提供了回收功能函数zff，将会在data数据不再使用的时候被ZMQ调用一次，ZMQ会将函数中的参数data和hint参数传递给zff函数
typedef void (zmq_free_fn) (void *data, void *hint);
int zmq_msg_init_data (zmq_msg_t *msg, void *data, size_t size, zmq_free_fn *zff, void *hint);
使用一个指定的空间大小初始化ZMQ消息对象，函数执行的时候会选择是否把消息存储在栈里面(小消息)，还是堆里面(大消息)，考虑到性能原因，zmq_msg_init_size()函数不会清除消息数据
int zmq_msg_init_size (zmq_msg_t *msg, size_t size);

zmq_msg_init()、zmq_msg_init_data()、zmq_msg_init_size() 三个函数是互斥的，每次使用其中一个即可

设置和获取消息属性
int zmq_msg_get (zmq_msg_t *message, int property);
int zmq_msg_set (zmq_msg_t *message, int property, int value);

释放消息
int zmq_msg_close (zmq_msg_t *msg);

收发消息帧,flags参数由下列标志组合(|)而成:
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

函数把src对象中的内容复制到dest指定的消息对象中，属于浅拷贝。如果dest中有数据的话，原有数据将被释放
int zmq_msg_copy (zmq_msg_t *dest, zmq_msg_t *src);
函数把src对象里面的内容移动到dest指定的消息对象里面，src变为一个空消息。dest原有数据将被释放
int zmq_msg_move (zmq_msg_t *dest, zmq_msg_t *src);
```

错误处理，ZMQ库使用POSIX处理函数错误，返回NULL指针或者负数时表示调用出错
```c
函数返回当前线程的错误码errno变量的值
int zmq_errno (void);
函数将错误映射成错误字符串
const char *zmq_strerror (int errnum);
```

ZMQ提供代理功能，代理可以在前端和后端的socket之间转发消息
```c
共享队列(shared queue)，前端是ZMQ_ROUTER socket，后端是ZMQ_DEALER socket，代理会扮演一个共享队列的角色，从多个客户端接收消息，并且把这些消息公平的分发到服务端。这些请求会被前端公平的放置在队列里进行接收，并通过后端进行均衡的分发
转发队列(forwarded)，前端是ZMQ_XSUB socket, 后端是ZMQ_XPUB socket，代理会把从前端收到的消息转发给后端
流(streamer)，前端是ZMQ_PULL socket, 后端是ZMQ_PUSH socket，代理会从后端收集消息并发送给前端使用管道传输模式的工作端
int zmq_proxy (const void *frontend, const void *backend, const void *capture);
int zmq_proxy_steerable (const void *frontend, const void *backend, const void *capture, const void *control);
```

proxy使用的一个示例:
```c
// Create frontend and backend sockets
void *frontend = zmq_socket (context, ZMQ_ROUTER);
assert (backend);
void *backend = zmq_socket (context, ZMQ_DEALER);
assert (frontend);

// Bind both sockets to TCP ports
assert (zmq_bind (frontend, "tcp://*:8080") == 0);
assert (zmq_bind (backend, "tcp://*:9090") == 0);
// Start the queue proxy, which runs until ETERM
zmq_proxy(frontend, backend, NULL);
```

zmq_proxy_steerable使用的一个示例：
```
创建一个共享的代理队列
//  Create frontend, backend and control sockets
void *frontend = zmq_socket (context, ZMQ_ROUTER);
void *backend = zmq_socket (context, ZMQ_DEALER);
void *control = zmq_socket (context, ZMQ_SUB);
//  Bind sockets to TCP ports
assert (zmq_bind (frontend, "tcp://*:5555") == 0);
assert (zmq_bind (backend, "tcp://*:5556") == 0);
assert (zmq_connect (control, "tcp://*:5557") == 0);
// Subscribe to the control socket since we have chosen SUB here
assert (zmq_setsockopt (control, ZMQ_SUBSCRIBE, "", 0));
//  Start the queue proxy, which runs until ETERM or "TERMINATE" received on the control socket
zmq_proxy_steerable (frontend, backend, NULL, control);

在另一个节点上创建一个控制器，进程或者其它
void *control = zmq_socket (context, ZMQ_PUB);
assert (zmq_bind (control, "tcp://*:5557") == 0);
// stop the proxy
assert (zmq_send (control, "STOP", 5, 0) == 0);
// resume the proxy
assert (zmq_send (control, "RESUME", 7, 0) == 0);
// terminate the proxy
assert (zmq_send (control, "TERMINATE", 10, 0) == 0);
```

利用zmq_stream创建一个简单的http服务器
```
void *ctx = zmq_ctx_new ();
assert (ctx);
/* Create ZMQ_STREAM socket */
void *socket = zmq_socket (ctx, ZMQ_STREAM);
assert (socket);
int rc = zmq_bind (socket, "tcp://*:8080");
assert (rc == 0);
/* Data structure to hold the ZMQ_STREAM ID */
uint8_t id [256];
size_t id_size = 256;
while (1)
{
    /* Get HTTP request; ID frame and then request */
    id_size = zmq_recv (socket, id, 256, 0);
    assert (id_size > 0);
    /* Prepares the response */
    char http_response [] =
    "HTTP/1.0 200 OK\r\n"
    "Content-Type: text/plain\r\n"
    "\r\n"
    "Hello, World!";
    /* Sends the ID frame followed by the response */
    zmq_send (socket, id, id_size, ZMQ_SNDMORE);
    zmq_send (socket, http_response, strlen (http_response), ZMQ_SNDMORE);
    /* Closes the connection by sending the ID frame followed by a zero response */
    zmq_send (socket, id, id_size, ZMQ_SNDMORE);
    zmq_send (socket, 0, 0, ZMQ_SNDMORE);
    /* NOTE: If we don't use ZMQ_SNDMORE, then we won't be able to send more */
    /* message to any client */
}
zmq_close (socket);
zmq_ctx_destroy (ctx);
```
