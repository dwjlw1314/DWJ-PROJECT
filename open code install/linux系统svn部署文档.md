方案1：采用非IDE集成

1.下载subversion-1.8.9.tar.gz 安装包（非新版本，可下载最新的linux svn客户端版本）

2.Linux下解压subversion-1.8.9.tar.gz安装包； 命令：tar -zxvf subversion-1.8.9.tar.gz

3.编译svn源码并安装； 命令：./configure    make && make install

4.验证svn安装是否成功； 命令：svn help

5.如果出现Available subcommands列表表示成功

方案2：采用IDE集成

1.下载site-1.10.6.zip 安装包

2.打开Eclipse开发环境，菜单选择help—install new software

3.![image](https://github.com/dwjlw1314/DWJ-PROJECT/raw/master/PictureSource/3.20.1.png)

4.![image](https://github.com/dwjlw1314/DWJ-PROJECT/raw/master/PictureSource/3.20.2.jpg)

5.![image](https://github.com/dwjlw1314/DWJ-PROJECT/raw/master/PictureSource/3.20.3.png)

6.选中Subclipes 和 SVNkit  然后一路next 即可完成安装

7.启动windows svn 服务(自定义安装)； 命令格式：svnserve -d -r D:\svnhome

8.Eclipse中新建工程，选择svn选项  <br>
![image](https://github.com/dwjlw1314/DWJ-PROJECT/raw/master/PictureSource/3.20.4.jpg)

9.选择Create a new repository location 然后next  <br>
![image](https://github.com/dwjlw1314/DWJ-PROJECT/raw/master/PictureSource/3.20.5.jpg)

10.url中输入对应的版本控制工程地址 ，next  <br>
![image](https://github.com/dwjlw1314/DWJ-PROJECT/raw/master/PictureSource/3.20.6.jpg)

11.![image](https://github.com/dwjlw1314/DWJ-PROJECT/raw/master/PictureSource/3.20.7.jpg)

12.![image](https://github.com/dwjlw1314/DWJ-PROJECT/raw/master/PictureSource/3.20.8.jpg)

13.![image](https://github.com/dwjlw1314/DWJ-PROJECT/raw/master/PictureSource/3.20.9.jpg)

14.Finish完成后eclipse工程浏览中出现callback工程

15.![image](https://github.com/dwjlw1314/DWJ-PROJECT/raw/master/PictureSource/3.20.10.jpg)

16.右键对应工程文件，选择相应的svn功能

17.![image](https://github.com/dwjlw1314/DWJ-PROJECT/raw/master/PictureSource/3.20.11.jpg)
