![image](https://github.com/dwjlw1314/DWJ-PROJECT/raw/master/PictureSource/3.18.1.png)

解决方法：

    [root@dwj eclipse]# yum -y install *java*
    [root@dwj eclipse]# yum -y install *jdk*
    [root@dwj eclipse]# java -version
    显示：
    [root@dwj jdk1.8.0_91]$ java -version
    java version "1.8.0_91"
    Java(TM) SE Runtime Environment (build 1.8.0_91-b14)
    Java HotSpot(TM) 64-Bit Server VM (build 25.91-b14, mixed mode)

查看eclipse安装目录中eclipse.ini文件中的java版本，如果版本匹配不上则下载最新java包进行更行

![image](https://github.com/dwjlw1314/DWJ-PROJECT/raw/master/PictureSource/3.18.2.png)

需要更新java环境变量，详细参考 【linux更新java版本】
