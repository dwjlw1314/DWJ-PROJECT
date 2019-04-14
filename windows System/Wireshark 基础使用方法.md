Wireshark是世界上最流行的网络分析工具。这个强大的工具可以捕捉网络中的数据，并为用户提供关于网络和上层协议的各种信息。与很多其他网络工具一样，Wireshark也使用pcap network library来进行封包捕捉。wireshark的原名是Ethereal

![image](https://github.com/dwjlw1314/DWJ-PROJECT/raw/master/PictureSource/2.2.1.png)

显示过滤器用于查找捕捉记录中的内容,请不要将捕捉过滤器和显示过滤器的概念相混淆
![image](https://github.com/dwjlw1314/DWJ-PROJECT/raw/master/PictureSource/2.2.2.png)
![image](https://github.com/dwjlw1314/DWJ-PROJECT/raw/master/PictureSource/2.2.3.png)

封包列表中显示所有已经捕获的封包。可以看到发送或接收方的MAC/IP地址，TCP/UDP端口号，协议或者封包的内容

如果捕获的是一个OSI layer 2的封包，在Source(来源)和Destination(目的地)列中将看到MAC地址，当然，此时Port(端口)列将会为空
如果捕获的是一个OSI layer 3或者更高层的封包，您在Source(来源)和Destination(目的地)列中看到的将是IP地址。Port(端口)列仅会在这个封包属于第4或者更高层时才会显示

![image](https://github.com/dwjlw1314/DWJ-PROJECT/raw/master/PictureSource/2.2.4.png) <br>
这里显示的是在封包列表中被选中项目的详细信息

“解析器”在Wireshark中也被叫做“16进制数据查看面板”。这里显示的内容与“封包详细信息”中相同，只是改为以16进制的格式表述。我们在“封包详细信息”中选择查看TCP端口（80），其对应的16进制数据将自动显示在下面的面板中(0050)

捕捉过滤器：用于决定将什么样的信息记录在捕捉结果中。需要在开始捕捉前设置。  <br>
显示过滤器：在捕捉结果中进行详细查找。他们可以在得到捕捉结果后随意修改。

两种过滤器使用的语法是完全不同的
<p align="center">1.捕捉过滤器</p>  
捕捉过滤器的语法与其它使用Lipcap（Linux）或者Winpcap（Windows）库开发的软件一样，比如著名的TCPdump。捕捉过滤器必须在开始捕捉前设置完毕

设置捕捉过滤器的步骤是：选择 capture->options。填写"capture filter"栏或者点击"capture filter"按钮为您的过滤器起一个名字并保存，以便在今后的捕捉中继续使用这个过滤器。点击开始（Start）进行捕捉

![image](https://github.com/dwjlw1314/DWJ-PROJECT/raw/master/PictureSource/2.2.5.png)

![image](https://github.com/dwjlw1314/DWJ-PROJECT/raw/master/PictureSource/2.2.6.png)

![image](https://github.com/dwjlw1314/DWJ-PROJECT/raw/master/PictureSource/2.2.7.png)

Protocol可能的值: ether, fddi, ip, arp, rarp, decnet, lat, sca, moprc, mopdl, tcp and udp.默认使用所有支持的协议

Direction(方向):

可能的值: src, dst, src and dst, src or dst. 如果没有特别指明来源或目的地，则默认使用 "src or dst" 作为关键字
> "host 10.2.2.2"与"src or dst host 10.2.2.2"是一样的

Host(s): 可能的值： net, port, host, portrange. 如果没有指定此值，则默认使用"host"关键字
> "src 10.1.1.1"与"src host 10.1.1.1"相同

Logical Operations(逻辑运算)，可能的值：not, and, or. 否(not)具有最高的优先级。或(or)和与(and)具有相同的优先级，运算是从左至右
> not tcp port 3128 and tcp port 23与(not tcp port 3128) and tcp port 23相同  <br>
> not tcp port 3128 and tcp port 23与not (tcp port 3128 and tcp port 23)不同

通用简单例子：

显示目的TCP端口为3128的封包
> tcp dst port 3128

显示来源IP地址为10.1.1.1的封包
> ip src host 10.1.1.1

显示目的或来源IP地址为10.1.2.3的封包
> host 10.1.2.3

显示来源为UDP或TCP，并且端口号在2000至2500范围内的封包
> src portrange 2000-2500

显示除了icmp以外的所有封包。(icmp通常被ping工具使用)
> not imcp

显示来源IP地址为10.7.2.12，但目的地不是10.200.0.0/16的封包
> src host 10.7.2.12 and not dst net 10.200.0.0/16

显示来源IP为10.4.1.12或者来源网络为10.6.0.0/16，目的地TCP端口号在200至10000之间，并且目的位于网络10.0.0.0/8内的所有封包
> (src host 10.4.1.12 or src net 10.6.0.0/16) and tcp dst portrange 200-10000 and dst net 10.0.0.0/8

注意事项：

当使用关键字作为值时，需使用反斜杠“\\”。

"ether proto \\ip" (与关键字"ip"相同).

这样写将会以IP协议作为目标。

"ip proto \\icmp" (与关键字"icmp"相同).

这样写将会以ping工具常用的icmp作为目标

可以在"ip"或"ether"后面使用"multicast"及"broadcast"关键字。当您想排除广播请求时，"no broadcast"就会非常有用

<p align="center">2.显示过滤器</p>

![image](https://github.com/dwjlw1314/DWJ-PROJECT/raw/master/PictureSource/2.2.8.png)

Protocol（协议）:

您可以使用大量位于OSI模型第2至7层的协议。点击"Expression..."按钮后，您可以看到它们。比如：IP，TCP，DNS，SSH

![image](https://github.com/dwjlw1314/DWJ-PROJECT/raw/master/PictureSource/2.2.9.png)

您同样可以在如下所示位置找到所支持的协议：

![image](https://github.com/dwjlw1314/DWJ-PROJECT/raw/master/PictureSource/2.2.10.jpg)

![image](https://github.com/dwjlw1314/DWJ-PROJECT/raw/master/PictureSource/2.2.11.png)

Wireshark的网站提供了对各种 协议以及它们子类的说明

String1, String2 (可选项):

协议的子类。点击相关父类旁的"+"号，然后选择其子类

Comparison operators （比较运算符）,可以使用6种比较运算符：

英文写法: |C语言写法: |含义：
:-:|:-:|:-:
eq |== |等于
ne |!= |不等于
gt |> |大于
lt |< |小于
ge |>= |大于等于
le |<= |小于等于

Logical expressions（逻辑运算符）:

英文写法: |C语言写法: |含义：
:-:|:-:|:-:
and |&& |逻辑与
or |&#124;&#124; |逻辑或
xor |^^ |逻辑异或
not |! |逻辑非

我们熟知的逻辑异或是一种排除性的或。当其被用在过滤器的两个条件之间时，只有当且仅当其中的一个条件满足时，这样的结果才会被显示在屏幕上

让我们举个例子："tcp.dstport 80 xor tcp.srcport 1025"

只有当目的TCP端口为80或者来源于端口1025（但又不能同时满足这两点）时，这样的封包才会被显示

显示SNMP或DNS或ICMP封包例子
> snmp || dns || icmp

显示来源或目的IP地址为10.1.1.1的封包
> ip.addr == 10.1.1.1

显示来源不为10.1.2.3或者目的不为10.4.5.6的封包。也就是显示的封包将会为：来源IP除了10.1.2.3以外任意；目的IP除了10.4.5.6以外任意
> ip.src != 10.1.2.3 or ip.dst != 10.4.5.6

显示来源不为10.1.2.3并且目的IP不为10.4.5.6的封包。换句话说，显示的封包将会为：  <br>
来源IP：除了10.1.2.3以外任意；同时须满足，目的IP：除了10.4.5.6以外任意
> ip.src != 10.1.2.3 and ip.dst != 10.4.5.6

显示来源或目的TCP端口号为25的封包
> tcp.port == 25

显示目的TCP端口号为25的封包
> tcp.dstport == 25

显示包含TCP标志的封包
> tcp.flags

显示包含TCP SYN标志的封包
> tcp.flags.syn == 0x02

<font color=#DC143C>如果过滤器的语法是正确的，表达式的背景呈绿色。如果呈红色，说明表达式有误</font>

![image](https://github.com/dwjlw1314/DWJ-PROJECT/raw/master/PictureSource/2.2.12.png)表达式正确
![image](https://github.com/dwjlw1314/DWJ-PROJECT/raw/master/PictureSource/2.2.13.png)表达式错误
