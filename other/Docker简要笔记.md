虚拟化是一种资源管理技术，是将计算机的各种实体资源，如服务器，网络，内存及存储等，予以抽象，转换后呈现出来，
打破实体结构间的不可切割的障碍，使用户可以比原来组态更好的方式来应用这些资源

Docker的目标是实现轻量级的操作系统虚拟化解决方案。Docker的基础是Linux容器(LXC)等技术，它基于Go语言实现

容器是在操作系统层面上实现虚拟化，直接复用本地主机的操作系统，而传统方式则是在硬件层面实现
![image](https://github.com/dwjlw1314/DWJ-PROJECT/raw/master/PictureSource/7.3.1.jpg)

Docker跟传统的虚拟化方式相比的优势:
```
1.Docker容器可以实现秒级启动，其次Docker对系统资源利用率很高，且基本不消耗额外的系统资源，一台主机可以同时运行数千个Docker容器
2.Docker容器的运行不需要额外的hypervisor(虚拟机监视器)支持，它是内核级的虚拟化，因此可以实现更高的性能和效率
3.Docker只需要小改动就可以替代以往大量的更新工作。所有的修改都以增量的方式被分发和更新，从而实现高效的自动化管理
4.更轻松的迁移和扩展
5.更快速的交付和部署
```
Dockerr整个生命周期有三个基本概念:1.镜像(Image)、2.容器(Container)、3.仓库(Repository)
```
Docker镜像就是一个只读的模板，可以用来创建Docker容器。提供了一个简单的机制来创建镜像或者更新现有的镜像

Docker容器用来运行应用程序，它是从镜像创建的运行实例。它可以被启动、开始、停止、删除。每个容器都是相互隔离的、保证平台安全，
容器是一个简易版的Linux环境(包括root用户权限、进程空间、用户空间和网络空间等)和运行在其中的应用程序，镜像是只读的，
容器在启动的时候在最上层创建一层可写层

Docker仓库是集中存放镜像文件的场所。仓库和仓库注册服务器(Registry)并不严格区分。实际上，仓库注册服务器上往往存放着多个仓库，
每个仓库中又包含了多个镜像，每个镜像有不同的标签(tag)，仓库分为公开和私有仓库两种形式
```

查看linux系统版本信息
>[root@dwj /]# lsb_release -a

<font color=#FF0000 size=4> <p align="center">Docker安装</p></font>

官方网站上有各种环境下的安装指南，该文档是CentOS的安装

<font color=#FF0000 size=4> <p align="center">Docker基本命令</p></font>

docker启动命令
>[root@dwj /]# systemctl start docker

获取docker所有启动参数
>[root@dwj /]# docker help run

查看镜像层组成和大小
>[root@dwj /]# docker history imageName

通过镜像创建容器(-p大小写不一样)
>[root@dwj /]# docker run -itd --gpus all --privileged --name caailast_dwj -p hostport:containerport -e NVIDIA_DRIVER_CAPABILITIES=compute,utility,video,display,graphics -e NVIDIA_VISIBLE_DEVICES=all caai0318 /bin/bash

进入启动的镜像，加权限
>[root@dwj /]# sudo docker exec -it 775c7c9ee1e1 /bin/bash

通过镜像创建容器，挂载宿主机的/test目录到容器的/soft目录
>[root@dwj /]# docker run -it -v /test:/soft centos /bin/bash

特权模式启动容器
>[root@dwj /]# docker run -it --privileged centos /bin/bash

启动容器,containerId是容器的ID
>[root@dwj /]# docker start containerId

停止容器
>[root@dwj /]# docker stop containerId

stop停止所有容器
>[root@dwj /]# docker stop $(docker ps -a -q)

查看指定容器启动参数详情
>[root@dwj /]# docker inspect containerId

查看容器的IP
>[root@dwj /]# docker network inspect bridge  <br>
>[root@dwj /]# ip addr show docker0

查看路由信息
>[root@dwj /]# ip route show

容器转镜像
>[root@dwj /]# docker commit containerId videoAccess-1.0

镜像保存
>[root@dwj /]# docker save -o videoAccess.tar 4ce61b979dbb

镜像加载
>[root@dwj /]# docker load -i videoAccess.tar

<font color=#FF0000 size=4> <p align="center">Docker错误汇总</p></font>

1.进入docker容器后，想创建文件，但是提示 cannot touch 'xxx': Permission denied
```
解决方法：
第一种、进入容器的命令改为 sudo docker exec -it -u root 9b98c3dcb2d0 /bin/bash
第二种、创建容器实例的时候，增加参数--privileged=true
```

2.进入docker容器后，运行systemclt，但是提示 Failed to connect to bus: Host is down
```
原因：dbus-daemon没能启动,systemctl并不是不能使用。将CMD或者entrypoint设置为/usr/sbin/init即可
解决方法：
docker run --name test -it --privileged=true -p 8090:22 -v /home/:/home 300e315adb2f /usr/sbin/init
```

<font color=#FF0000 size=4> <p align="center">Dockerfile</p></font>
