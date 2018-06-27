查看PATH：echo $PATH<br>
以添加mongodb server为列

修改方法一：<br>
>[root@dwj ~]# export PATH=/usr/local/mongodb/bin:$PATH <br>
生效方法：立即生效<br>
有效期限：临时改变，只能在当前的终端窗口中有效，当前窗口关闭后就会恢复原有的path配置<br>
用户局限：仅对当前用户

修改方法二：<br>
通过修改.bashrc文件:  <br>
>[root@dwj ~]# vim ~/.bashrc <br>
在最后一行添上： export PATH=/usr/local/mongodb/bin:$PATH <br>
生效方法：（有以下两种）<br>
1、关闭当前终端窗口，重新打开一个新终端窗口就能生效<br>
2、输入[root@dwj ~]# source ~/.bashrc 命令，立即生效，多次运行，会重复生成<br>
有效期限：永久有效<br>
用户局限：仅对当前用户

修改方法三:<br>
通过修改profile文件: <br>
>[root@dwj ~]# vim /etc/profile <br>
在vim编辑界面输入/export PATH --> 找到设置PATH的行，添加下面行 <br>
export PATH=/usr/local/mongodb/bin:$PATH <br>
生效方法：输入[root@dwj ~]# source  /etc/profile 命令，立即生效，多次运行，会重复生成<br>
有效期限：永久有效<br>
用户局限：对所有用户

修改方法四:<br>
通过修改environment文件:  <br>
>[root@dwj ~]# vim /etc/environment <br>
在PATH="/usr/local/sbin:/usr/local/bin:/usr/sbin:/usr/bin:/sbin:/bin:/usr/games:/usr/local/games"中加入“:/usr/local/mongodb/bin” <br>
生效方法：系统重启<br>
有效期限：永久有效<br>
用户局限：对所有用户
