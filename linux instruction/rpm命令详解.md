>[root@dwj /opt]#rpm -ivh yum                 #yum软件包安装

>[root@dwj /opt]#rpm -Uih yum.rpm             #升级rpm包

>[root@dwj /opt]#rpm -qi yum                  #yum软件包安装详情

>[root@dwj /opt]#rpm -qip http-2.3-4.rpm      #http未安装软件包详情

>[root@dwj /opt]#rpm -qf /etc/yum.conf        #查看yum.conf文件的rpm包

>[root@dwj /opt]#rpm -ql yum                  #yum软件包内容安装的路径

>[root@dwj /opt]#rpm -qR yum                  #yum软件包依赖包查询

>[root@dwj /opt]#rpm -qc yum                  #查找yum软件的配置文件路径

>[root@dwj /opt]#rpm -q --scripts bind        #查看dns服务器安装包安装之前脚本

>[root@dwj /opt]#rpm -V yum      #yum软件包的校验

```
验证内容具体描述如下：
S-------文件大小是否改变
M------文件类型或文件权限是否改变
5-------文件MD5校验和是否改变（内容是否改变）
D-------设备中的从代码是否改变
L-------文件路劲是否改变
U-------文件的所有者是否改变
G-------文件的属主是否改变
T-------文件的修改时间是否改变

文件类型描述如下：
c------配置文档（config file）
d------普通文档（documentation）
g------组文件（ghost file）
l------授权文件（license file）
r------描述文件（read me）
```
#通过rpm软件包进行已安装修复命令，首先可以通过rpm -qf查看损坏文件属于哪个rpm包
>[root@dwj /opt]#rpm2cpio coreutils-8.22-15.el7.x86_64.rpm | cpio -idv ./usr/bin/ls

```
参数(idv)具体描述：
  -i：copy-in模式，还原
  -d：还原时自动新建目录
  -v：显示还原过程
  .： 当前目录
```
然后可以查看当前目录提取的ls文件
