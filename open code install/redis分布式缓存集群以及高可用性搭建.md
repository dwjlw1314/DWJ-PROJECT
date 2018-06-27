------ rhel-server-6.4-x86_64 系统 --------

前提： redis-2.8.13.tar.gz，keepalived-1.2.13.tar.gz

redis有三种模式：主从模式、sentinel(哨兵模式基于主从模式)、mastercluster模式

解压redis源码包
>[root@diwj /]# tar -zxvf redis-2.8.13.tar.gz

进入解压目录进行源码编译
>[root@diwj /]# cd redis-2.8.13 && make --- 可以参考该目录下的readme文件

创建编译后程序存放目录，为了将Redis相关的资源统一管理而建立相关目录

    [root@diwj  redis-2.8.13]# mkdir /usr/local/redis
    [root@diwj  redis-2.8.13]# mkdir -p /usr/local/redis/etc
    [root@diwj  redis-2.8.13]# mkdir -p /usr/local/redis/var
    [root@diwj  redis-2.8.13]# mkdir -p /usr/local/redis/logs

生成可执行文件，同时拷贝主配置文件

    [root@diwj  redis-2.8.13]# make PREFIX=/usr/local/redis/ install
    [root@diwj  redis-2.8.13]# cp redis.conf /usr/local/redis/etc/

可选，修改内核参数,并生效
>[root@diwj  redis-2.8.13]# vim /etc/sysctl.conf

```
最后一行添加如下：
vm.overcommit_memory=1
注释：新添加行指定了内核针对内存分配策略，其值可取0，1，2；
0 表示内核将检查是否有足够的可用内存供应用进程使，如果有可用内存，申请允许，否则失败，告知上层应用进程
1 表示内核允许分配所有物理内存，而不管当前的内存状态如何
2 表示内核允许分配超过所有物理内存和交换空间总和的内存
```
保存完成后运行 sysctl -p 使配置生效

编译后的可执行程序

    redis-server       #redis服务器的daemon启动程序
    redis-cli          #redis命令行操作工具
    redis-benchmark    #redis性能测试工具，测试当前系统配置下读写性能
    redis-stat         #redis状态检测工具，可检测redis当前状态参数及延迟情况

进入可执行程序目录
>[root@diwj  redis-2.8.13]# cd /usr/local/redis/bin

非daemon启动服务和检查是否成功,如果出现 PONG 表示成功

    [root@diwj bin]# ./redis-server
    [root@diwj bin]# ./redis-cli -p 6379 ping

使用内置的客户端连接Redis server测试

    [root@diwj bin]# ./redis-cli -p 6379
    127.0.0.1:6379> set gjsy diwj   ---  OK
    127.0.0.1:6379> get gjsy  ---   "diwj"

关闭指定端口的redis服务
>[root@diwj bin]# ./redis-cli -p 6379 shutdown

主配置文件常用参数详解
```
daemonize yes       #是否以后台daemon方式运行，非daemon启动时会每隔5秒输出一行监控信息
maxmemory 256000000 #分配256M内存
pidfile：           #pid文件位置
port：              #监听的端口号
timeout：           #连接超时时间
loglevel：          #log信息级别
logfile：           #log文件位置
databases：         #开启数据库的数量
save * *：          #保存快照频率，第一个*表示多长时间，第二个*表示执行多少次写操作。在一定时间内执行一定数量的写操作时，自动保存快照。可设置多个条件
rdbcompression：    #是否使用压缩
dbfilename：        #数据快照文件名（只是文件名，不包括目录）
dir：               #数据快照的保存目录（这个是目录）
appendonly：        #是否开启appendonlylog，开启的话每次写操作会记一条log，这会提高数据抗风险能力，但影响效率。
appendfsync：       #appendonlylog如何同步到磁盘（三个选项，分别是每次写都强制调用fsync、每秒启用一次fsync、不调用fsync等待系统自己同步）
requirepass：       #节点密码认证设置
masterauth：        #slave节点加入master节点的密码认证
```
指定自定义的主配置文件运行daemon模式
>[root@diwj bin]# ./redis-server ../etc/redis.conf

