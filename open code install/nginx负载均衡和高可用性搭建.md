详细参考腾讯微云-xx文件-资料文件（nginx+redis+nlb+文件服务器+ftp服务器）

#LVS（linux virtual system） 是在传输层，nginx是在应用层
#LVS有NAT、IP Tunneling、DR(Direct Routing)这3种模式

<font color=#FF0000 size=5>1、nginx平滑升级</font>

    [root@dwj /usr/local/nginx]#ps -ef|grep nginx        #查看旧版本nginx服务
    [root@dwj /usr/local/nginx]#sbin/nginx -V            #查看旧版本编译选项，可以沿用configure arguments

编译新版本nginx，不使用<make install>命令

    [root@dwj /usr/local/nginx]#mv sbin/nginx sbin/nginx-old               #把旧nginx改名
    [root@dwj /opt/nginx-1.12.1]#cp objs/nginx /usr/local/nginx/sbin/      #将新nginx拷贝到旧版本sbin目录下
    [root@dwj /usr/local/nginx]#sbin/nginx -V                              #查看新nginx版本信息
    [root@dwj /usr/local/nginx]#kill -USR2 `cat logs/nginx.pid`            #启动新的nginx程序

生成新的nginx.pid文件，旧的被改成nginx.pid.oldbin

    [root@dwj /usr/local/nginx]#kill -WINCH `cat logs/nginx.pid.oldbin`   #关闭旧的nginx服务worker process
    [root@dwj /usr/local/nginx]#kill -9 `cat logs/nginx.pid.oldbin`       #关闭旧的nginx服务master process

访问html页面查看版本信息，升级成功

<font color=#FF0000 size=5>2、nginx 配置文件说明</font>

nginx.conf文件常用字段优化含义(后期扩展):
```
worker_processes 2                        #nginx运行的进程数
work_cpu_affinity 00000001 00000002       #为每个进程分配cpu
worker_rlimit_nofile 10240                #设置nginx进程最多打开的文件描述符,最好与ulimit -n保持一致
events {
	use epoll;                    #使用epoll多路复用IO
	worker_connections 1024       #单个后台进程的最大并发链接数
	multi_accept on;              #尽可能接收多的请求
}

http {
	sendfile on;                  #指定nginx是否调用sendfile函数来输出文件,普通应用必须设置on,如果用来进行下载等磁盘IO重负载应用,可设置off
	autoindex on;                 #开启目录列表访问,适合下载服务器,默认关
	tcp_nopush on;                #防止网络阻塞
	keepalive_timeout 60;         #超时时间
	tcp_nodelay on;               #提高数据的实时响应性
	gzip_buffers 512k;            #压缩缓存空间大小
	gzip_min_length 1k;           #最小压缩大小
	gzip_comp_level 5;            #压缩等级,最大是9,值越小压缩比例越小,cpu处理更快
	client_max_body_size 10m;     #允许客户端请求最大的单文件字节数10M
	client_body_buffer_size 128k; #缓存区代理缓冲用户端请求的最大字节数
	proxy_connect_timeout 120;    #nginx与后端服务器的链接超时时间
	proxy_send_timeout 120;       #后端服务器数据回传时间
	proxy_read_timeout 120;       #连接成功后，后端服务器处理请求时间

  include vhost.conf {          #后端服务单独出来做成配置文件
  }
}
  vhost.conf配置文件如下：
  server {
      location / {
          expires  3d           #定义用户浏览器缓存的时间3天，对于不长更新的静态页面，可以设置更长时间，可以节省带宽和缓存服务器压力
      }
  }
```

