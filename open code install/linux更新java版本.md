三种配置环境变量的方法

1.修改/etc/profile文件,仅开发时推荐使用这种方法，因为所有用户的shell都有权使用这些环境变量，可能会给系统带来安全性问题
>[root@dwj eclipse]# vim /etc/profile    #在profile文件末尾加入：

```
export JAVA_HOME=/usr/share/jdk1.6.0_14      #是实际最新java包的路径
export PATH=$JAVA_HOME/bin:$PATH
export CLASSPATH=.:$JAVA_HOME/lib/dt.jar:$JAVA_HOME/lib/tools.jar
```
>[root@dwj eclipse]# source /etc/profile            #让系统环境变量重新加载生效

2.修改.bash_profile文件,它可以把使用这些环境变量的权限控制到用户级别，如果需要给某个用户权限使用这些环境变量，只需要修改其个人用户主目录下的.bash_profile文件就可以了
>[root@dwj eclipse]# vim .bash_profile    #在.bash_profile文件末尾加入：

```
export JAVA_HOME=/usr/share/jdk1.6.0_14               #是实际最新java包的路径  
export PATH=$JAVA_HOME/bin:$PATH
export CLASSPATH=:$JAVA_HOME/lib/dt.jar:$JAVA_HOME/lib/tools.jar
```
>[root@dwj eclipse]# source .bash_profile                   #让用户环境变量重新加载生效

3.直接在shell下设置变量,换个shell，该设置就无效了，需要重新设置，因此这种方法是临时使用,只需在shell终端执行下列命令：
```
export JAVA_HOME=/usr/share/jdk1.6.0_14               #是实际最新java包的路径
export PATH=$JAVA_HOME/bin:$PATH
export CLASSPATH=.:$JAVA_HOME/lib/dt.jar:$JAVA_HOME/lib/tools.jar
```

测试jdk,用文本编辑器新建一个Test.java文件，在其中输入以下代码并保存：
```
public class test {
  public static void main(String args[]) {
    System.out.println("A new jdk test !");
  }
}
```
编译
>[root@dwj eclipse]# javac Test.java

运行
>[root@dwj eclipse]# java Test

当shell下出现“A new jdk test !”字样则jdk运行正常

卸载jdk,找到jdk安装目录的_uninst子目录
>[root@dwj eclipse]# ./uninstall.sh  <br>
Desire has no rest.