Redis常用内存优化手段与参数---参考优化
* 不要开启Redis的VM选项，即虚拟内存功能，这个本来是作为Redis存储超出物理内存数据的一种数据在内存与，磁盘换入换出的一个持久化策略，
但是其内存管理成本也非常的高，并且我们后续会分析此种持久化策略并不成熟，所以要关闭VM功能，请检查你的redis.conf文件中 vm-enabled 为 no
* 设置redis.conf中的maxmemory选项，该选项是告诉Redis当使用了多少物理内存后就开始拒绝后续的写入请求,
该参数能很好的保护好你的Redis不会因为使用了过多的物理内存而导致swap,最终严重影响性能甚至崩溃
* Redis为不同数据类型分别提供了一组参数来控制内存使用，我们在前面详细分析过Redis Hash是value内部为一个HashMap,
如果该Map的成员数比较少，则会采用类似一维线性的紧凑格式来存储该Map, 即省去了大量指针的内存开销
```
这个参数控制对应在redis.conf配置文件中下面2项：
hash-max-zipmap-entries 64  
hash-max-zipmap-value 512
```
hash-max-zipmap-entries含义是当value这个Map内部不超过多少个成员时会采用线性紧凑格式存储，默认是64,即value内部有64个以下的成员
就是使用线性紧凑存储，超过该值自动转成真正的HashMap，hash-max-zipmap-value 含义是当 value这个Map内部的
每个成员值长度不超过多少字节就会采用线性紧凑存储来节省空间，以上2个条件任意一个条件超过设置值都会转换成真正的HashMap，
也就不会再节省内存了，那么这个值是不是设置的越大越好呢，答案当然是否定的，HashMap的优势就是查找和操作的时间复杂度都是O(1)的，
而放弃Hash采用一维存储则是O(n)的时间复杂度，如果成员数量很少，则影响不大，否则会严重影响性能，所以要权衡好这个值的设置，
总体上还是最根本的时间成本和空间成本上的权衡,同样类似的参数有：
```
list-max-ziplist-entries 512 说明：list数据类型多少节点以下会采用去指针的紧凑存储格式
list-max-ziplist-value 64    说明：list数据类型节点值大小小于多少字节会采用紧凑存储格式
set-max-intset-entries 512   说明：set数据类型内部数据如果全部是数值型，且包含多少节点以下会采用紧凑格式存储
```
最后想说的是Redis内部实现没有对内存分配方面做过多的优化，在一定程度上会存在内存碎片，不过大多数情况下这个不会成为Redis的性能瓶颈，不过如果在Redis内部存储的大部分数据是数值型的话，Redis内部采用了一个shared integer的方式来省去分配内存的开销，即在系统启动时先分配一个从1~n 那么多个数值对象放在一个池子中，如果存储的数据恰好是这个数值范围内的数据，则直接从池子里取出该对象，并且通过引用计数的方式来共享，这样在系统存储了大量数值下，也能一定程度上节省内存并且提高性能，这个参数值n的设置需要修改源代码中的一行宏定义REDIS_SHARED_INTEGERS，该值默认是10000，可以根据自己的需要进行修改，修改后重新编译就可以了

* Rredis的6种过期策略redis 中的默认的过期策略是volatile-lru 。设置方式：
```
config set maxmemory-policy volatile-lru
maxmemory-policy 六种方式
volatile-lru：      只对设置了过期时间的key进行LRU（默认值）
allkeys-lru：       是从所有key里 删除 不经常使用的key
volatile-random：   随机删除即将过期key
allkeys-random：    随机删除
volatile-ttl ：     删除即将过期的
noeviction ：       永不过期，返回错误
maxmemory-samples 3 是说每次进行淘汰的时候 会随机抽取3个key 从里面淘汰最不经常使用的（默认选项）
```
<p align="center">双机热备配置过程</p>

基本信息 : Master  172.16.10.210   Slave   172.16.10.211

Master的配置,首先创建启动脚本,添加如下内容
>[root@diwj /]# vim /usr/local/redis/redis-start.sh