官方最新 ngx_http_upstream_module 指令详述。ngx_http_upstream_module 模块用于定义可以被 proxy_pass、fastcgi_pass
以及memcached_pass 等指令引用的服务器群。配置示例: <br>
```
[plain] view plaincopyprint?
upstream backend {  
    server backend1.example.com       weight=5;  
    server backend2.example.com:8080;  
    server unix:/tmp/backend3;  

    server backup1.example.com:8080   backup;  
    server backup2.example.com:8080   backup;  
}  

server {  
    location / {  
        proxy_pass http://backend;  
    }  
}  
```
动态可配置群，仅作为我们商业订阅的一部分：
```
[plain] view plaincopyprint?
upstream appservers {  
    zone appservers 64k;  

    server appserv1.example.com      weight=5;  
    server appserv2.example.com:8080 fail_timeout=5s slow_start=30s;  
    server 192.0.2.1                 max_fails=3;  

    server reserve1.example.com:8080 backup;  
    server reserve2.example.com:8080 backup;  
}  

server {  
    location / {  
        proxy_pass http://appservers;  
        health_check;  
    }  

    location /upstream_conf {  
        upstream_conf;  
        allow 127.0.0.1;  
        deny all;  
    }  
}  
```
指令语法：upstream name { ... }    默认值：— <br>
上下文：http
定义一群服务器。服务器可以监听到不同的端口。此外，监听 TCP 和 UNIX-domian socket 的服务器可以混合定义 <br>
例如：<br>
```
[plain] view plaincopyprint?
upstream backend {  
    server backend1.example.com weight=5;  
    server 127.0.0.1:8080       max_fails=3 fail_timeout=30s;  
    server unix:/tmp/backend3;  
}  
```
默认情况下，请求是使用一个加权重的循环负载方法分配给各个主机的。在上面的例子中，每七个请求会以以下方案进行分配：五个请求分给 backend1.example.com 而另外两个请求分别分配给第二和第三个主机。如果在连接一台主机时发生错误，当前请求会被传给下一台主机，如此这般知道所有运行中的服务器都被尝试。如果没有任何主机成功返回，客户端会受到从最后一台主机返回的通信结果

语法：server address [parameters]; 默认值：— <br>
上下文：upstream
定义一台主机的地址以及其他一些参数。地址可以被指定为一个域名或一个 IP 地址，端口号参数可选，或者被指定为 "unix:" 前缀之后的一个 UNIX-domain socket 路径。端口号没指定的话就使用端口号 80。解析到多个 IP 地址的域名会一次性定义多台主机 <br>

可以定义以下参数：
```
weight=number        设置服务器的权重，默认为 1
max_fails=number     设置由fail_timeout定义的时间段内连接该主机的失败次数，以此来断定fail_timeout定义的时间段内该主机是否可用。默认情况下这个数值设置为1。零值的话禁用这个数量的尝试。由proxy_next_upstream、fastcgi_next_upstream、以及memcached_next_upstream等指令来判定错误尝试
fail_timeout=time    设置在指定时间内连接到主机的失败次数，超过该次数该主机被认为不可用。服务器被视为无效的时段。这个参数默认是10秒钟
slow_start=time      设置一台不健康的主机变成健康主机，或者当一台主机在被认为不可用变成可用时，将其权重由零恢复到标称值的时间。默认值为零
backup               将当前服务器标记为备份服务器。当主服务器不可用时，向备用服务器传递请求
down                 标记当前服务器为永不可用；和 ip_hash 指令一起使用
```
举例：
```
[plain] view plaincopyprint?
upstream backend {  
    server backend1.example.com     weight=5;  
    server 127.0.0.1:8080           max_fails=3 fail_timeout=30s;  
    server unix:/tmp/backend3;  
    server backup1.example.com:8080 backup;  
}  
```
如果群里面只有一台主机，那么 max_fails、 fail_timeout 和 slow_start 参数将被忽略，而且这样的主机也永远不会被认为不可用

语法：zone name size; 默认值：— <br>
上下文：upstream <br>
使群动态可配。定义持有群的配置和工作进程之间共享的运行时状态的共享内存区域的名字和大小。这样的群允许在运行时添加、删除和修改服务器。这个配置通过 upstream_conf 进行访问。这一指令仅作为商业订购的一部分

语法：ip_hash; 默认值：— <br>
上下文：upstream <br>
指定一个使用负载均衡方法根据客户端 IP 地址将请求分发给一些服务器的群。客户 IPv4 地址或者 IPv6 地址的前三个位群作为一个散列键。这个方法可以使同一个客户端的常常被发送给同一台主机，除非这台主机是不可用状态。在这种情况下(该主机不可用)客户端请求会被传递到另一台主机。大多数情况下，它将被发送给同一台主机。Nginx 1.3.2 和 1.2.2 之后开始支持 IPv6 地址。如果服务器中的一台需要临时移除掉，那么它应该使用 down 参数标记以保持客户 IP 地址的当前散列 <br>
例如：
```
[plain] view plaincopyprint?
upstream backend {  
    ip_hash;  

    server backend1.example.com;  
    server backend2.example.com;  
    server backend3.example.com down;  
    server backend4.example.com;  
}  
```
在 Nginx 1.3.1 和 1.2.2 版本之前的版本是无法使用 ip_hash 负载均衡方式定义服务器权重的

