一. 在eclipse中加入.a静态链接库的方法

Project->Properties->C/C++ Build->Tool Settings->Miscellaneous->other Objects：在文件系统中找到*.a加入

程序在连接时使用了共享库，就必须在运行的时候能够找到共享库的位置。linux的可执行程序在执行的时候默认是先搜索/lib和/usr/lib这两个目录，然后按照 /etc/ld.so.conf里面的配置搜索绝对路径

同时，Linux也提供了环境变量LD_LIBRARY_PATH供用户选择使用，用户可以通过设定它来查找除默认路径之外的其他路径，如查找/work/lib路径,你可以在/etc/rc.d/rc.local或其他系统启动后即可执行到的脚本添加如下语句：LD_LIBRARY_PATH=/work/lib:$(LD_LIBRARY_PATH),并且LD_LIBRARY_PATH路径优先于系统默认路径之前查找,不过LD_LIBRARY_PATH的设定作用是全局的，过多的使用可能会影响到其他应用程序的运行，所以多用在调试

LD_LIBRARY_PATH的缺陷和使用准则，可以参考《Why LD_LIBRARY_PATH is bad》。通常情况下推荐还是使用gcc的-R或-rpath选项来在编译时就指定库的查找路径，并且该库的路径信息保存在可执行文件中，运行时它会直接到该路径查找库，避免了使用LD_LIBRARY_PATH环境变量查找

库的链接时路径和运行时路径：
现代连接器在处理动态库时将链接时路径（Link-time path）和运行时路径（Run-time path）分开,用户可以通过-L指定连接时库的路径，
通过-R（或-rpath）指定程序运行时库的路径，大大提高了库应用的灵活性。比如我们做嵌入式移植时
#arm-linux-gcc $(CFLAGS) –o target –L/work/lib/zlib/-llibz-1.2.3 (work/lib/zlib下是交叉编译好的zlib库)，
将target编译好后我们只要把zlib库拷贝到开发板的系统默认路径下即可，或者通过- rpath（或-R）、LD_LIBRARY_PATH指定查找路径

linux下的静态链接库 为.a文件，动态链接库为.so文件。用eclipse创建并使用动态链接库的方法如下：  <br>
一、eclipse创建动态链接库

1、new -> project -> c++ Project 选择Share Library -> Empty Project ,并输入工程名：share ，点击Finish，完成工程的创建

2、在share工程中编写代码，或者引入文件系统  <br>
引入文件系统的方法：右键 share工程 -> import ->General ->File System ->选择想要制作成动态链接库的文件

3、share工程中如果需要调用其他动态链接库，则需要添加其他动态链接库,再编译。否则直接进行编译。
编译之后，会在Debug目录下产生 libshare.so 文件，即是我们需要的动态链接库a

二、动态链接库的使用

1、编译时，在工程的include搜索路径中需要添加share工程的路径（即动态链接库 工程所在路径）,因为编译时要用到里面的头文件（.h文件）

2、添加要搜索的动态链接库库名，此处是libshare.so，以及搜索路径

3、添加环境变量 LD_LIBRARY_PATH ,并且它的值是share工程的路径（即动态链接库 工程所在路径）
或者编译的时候，将动态链接库的地址编译到工程中（当动态链接库又要调用自己创建的其他动态链接库时，此方法好用）
还有一种方法就是将动态链接库拷贝到 /usr/lib 或者 /lib 目录下

4、当运行时，系统会在LD_LIBRARY_PATH所包含的路径中去寻找动态链接库，即libshare.so，当然系统默认也会在/usr/lib 、/lib 目录下寻找动态链接库

5、新建动态链接库工程 new -> c++ Project -> Executable -> Empty Project  工程名为：mass

6、编写所需代码，包含动态链接库，此处是libshare.so,右键工程mass-> Properites -> C/C++ Build -> Settings -> Tool Settings  <br>
 选择GCC C++ Linker -> Libraries,在右上面的窗口处，点击+加号,添加share在下面窗口处点击 + 加号，添加动态链接库所在路径

7、添加include搜索路径，选择C/C++ General -> Path and Symbols ->Reference,会看到所有的工程名称，选择share工程，这样会自动在include 的搜索路径中加入 share所在路径。注意：在Configuration 一栏 最好选择 All Configurations，一般不支持

8、添加LD_LIBRARY_PATH环境变量，右键 mass 工程 -> Run As ->Run Configurations
![image](https://github.com/dwjlw1314/DWJ-PROJECT/raw/master/PictureSource/3.17.1.png)

左边选择 mass 工程 -> Environment -> New，name为LD_LIBRARY_PATH，value为${workspace_loc:/mylib/Debug}，
当然也可以不添加LD_LIBRARY_PATH ,而是将动态链接库的路径编译到工程中

这样程序能在eclipse中运行了，但我们想到终端运行却还是"error while loading sharedlibraries:..."，要在终端运行，需要设置该坏境变量
(export LD_LIBRARY_PATH=/root/workspace/mylib/Debug/)，很是不爽

三、其他方法
1、我们可以在mass工程中设置中加入一段，即工程Property -> C/C++build -> settings -> GCC CLinker-> Miscellaneous -> Link flags中添加 -Wl,-rpath=/root/workspace/mylib/Debug

2、或者在工程Property -> C/C++ build-> settings -> GCC CLinker-> Miscellaneous-> other option 中添加
-R"${workspace_loc:/mylib/Debug}"

3、Linux动态库的默认搜索路径是/lib和/usr/lib64等类似的目录，如果在这些地方找不到会按照配置文件/etc/ld.so.conf中指定的目录查找，所以我们可以在/etc/ld.so.conf.d目录下创建*.conf(如workspace.conf)配置文件，用来指定我们希望被查找到的lib*.so，例如往里写入/root/lib;在程序运行时，此目录便会被搜寻，我们可以将build好的lib*.so软连接到此
```
#cd /root/lib/
#ln -s /root/workspace/mylib/Debug/libmylib.so .  // <.>表示当前目录
#ldconfig
#cd /root/workspace/test/Debug/
#ldd test
查看连接情况如下所示：
libmylib.so =>/root/lib/libmylib.so (0x00002b5f040f3000)
libc.so.6 => /lib64/libc.so.6(0x0000003829a00000)
/lib64/ld-linux-x86-64.so.2(0x0000003828a00000)
```
