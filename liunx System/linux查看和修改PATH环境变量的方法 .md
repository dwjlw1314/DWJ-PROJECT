查看PATH：
>[root@dwj ~]# echo $PATH

以添加mongodb server为例

修改方法一：
>[root@dwj ~]# export PATH=/usr/local/mongodb/bin:$PATH
```
生效方法：立即生效
有效期限：临时改变，只能在当前的终端窗口中有效，当前窗口关闭后就会恢复原有的path配置
用户局限：仅对当前用户
```

修改方法二：
>[root@dwj ~]# vim ~/.bashrc     修改.bashrc文件

```
在最后一行添上： export PATH=/usr/local/mongodb/bin:$PATH
生效方法：（有以下两种）
1、关闭当前终端窗口，重新打开一个新终端窗口就能生效
2、输入[root@dwj ~]# source ~/.bashrc 命令，立即生效，多次运行，会重复生成
有效期限：永久有效
用户局限：仅对当前用户
```
修改方法三:
>[root@dwj ~]# vim /etc/profile    #修改profile文件
```
在vim编辑界面输入/export PATH --> 找到设置PATH的行，添加下面行
export PATH=/usr/local/mongodb/bin:$PATH
生效方法：输入[root@dwj ~]# source  /etc/profile 命令，立即生效，多次运行，会重复生成
有效期限：永久有效
用户局限：对所有用户
```

修改方法四:
>[root@dwj ~]# vim /etc/environment   修改environment文件
```
在PATH="/usr/local/sbin:/usr/local/bin:..."中加入“:/usr/local/mongodb/bin”
生效方法：系统重启
有效期限：永久有效
用户局限：对所有用户
```