语法：keepalive connections; 默认：— <br>
上下文：upstream <br>
这一指令在 Nginx 版本 1.1.4 之后出现，激活 upstream 服务器的连接缓存。connections 参数设置保存在每个工作进程缓存中的 upstream 主机的闲置 keepalive 连接的最大个数。超出这个数目时，最近很少使用的连接被关闭。应该特别指出的是 keepalive 指令并没有限制一个 Nginx 工作进程所能承载的连接总量。connections 应该设置为一个足够小的值以使 upstream 服务器也足以应对新的连接

带有 keepalive 连接的 memcached upstream 配置例子：
```
[plain] view plaincopyprint?
upstream memcached_backend {  
    server 127.0.0.1:11211;  
    server 10.0.0.2:11211;  
    keepalive 32;  
}  

server {  
    ...  

    location /memcached/ {  
        set $memcached_key $uri;  
        memcached_pass memcached_backend;  
    }  

}  
```
对于 HTTP，proxy_http_version 指令应该设置为 "1.1"，并且清空 "Connection" 头字段：
```
[plain] view plaincopyprint?
upstream http_backend {  
    server 127.0.0.1:8080;  
    keepalive 16;  
}  

server {  
    ...  

    location /http/ {  
        proxy_pass http://http_backend;  
        proxy_http_version 1.1;  
        proxy_set_header Connection "";  
        ...  
    }  
}  
```
作为一种选择，HTTP/1.0 持久连接可以通过传递 "Connection: Keep-Alive" 头字段到 upstream 服务器来使用，虽然这种方法并不被推荐。
对于 FastCGI 服务器，需要为持久连接设置 fastcgi_keep_conn：
```
[plain] view plaincopyprint?
upstream fastcgi_backend {  
    server 127.0.0.1:9000;  
    keepalive 8;  
}  

server {  
    ...  

    location /fastcgi/ {  
        fastcgi_pass fastcgi_backend;  
        fastcgi_keep_conn on;  
        ...  
    }  
}  
```


当使用 round-robin 之外的负载均衡方法时，需要在 keepalive 指令之前将他们激活, SCGI 和 uwsgi 协议没有长连接的概念 <br>
语法：least_conn; <br>
默认值：— <br>
上下文：upstream <br>
描述：这一指令出现在版本 1.3.1 和 1.2.2 中 <br>
定义一群应该在请求传递给具有最小有效连接的服务器时使用的负载均衡方法，要考虑到服务器的权重。如果有很多这样的服务器，将会使用带权重的 round-robin 方法

语法：health_check [interval=time] [fails=number] [passes=number] [uri=uri] [match=name]; <br>
默认值：— <br>
上下文：location 激活定期健康检查
支持以下可选参数：
```
interval：两次连续性健康检查的间隔时间，默认为 5 秒;
fails：   设置连续性健康检查失败的次数，超过这个次数的话这台服务器就被认为是不健康的，默认为 1;
passes：  设置连续性健康检查通过的次数，超过这个次数的话这台服务器就被认为是健康的，默认为 1;
uri：     定义健康检查请求用的 URI，默认为 "/";
match：   定义健康检查返回通过的匹配块的配置；默认情况下，返回应该具有状态码2XX或者3XX;
```
例如:
```
[plain] view plaincopyprint?
location / {  
    proxy_pass http://backend;  
    health_check;  
}  
```
将会每隔五秒钟发送 "/" 到 backend 群的每个服务器。如果有任何通信错误或者超时发生，或者代理服务器返回为 2XX 或者 3XX 之外的状态码，健康检查失败，这台服务器就被认为是不健康的了。来自客户端的请求不会被传递给不健康的服务器。 <br>
健康检查可以被配置成测试返回的状态码，头字段以及其值，还有正文内容。测试单独地使用 match 参数引用到的 match 指令进行配置，例如：
```
[plain] view plaincopyprint?
http {  
    server {  
    ...  
        location / {  
            proxy_pass http://backend;  
            health_check match=welcome;  
        }  
    }  

    match welcome {  
        status 200;  
        header Content-Type = text/html;  
        body ~ "Welcome to nginx!";  
    }  
}  
```
这一配置说明了健康检查通过的条件是，健康检查请求应该成功返回，拥有状态码 200，Content-Type 是为 "text/html"，并且在正文中包含 "Welcome to nginx!"。服务器群必须属于共享内存。如果一些健康检查是为同一群服务器而定义，一个失败的任何检查就会使相关服务器被认为是不健康的。 <br>
语法：match name { ... } <br>  <br>
默认值：—  <br>
上下文：http  <br>
定义已命名测试设置，用于核对健康检查请求的返回。  <br>
可以在一个返回中进行以下项目测试：