```bash
#!/bin/bash
REDISPATH=/usr/local/redis
REDISCLI=$REDISPATH/bin/redis-cli
LOGFILE=$REDISPATH/logs/redis-state.log
LOCALIP=172.16.10.210
REMOTEIP=172.16.10.211

REMOTEREDISROLE=`$REDISCLI -h $REMOTEIP info | grep "role"`
if grep "role:master" <<< $REMOTEREDISROLE ; then
#start as slave
$REDISPATH/bin/redis-server $REDISPATH/etc/redis_slave.conf;
          if [ "$?" == "0" ];then
	echo "[INFO]`date +%F/%H:%M:%S` :$LOCALIP start as slave successful." >> $LOGFILE
          else
              echo "[ERROR]`date +%F/%H:%M:%S` :$LOCALIP start as slave error." >> $LOGFILE
    fi
else
#start as master
$REDISPATH/bin/redis-server $REDISPATH/etc/redis_master.conf;
          if [ "$?" == "0" ];then
	echo "[INFO]`date +%F/%H:%M:%S` :$LOCALIP start as master successful." >> $LOGFILE
          else
              echo "[ERROR]`date +%F/%H:%M:%S` :$LOCALIP start as master error." >> $LOGFILE
          fi
fi
```
创建redis关闭脚本，添加如下内容  <br>
[root@diwj /]# vim /usr/local/redis/redis-stop.sh
```bash
#!/bin/bash
REDISPATH=/usr/local/redis
LOGFILE=$REDISPATH/logs/redis-state.log
LOCALIP=172.16.10.210
REMOTEIP=172.16.10.211
kill -9 `ps -ef|grep 'redis-server'|grep -v grep|awk  '{print $2}'`
if [ "$?" == "0" ];then
	echo "[INFO]`date +%F/%H:%M:%S` :redis shutdown completed!" >> $LOGFILE
else
              echo "[ERROR]`date +%F/%H:%M:%S` :redis is not started." >> $LOGFILE
fi
```
更改脚本权限

    [root@diwj /]# chmod 755 /usr/local/redis/redis-start.sh
    [root@diwj /]# chmod 755 /usr/local/redis/redis-stop.sh

创建主从配置文件

    [root@diwj /]# cp /usr/local/redis/etc/redis.conf /usr/local/redis/etc/redis_master.conf
    [root@diwj /]# cp /usr/local/redis/etc/redis.conf /usr/local/redis/etc/redis_slave.conf

修改主配置文件 172.16.10.210/211服务器redis_master.conf对应配置项,以下为改动部分，其他的依据实际调整

    daemonize yes
    pidfile /usr/local/redis-2.8.13/var/redis.pid
    timeout 300
    loglevel debug        //后期发布版本修改
    dir /usr/local/redis-2.8.13/var/
    appendfsync always
    logfile "/usr/local/redis-2.8.13/logs/redis.log"

修改从配置文件 172.16.10.210服务器redis_slave.conf对应配置项,以下为改动部分，其他配置依据实际修改

    daemonize yes
    logfile "/usr/local/redis-2.8.3/logs/redis.log"
    slaveof 172.16.10.211 6379

修改从配置文件 172.16.10.211服务器redis_slave.conf对应配置项,以下为改动部分，其他配置依据实际修改

    daemonize yes
    logfile "/usr/local/redis-2.8.3/logs/redis.log"
    slaveof 172.16.10.210 6379

上述主备conf文件的bind 选项注释掉，让redis监听所有本地网卡数据包

<p align="center">高可用性配置</p>
vip == 172.16.10.213

编译安装keepalived，前提安装相关依赖包 openssl,popt,libnl-dev等，可缺失，暂时不影响

创建编译后程序存放目录，为了keepalived相关的资源统一管理而建立相关目录
>[root@diwj /]# mkdir /usr/local/keepalived

执行keepalived的编译命令

    [root@diwj keepalived]# ./configure --prefix=/usr/local/keepalived
    [root@diwj keepalived]# make && make install

建立服务启动脚本，方便使用service控制启停

    [root@diwj keepalived]# cp /usr/local/keepalived/etc/rc.d/init.d/keepalived /etc/init.d/keepalived
    [root@diwj keepalived]# chmod +x /etc/init.d/keepalived

如果我们使用非默认路径安装，则需要修改几处路径，保证keepalived正常启动
>[root@diwj keepalived]# vim /etc/init.d/keepalived

