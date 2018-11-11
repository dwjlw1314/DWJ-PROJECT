```
全局配置文件 /etc/yum.conf
yum日志文件 /var/log/yum.log
repository部分定义了每个源/服务器的具体配置
```
yum.conf 字段详细内容：
```
[main]
cachedir=/var/cache/yum　　     #yum缓存的目录，yum在此存储下载的rpm包和数据库，默认设置为/var/cache/yum
keepcache=0                    #安装完成后是否保留软件包，0为不保留（默认为0），1为保留
debuglevel=2                   #Debug信息输出等级，范围为0-10，缺省为2
logfile=/var/log/yum.log 　　   #yum日志文件位置。用户可以到/var/log/yum.log 文件去查询过去所做的更新。
pkgpolicy=newest            　　#包的策略。一共有两个选项，newest和last，这个作用是如果你设置了多个repository，而同一软件在不同的repository中同时存在，yum应该安装哪一个，如果是newest，则yum会安装最新的那个版本。如果是last，则yum会将服务器id以字母表排序，并选择最后的那个服务器上的软件安装,一般都是选newest
distroverpkg=redhat-release    #指定一个软件包，yum会根据这个包判断你的发行版本，默认是redhat-release，也可以是安装的任何针对自己发行版的rpm包
tolerant=1                     #有1和0两个选项，表示yum是否容忍命令行发生与软件包有关的错误，比如你要安装1,2,3三个包，而其中3此前已经安装，如果你设为1,则yum不会出现错误信息。默认是0
exactarch=1                    #有1和0两个选项，设置为1，则yum只会安装和系统架构匹配的软件包，例如，yum不会将i686的软件包安装在适合i386的系统中。默认为1
timeout=120                    #改善网络慢问题，增加yum的超时时间
retries=6                      #网络连接发生错误后的重试次数，如果设为0，则会无限重试。默认值为6
obsoletes=1                    #这是一个update的参数，具体请参阅yum(8)，简单的说就是相当于upgrade，允许更新陈旧的RPM包
plugins=1                      #是否启用插件，默认1为允许，0表示不允许。我们一般会用yum-fastestmirror这个插件
bugtracker_url=http://bugs.centos.org/set_project.php?project_id=16&ref=http://bugs.centos.org/bug_report_page.php?category=yum

# Note: yum-RHN-plugin doesn't honor this.
metadata_expire=1h
installonly_limit = 5

# PUT YOUR REPOS HERE OR IN separate files named file.repo
# in /etc/yum.repos.d

除了上述之外，还有一些可以添加的选项，如：
exclude=selinux*　  #排除某些软件在升级名单之外，可以用通配符，列表中各个项目要用空格隔开，这个对于安装了诸如美化包，中文补丁的朋友特别有用
gpgcheck=1          #有1和0两个选择，分别代表是否是否进行gpg(GNU Private Guard) 校验，以确定rpm包的来源是有效和安全的。这个选项如果设置在[main]部分，则对每个repository都有效。默认值为0
```
前提是已经通过mount命令挂载了iso文件，路径为 /mnt/Server
```
[root@dwj /]# rpm -qa|grep yum                         #查看系统默认安装的yum  
[root@dwj /]# rpm -qa|grep yum|xargs rpm -e --nodeps   #删除redhat原有的yum
[root@dwj /]# cd /mnt/Packages                         #进去系统安装包目录
```
通过find命令查找到对应的yum包 ---> find *yum*
>[root@dwj /]# rpm ivh 对应yum包名称         #安装yum程序

 1.以本地ISO镜像为例，进行yum源配置：
 >[root@dwj /]# cd /etc/yum.repos.d/       #进入yum配置目录

该目录下建立以".repo"结尾的文件，这里我建立的是rhel-media.repo。默认ISO镜像里有四类软件包，我这里建立的是常用的Server包,如果有类似文件，可以直接进行编辑，无须创建
```
[root@dwj yum.repos.d]# vim  rhel-media.repo              #编辑配置文件，添加以下内容
[rhel-media]
name=Red Hat Enterprise Linux $releasever - $basearch - Source      #自定义名称
baseurl=file:///mnt/Server                                          #本地光盘挂载路径
enabled=1                                                           #启用yum源，0为不启用，1为启用
gpgcheck=0                                                          #检查GPG-KEY，0为不检查，1为检查
gpgkey=file:///etc/pki/rpm-gpg/RPM-GPG-KEY-redhat-release           #GPG-KEY路径
```
```
[root@dwj yum.repos.d]# yum clean all                         #清除yum缓存
[root@dwj yum.repos.d]# yum makecache                         #缓存本地yum源中的软件包信息
[root@dwj yum.repos.d]# yum -y install *packet*               #可以安装软件包
```
常用命令如下：
```
yum install package1          #安装指定的安装包package1
yum groupinsall group1        #安装程序组group1
yum provides "*/nmcli"        #查看nmcli特定文件属于哪个软件包
yum update package1           #更新指定程序包package1
yum check-update              #检查可更新的程序
yum upgrade package1          #升级指定程序包package1
yum groupupdate group1        #升级程序组group1
yum info package1             #示安装包信息package1
yum list                      #显示所有已经安装和可以安装的程序包
yum list package1             #显示指定程序包安装情况package1
yum remove  package1          #删除程序包package1
yum  earse  package1          #删除程序包package1
yum groupremove group1        #删除程序组group1
```