* status 200           状态码为 200
* status ! 500;        状态码不是 500
* status 200 204;      状态码为 200 或者 400
* status ! 301 302;    状态码既不是 301 也不是 302
* status 200-399;      状态码在 200 - 399 之间
* status ! 400-599;    状态码不在 400 - 599 之间
* status 301-303 307;  状态码是 301，302，303，或者 307
* header Host;         头包含字段 "Host"
* header Content-Type = text/html;     头包含字段 "Content-Type" 值为 text/html
* header Content-Type != text/html;    头包含字段 "Content-Type" 值不是 text/html
* header Connection ~ close;           头包含字段 "Connection" 值匹配正则表达式 close
* header Connection !~ close;          头包含字段 "Connection" 值不匹配正则表达式 close
* header ! X-Accel-Redirect;           头没有 "X-Accel-Redirect" 字段
* body ~ "Welcome to nginx!";          正文匹配正则表达式 "Welcome to nginx!"
* body !~ "Welcome to nginx!";         正文不匹配正则表达式 "Welcome to nginx!"

如果以上有些被定义，那么返回必须全都匹配时才能说明它测试通过。仅仅检查返回正文的头 256 k 个字节
例如：
```
[plain] view plaincopyprint?
# status is 200, content type is "text/html",  
# and body contains "Welcome to nginx!"  
match welcome {  
    status 200;  
    header Content-Type = text/html;  
    body ~ "Welcome to nginx!";  
}  
# status is not one of 301, 302, 303, or 307, and header does not have "Refresh:"  
match not_redirect {  
    status ! 301-303 307;  
    header ! Refresh;  
}  
# status ok and not in maintenance mode  
match server_ok {  
    status 200-399;  
    body !~ "maintenance mode";  
}  
```
语法：sticky cookie name [expires=time] [domain=domain] [path=path];  <br>
sticky route variable ...;
默认值：—  <br>
上下文：upstream  <br>
这一指令开始出现于版本 1.5.7。  <br>
激活会话关联，可以使来自同一客户端的请求总是传递给一群服务器中的同一个。例如：
```
[plain] view plaincopyprint?
upstream backend {  
    server backend1.example.com;  
    server backend2.example.com;  

    sticky_cookie_insert srv_id expires=1h domain=example.com path=/;  
}  
```
没有绑定到特定服务器的客户端请求会被传递到由配置的负载均衡方法选中的服务器。这个客户端的而后的请求将被传递到同一台服务器。如果指定服务器无法处理请求，一台新的服务器会被选中绑定，就像这个客户端的这次请求前没有绑定到任何服务器一样。 <br>
关于绑定服务器的信息保存在 HTTP cookie 中。第一个参数设置 cookie 名。其他参数如下：
```
expires   设置浏览器保持cookie的时间。特别值max将会使cookie在"31 Dec 2037 23:55:55 GMT"才过期
domain    定义设置的 cookie 的域名
path      定义设置的 cookie 的路径
如果任何一个参数被遗漏掉，相应的 cookie 属性就不会被设置上
```
语法：upstream_conf; <br>
默认值：—  <br>
上下文：location  <br>
开启 location 域的 HTTP upstream 配置接口。对于这一 location 的访问应该是受限的。  <br>
配置命令可以用于：
* 查看一群中的主要和备用服务器
* 查看一个特别的服务器
* 修改一个特别的服务器
* 添加一个新的服务器(参考下边注释)
* 移除一个特别的服务器