修改内容在15行左右 --- 把/etc/sysconfig/keepalived改成/usr/local/keepalived/etc/sysconfig/keepalived
同时在此行下添加以下内容 （将keepavlied主程序所在路径导入到环境变量PATH中）

    PATH="$PATH:/usr/local/keepalived/sbin"
    export PATH

修改/usr/local/keepalived/etc/sysconfig/keepalived文件，设置正确的服务启动参数  <br>
KEEPALIVED_OPTIONS="-D -f /usr/local/keepalived/etc/keepalived/keepalived.conf"

keepalived基本安装完成，启动测试
>[root@diwj keepalived]# service keepalived start

使用ip a命令验证是否正常，如下图有虚拟ip显示表示成功
![image](https://github.com/dwjlw1314/DWJ-PROJECT/raw/master/PictureSource/3.12.1.jpg)

启动正确后将此服务设置为开机启动
>[root@diwj keepa ed]# chkconfig keepalived on

备份keepalived.conf配置文件,备用

创建脚本运行目录
>[root@diwj keepalived]# mkdir /usr/local/keepalived/etc/keepalived/scripts

修改主keepalived.conf配置文件
```bash
vrrp_script chk_redis {
	script "/usr/local/keepalived/etc/keepalived/scripts/chk_redis.sh"   ###监控脚本
	interval 2   				                        ###监控时间
	weight 2
}
vrrp_instance VI_1 {
	state MASTER                                ###设置为MASTER，备用的设置BACKUP
	interface eth0                              ###监控网卡,依据实际情况来定  
	virtual_router_id 51                        ###VRRP组名，两个节点必须一致，指明各个节点同属一个VRRP组
	priority 101                                ##权重值（1-254），高优先级大的竞选为MASTER，数值大为高
	advert_int 1                                ###组播信息发送间隔，两个节点必须一样，默认1s
	authentication {
		auth_type PASS                            ###认证方式
		auth_pass redis                           ###认证密码
	}
	track_script {
		chk_redis                       	        ###执行上面定义的chk_redis
	}
	virtual_ipaddress {
		172.16.10.213/24                	        ###VIP
	}
}
```

备的keepalived.conf文件同上，只需要修改state和priority字段

keepalived的配置文件详细说明：
```
global_defs {
   notification_email {         #指定keepalived在发生切换时需要发送email到的对象，一行一个
   }
   notification_email_from     #指定发件人
   smtp_server localhost       #指定smtp服务器地址
   smtp_connect_timeout 30     #指定smtp连接超时时间
   router_id LVS_DEVEL         #运行keepalived机器的一个标识
}
vrrp_sync_group VG_1{          #监控多个网段的实例
    group {
      inside_network                #实例名
      outside_network
    }
    notify_master /path/xx.sh  #指定当切换到master时，执行的脚本
    netify_backup /path/xx.sh  #指定当切换到backup时，执行的脚本
    notify_fault "path/xx.sh VG_1"   #故障时执行的脚本
    notify /path/xx.sh
    smtp_alert                 #使用global_defs中提供的邮件地址和smtp服务器发送邮件通知
}
vrrp_instance inside_network {
  state BACKUP      #指定那个为master，那个为backup，如果设置了nopreempt这个值不起作用，主备靠priority决定
  interface eth0    #设置实例绑定的网卡
  dont_track_primary #忽略vrrp的interface错误（默认不设置）
  track_interface{  #设置额外的监控，里面那个网卡出现问题都会切换
    eth0
    eth1
  }
  mcast_src_ip      #发送多播包的地址，如果不设置默认使用绑定网卡的primary ip
  garp_master_delay #在切换到master状态后，延迟进行gratuitous ARP请求
  virtual_router_id 50 #VPID标记
  priority 99       #优先级，高优先级竞选为master
  advert_int 1      #检查间隔，默认1秒
  nopreempt         #设置为不抢占 注：这个配置只能设置在backup主机上，而且这个主机优先级要比另外一台高
  preempt_delay     #抢占延时，默认5分钟
  debug #debug级别
  authentication {  #设置认证
    auth_type PASS  #认证方式
    auth_pass 111111 #认证密码
  }
  virtual_ipaddress { #设置vip
    192.168.202.200
  }
}
virtual_server 192.168.202.200 23 {
  delay_loop 6          #健康检查时间间隔
  lb_algo rr            #lvs调度算法rr|wrr|lc|wlc|lblc|sh|dh
  lb_kind DR            #负载均衡转发规则NAT|DR|RUN
  persistence_timeout 5 #会话保持时间
  protocol TCP          #使用的协议
  persistence_granularity <NETMASK>   #lvs会话保持粒度
  virtualhost <string>  #检查的web服务器的虚拟主机（host：头）
  sorry_server<IPADDR> <port>      #备用机，所有realserver失效后启用
  real_server 192.168.200.5 23 {
     weight 1                 #默认为1,0为失效
     inhibit_on_failure #在服务器健康检查失效时，将其设为0，而不是直接从ipvs中删除
     notify_up <string> | <quoted-string> #在检测到server up后执行脚本
     notify_down <string> | <quoted-string> #在检测到server down后执行脚本
     TCP_CHECK {
         connect_timeout 3    #连接超时时间
         nb_get_retry 3       #重连次数
         delay_before_retry 3 #重连间隔时间
         connect_port 23      #健康检查的端口的端口
         bindto <ip>
      }
      HTTP_GET | SSL_GET{
         url{                 #检查url，可以指定多个
            path /
            digest <string>   #检查后的摘要信息
            status_code 200   #检查的返回状态码
         }
         connect_port <port>
         bindto <IPADD>
         connect_timeout 5
         nb_get_retry 3
         delay_before_retry 2
      }
     SMTP_CHECK{
       host{
         connect_ip <IP ADDRESS>
         connect_port <port>   #默认检查25端口
         bindto <IP ADDRESS>
      }
       connect_timeout 5
       retry 3
       delay_before_retry 2
       helo_name <string> | <quoted-string>      #smtp helo请求命令参数，可选
     }
      MISC_CHECK{
         misc_path <string> | <quoted-string>    #外部脚本路径
         misc_timeout       #脚本执行超时时间
         misc_dynamic       #如设置该项，则退出状态码会用来动态调整服务器的权重，返回0 正常，不修改；返回1，
        检查失败，权重改为0；返回2-255，正常，权重设置为：返回状态码-2
     }
  }
}
  real_server 192.168.0.20 80 {     //真实IP web的IP
    weight 1                        //默认为1,0为失效
    HTTP_GET {
      connect_port 80              //健康检查端口
      connect_timeout 3            //链接超时时间
      nb_get_retry 3               //重链次数
      delay_before_retry 3         //重连讲时间(秒)
    }
  }
}
```
---

创建监控redis服务脚本,添加以下内容
>[root@diwj keepalived]# vim /usr/local/keepalived/etc/keepalived/scripts/chk_redis.sh

```bash
#!/bin/bash
REDISPATH=/usr/local/redis
REDISCLI=$REDISPATH/bin/redis-cli
LOGFILE=$REDISPATH/logs/redis-state.log
LOCALIP=172.16.10.210
REMOTEIP=172.16.10.211
VIP=172.16.10.213

VIPALIVE=`ip a | grep "$VIP"`
if [ "$VIPALIVE" == "" ]; then
	   echo "[info]:"`date`" keepalived server is pengding or stop" >> $LOGFILE
	   if [ "`ps -e|grep redis-server | wc -l`" == 0 ]; then
		   echo "[warn]:"`date`"  redis server($LOCALIP) is starting..." >> $LOGFILE
		   /usr/local/redis/redis-start.sh
	   fi
else
		echo "keepalived service alived" >> $LOGFILE
		#check local service is running
		if [ "`$REDISCLI -h $LOCALIP -p 6379 PING`" == "PONG" ]; then
				# check local redis server role.
				REDISROLE=`$REDISCLI info | grep "role"`
				if grep "role:slave" <<< $REDISROLE ; then
						#change local redis server as master
						echo "[info1]:"`date`" Run SLAVEOF NO ONE cmd ..." >> $LOGFILE
						$REDISCLI SLAVEOF NO ONE >> $LOGFILE 2>&1

						#change remoting redis server as slave
						REMOTEREDISROLE=`$REDISCLI -h $REMOTEIP info | grep "role"`
						if grep "role:master" <<< $REMOTEREDISROLE ; then
								echo "[info2]:"`date`" Run remote server SLAVEOF cmd ..." >> $LOGFILE
								$REDISCLI -h $REMOTEIP SLAVEOF $LOCALIP 6379 >> $LOGFILE  2>&1
						fi
				else
						REMOTEREDISROLE=`$REDISCLI -h $REMOTEIP info | grep "role"`
						if grep "role:master" <<< $REMOTEREDISROLE ; then
								echo "[info3]:"`date`" Run remote server SLAVEOF cmd ..." >> $LOGFILE
								$REDISCLI -h $REMOTEIP SLAVEOF $LOCALIP 6379 >> $LOGFILE  2>&1
						fi
				fi
		else
				echo "[warn]:"`date`"  redis server($LOCALIP) is not health..." >> $LOGFILE
		echo "[warn]:"`date`"  redis server($LOCALIP) is closing..." >> $LOGFILE
		/usr/local/redis/redis-stop.sh
		echo "[warn]:"`date`"  redis server($LOCALIP) is starting..." >> $LOGFILE
		/usr/local/redis/redis-start.sh
				sleep 1
				if [ "`$REDISCLI -h $LOCALIP -p 6379 PING`" != "PONG" ]; then
						echo "[error]:"`date`"  redis server($LOCALIP) will be stop..." >> $LOGFILE
						service keepalived stop
				fi
		fi
fi
```
更改脚本权限
>[root@diwj keepalived]# chmod 755 /usr/local/keepalived/etc/keepalived/scripts/chk_redis.sh

脚本创建完成以后，我们开始按照如下流程进行测试：
```
1.启动Master上的Redis   
/usr/local/redis/redis-start.sh
#关闭时，直接杀死进程或执行以下脚本
/usr/local/redis/redis-stop.sh
2.启动Slave上的Redis   
/usr/local/redis/redis-start.sh
#关闭时，直接杀死进程或执行以下脚本
#/usr/local/redis/redis-stop.sh
3.启动Master上的Keepalived   
/etc/rc.d/init.d/keepalived start            //service keepalived start
#关闭方法
#/etc/rc.d/init.d/keepalived stop            //service keepalived stop
4.启动Slave上的Keepalived   
/etc/rc.d/init.d/keepalived start
#关闭方法
#/etc/rc.d/init.d/keepalived stop
```
<p align="center">keepalived相关问题</p>
测试及验证：拔掉节点A的网线，就发现虚拟IP已经绑定到节点B上，再恢复A节点的网线，虚拟IP又绑定回节点A之上，但是这种方式存在恼裂的可能，即两个节点实际都处于正常工作状态，但是无法接收到彼此的组播通知，这时两个节点均强行绑定虚拟IP，导致不可预料的后果，这时就需要设置仲裁，即每个节点必须判断自身的状态（应用服务状态及自身网络状态），要实现这两点可使用自定义shell脚本实现，通过周期性地检查自身应用服务状态，并不断ping网关（或其它可靠的参考IP）均可。当自身服务异常、或无法ping通网关，则认为自身出现故障，就应该移除掉虚拟IP(停止keepalived服务即可）。主要借助keepalived提供的vrrp_script及track_script实现

在keepalived的配置文件最前面加入以下代码，定义一个跟踪脚本：
```
vrrp_script check_local {                              #定义一个名称为check_local的检查脚本
    script "/usr/local/keepalived/bin/check_local.sh"  #shell脚本的路径
    interval 5                            			       #运行间隔
    weight 2
}
```
再在vrrp_instance配置中加入以下代码使用上面定义的检测脚本：
```
track_script {
    check_local
}
```
我们在check_local.sh定义的检测规则是

1.自身web服务故障（超时，http返回状态不是200）

2.无法ping通网关

3.产生以上任何一个问题，均应该移除本机的虚拟IP(停止keepalived实例即可)

但这里有个小问题，如果本机或是网关偶尔出现一次故障，那么我们不能认为是服务故障。更好的做法是如果连续N次检测本机服务不正常或连接N次无法ping通网关，才认为是故障产生，才需要进行故障转移。另一方面，如果脚本检测到故障产生，并停止掉了keepalived服务，那么当故障恢复后，keepalived是无法自动恢复的。我觉得利用独立的脚本以秒级的间隔检查自身服务及网关连接性，再根据故障情况控制keepalived的运行或是停止

<p align="center">mastercluster模式</p>    <br>
redis版本：redis-4.0.9.tar.gz

1.单机模拟mastercluster模式，创建3组master-slave的服务；创建6个文件夹模拟6个redis服务
```
root@dwj local]# mkdir -p redis-cluster
[root@dwj local]# cd redis-cluster/
[root@dwj redis-cluster]# mkdir {7001..7006}
```
2.拷贝redis.conf文件到7001个文件夹下，修改配置文件
```
[root@dwj redis-cluster]# cp /opt/redis-4.0.9/redis.conf 7001-7006
[root@dwj 7001]# vim redis.conf   #修改内容如下：
bind 192.168.8.45        #本地ip
port 7001                #监听端口
daemonize yes            #后台运行
appendonly yes           #aof模式
appendfsync always
pidfile /var/run/redis_6379.pid
logfile "/usr/local/redis-cluster/7001/redis.log"
dir /usr/local/redis-cluster/7001/
cluster-enabled yes      #开启mastercluster模式
cluster-config-file nodes-7001.conf
cluster-node-timeout 15000
```
2.拷贝redis.conf文件到其他5个文件夹下，修改配置文件
```
[root@dwj 7001]# vim redis.conf   #替换内容如下：
%s/7001/7002/g     #替换各自的文件夹下配置文件内容
```
3.redis集群需要ruby命令，安装ruby
```
[root@dwj redis-cluster]# yum -y install ruby
[root@dwj redis-cluster]# yum -y install rubygems
[root@dwj redis-cluster]# gem install redis
```
4.启动6个redis实例，检查是否成功
```
[root@dwj redis-cluster]# redis-server 7001/redis.conf
[root@dwj redis-cluster]# ps -ef |grep redis
root      64884      1  0 11:38 ?        00:00:00 redis-server 192.168.8.45:7001 [cluster]
root      64925      1  0 11:41 ?        00:00:00 redis-server 192.168.8.45:7002 [cluster]
root      64930      1  0 11:41 ?        00:00:00 redis-server 192.168.8.45:7003 [cluster]
root      64935      1  0 11:41 ?        00:00:00 redis-server 192.168.8.45:7004 [cluster]
root      64940      1  0 11:41 ?        00:00:00 redis-server 192.168.8.45:7005 [cluster]
root      64945      1  0 11:41 ?        00:00:00 redis-server 192.168.8.45:7006 [cluster]
root      64954  39206  0 11:42 pts/1    00:00:00 grep redis
```
5.进入redis-4.0.9安装目录src下，然后执行redis-trib.rb
```
[root@dwj src]# ./redis-trib.rb create --replicas 1 192.168.8.45:7001 192.168.8.45:7002 192.168.8.45:7003 192.168.8.45:7004 192.168.8.45:7005 192.168.8.45:7006        # 其中 1 是master节点与slave节点比值
  >>> Creating cluster
  >>> Performing hash slots allocation on 6 nodes...
  Using 3 masters:
  192.168.8.45:7001
  192.168.8.45:7002
  192.168.8.45:7003
  Adding replica 192.168.8.45:7005 to 192.168.8.45:7001
  Adding replica 192.168.8.45:7006 to 192.168.8.45:7002
  Adding replica 192.168.8.45:7004 to 192.168.8.45:7003
  >>> Trying to optimize slaves allocation for anti-affinity
  [WARNING] Some slaves are in the same host as their master
  M: 3cc311e2ab42acb088366bcc31909fd6ac1c96f3 192.168.8.45:7001
     slots:0-5460 (5461 slots) master
  M: ec23523f374f5b43c3b6d5ecb59bf331690f2cac 192.168.8.45:7002
     slots:5461-10922 (5462 slots) master
  M: ffb7d482bb228af14dc709c774f58fb22e63c7d8 192.168.8.45:7003
     slots:10923-16383 (5461 slots) master
  S: a04780d0dc77c102b5cd5594ce726383a3b00217 192.168.8.45:7004
     replicates ec23523f374f5b43c3b6d5ecb59bf331690f2cac
  S: c6cc52bcd62dc458c102cfe53c45a8340d4c03b1 192.168.8.45:7005
     replicates ffb7d482bb228af14dc709c774f58fb22e63c7d8
  S: c04144694f1ab301b5851433e82651905f9e835f 192.168.8.45:7006
     replicates 3cc311e2ab42acb088366bcc31909fd6ac1c96f3
  Can I set the above configuration? (type 'yes' to accept): yes
  >>> Nodes configuration updated
  >>> Assign a different config epoch to each node
  >>> Sending CLUSTER MEET messages to join the cluster
  Waiting for the cluster to join.......
  >>> Performing Cluster Check (using node 192.168.8.45:7001)
  [OK] All nodes agree about slots configuration.
  >>> Check for open slots...
  >>> Check slots coverage...
  [OK] All 16384 slots covered.
```
6.进行集群验证
```
[root@dwj src]# redis-cli -c -h 192.168.8.45 -p 7001  #-c是集群的意思
192.168.8.45:7001> cluster info
192.168.8.45:7001> cluster nodes
192.168.8.45:7001> set gjsy 100
  -> Redirected to slot [9953] located at 192.168.8.45:7002
  OK
```
7.关闭集群需要逐个关闭
>[root@dwj src]# redis-cli -c -h 192.168.8.45 -p 7001 shutdown

8.增加redis集群节点
```
[root@dwj redis-cluster]# mkdir 7007 7008
[root@dwj redis-cluster]# cp 7001/redis.conf 7007 7008
[root@dwj redis-cluster]# vim 7007/redis.conf   #替换内容如下：
%s/7001/7002/g     #替换各自的文件夹下配置文件内容
[root@dwj redis-cluster]# redis-server 7007/redis.conf
[root@dwj redis-cluster]# redis-server 7008/redis.conf
-->添加7007节点
[root@dwj src]# ./redis-trib.rb add-node 192.168.8.45:7007 192.168.8.45:7001
  >>> Check for open slots...
  >>> Check slots coverage...
  [OK] All 16384 slots covered.
  >>> Send CLUSTER MEET to node 192.168.8.45:7007 to make it join the cluster.
  [OK] New node added correctly.
添加新节点成功后，不会有任何数据，因为它没有分配slot，需要手动分配
[root@dwj src]# ./redis-trib.rb reshard 192.168.8.45:7001    #对任意主节点重新分配slot
  三次提示中 第1个是需要多少个slot移动到新的节点；第2个是需要把指定的slot移动到哪个节点上；第3个是输入all从所有主节点分别抽取相应的slot到新节点，第4个输入yes开始执行分片任务
-->添加7008节点
[root@dwj src]# ./redis-trib.rb add-node 192.168.8.45:7008 192.168.8.45:7001
7008节点是slave,所以不需要分配slot,只需要指定当前节点的主节点是那一个
[root@dwj src]# redis-cli -c -h 192.168.8.45 -p 7008
192.168.8.45:7008> cluster replicate c1896a005a38aa94c47658b93558bfa6500f7e8a
  ok
```
9.删除redis集群节点
```
-->删除slave节点7008
[root@dwj src]# ./redis-trib.rb del-node 192.168.8.45:7008 ca4bd3acd61c2fdc3b080d59da97b66baeda2e6a
  >>> Removing node ca4bd3acd61c2fdc3b080d59da97b66baeda2e6a from cluster 192.168.8.45:7008
  >>> Sending CLUSTER FORGET messages to the cluster...
  >>> SHUTDOWN the node.
-->删除master节点7007
删除7007节点之前，需要先把slot数据迁移到其他的主节点上(目前只能指定到一个主节点，没有实现平均分配功能)
[root@dwj src]# ./redis-trib.rb reshard 192.168.8.45:7007
  三次提示中 第1个是需要迁移多少个slot移动到指定的节点；第2个是需要把指定的slot移动到哪个节点上；第3个是移动的slot是在哪一个节点；第4个是输入done迁移slot到指定节点，第4个输入yes开始执行迁移任务
[root@dwj src]# ./redis-trib.rb del-node 192.168.8.45:7007 c1896a005a38aa94c47658b93558bfa6500f7e8a
  >>> Removing node c1896a005a38aa94c47658b93558bfa6500f7e8a from cluster 192.168.8.45:7007
  >>> Sending CLUSTER FORGET messages to the cluster...
  >>> SHUTDOWN the node.
```