正如在 server 指令中提到的那样，指定一个服务器作为一个域名可能导致多个服务器被添加到群。因为地址在一群不需要是独特的，单个服务器在一个群可以被唯一引用的话只有用他们的 ID。服务器的 ID 会被自动分配，并在群配置中显示。
一个配置命令包括作为请求参数传递的参数，例如：<br>
http://127.0.0.1/upstream_conf?upstream=appservers <br>
支持以下参数：
```
upstream=name       选择一群。这一参数是强制必须的
backup=             如果没有设置，选中一群中的主要服务器。如果设置了，则选中一群中的备用服务器
id=number           选中一群中特定的主要服务器或者备用服务器
remove=             移调一群中一台特定的主要服务器或者备用服务器
add=                添加一台主要服务器或者备用服务器到群
server=address      同 server 指令中的 "address" 参数
weight=number       同 server 指令中的 "weight" 参数
max_fails=number    同 server 指令中的 "max_fails" 参数
fail_timeout=time   同 server 指令中的 "fail_timeout" 参数
slow_start=time     同 server 指令中的 "slow_start" 参数
down=               同 server 指令中的 "down" 参数
up=                 同 server 指令中的 "down" 参数相反
```
前三个参数用于选择命令适用的对象
比如，查看群组里头的主服务器，发送：http://127.0.0.1/upstream_conf?upstream=appservers
查看群组里头的备用服务器，发送：http://127.0.0.1/upstream_conf?upstream=appservers&backup=
查看群组里头特定的主服务器，发送：http://127.0.0.1/upstream_conf?upstream=appservers&id=42
查看群组里头特定的备用服务器，发送：http://127.0.0.1/upstream_conf?upstream=appservers&backup=&id=42
要添加一台主服务器或者备用服务器到群组，在 "server=" 参数中定义其地址即可。如果没有定义其他参数，该服务器添加后，其他参数设置为默认值(参见 server指令)
例如，添加一台新的主服务器到群组，发送：http://127.0.0.1/upstream_conf?add=&upstream=appservers&server=127.0.0.1:8080
添加一台新的从服务器到群组，发送：http://127.0.0.1/upstream_conf?add=&upstream=appservers&backup=&server=127.0.0.1:8080
添加一台主服务器到群组，设置其参数非默认值，且将其标记为 "down"，发送：
http://127.0.0.1/upstream_conf?add=&upstream=appservers&server=127.0.0.1:8080&weight=2&max_fails=3&fail_timeout=3s&down=
移除群组中的一台特定主服务器或者备用服务器，可以使用 id= 参数将其选择
例如，移除群组中的一台特定主服务器，发送：http://127.0.0.1/upstream_conf?remove=&upstream=appservers&id=42
移除群组中的一台特定从服务器，发送：http://127.0.0.1/upstream_conf?remove=&upstream=appservers&backup=&id=42
修改群组中的一台特定的主服务器或从服务器，也使用 id= 参数将其选中
例如，修改群组中一台特定主服务器为 "down"，发送：http://127.0.0.1/upstream_conf?upstream=appservers&id=42&down=
修改群组里头的一台备用服务器地址，发送：http://127.0.0.1/upstream_conf?upstream=appservers&backup=&id=42&server=192.0.2.3:8123
修改群组里头的一台主服务器的其他参数，发送：http://127.0.0.1/upstream_conf?upstream=appservers&id=42&max_fails=3&weight=4

ngx_http_upstream_module 模块支持以下嵌入式变量： <br>
$upstream_addr ---  为 UNIX-domain socket 保存服务器地址及端口号。如果请求处理时涉及多台服务器，使用逗号将他们的地址进行分隔，比如 "192.168.1.1:80, 192.168.1.2:80, unix:/tmp/sock"。如果一个由 "X-Accel-Redirect" 或者错误页面 发出的从一个服务器群组到另一个群组的重定向发生时，不同群组之间的服务器地址使用冒号分隔，比如 "192.168.1.1:80, 192.168.1.2:80, unix:/tmp/sock : 192.168.10.1:80, 192.168.10.2:80"

$upstream_cache_status  ---  保存访问响应缓存的状态。这一状态可以是 "MISS"，"BYPASS"，"EXPIRED"，"STALE"，"UPDATING" 或者 "HIT"

$upstream_response_length 保存从 upstream 服务器获得的响应体长度(版本 0.7.27)字节的长度。几个响应长度的话使用逗号和冒号分隔，就像 $upstream_addr 中的地址那样

$upstream_response_time   保存从 upstream 服务器获得的响应次数，长度以毫秒的分辨率保存，单位是秒。几个响应次数的话使用逗号和冒号分隔，就像 $upstream_addr 中的地址那样

$upstream_status    保存从 upstream 服务器获得的响应码。几个响应码的话使用逗号和冒号分隔，就像 $upstream_addr 中的地址那样

$upstream_http_...  保存服务器响应头。比如，"Server" 响应头可以使用 $upstream_http_server 参数激活。将头信息转化为参数名字的规则和以 "$http_" 前缀开始的参数规则一样。只保存最后一个响应头

原文链接：http://nginx.org/en/docs/http/ngx_http_upstream_module.html
