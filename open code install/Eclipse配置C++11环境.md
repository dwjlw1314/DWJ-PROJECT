<font color=#FF0000 size=5><p align="center">C++11配置跨工程文件编译，内联文件编译</p></font>

1.菜单“Project”——“Properties”——“C++ General”，如图所示：

![image](https://github.com/dwjlw1314/DWJ-PROJECT/raw/master/PictureSource/3.6.1.png)

2.“C++ Build”——“Settings”，如图所示：

![image](https://github.com/dwjlw1314/DWJ-PROJECT/raw/master/PictureSource/3.6.2.png)

3.头文件路径设置如图所示：

![image](https://github.com/dwjlw1314/DWJ-PROJECT/raw/master/PictureSource/3.6.3.png)

4.预定义设置如图所示：

![image](https://github.com/dwjlw1314/DWJ-PROJECT/raw/master/PictureSource/3.6.4.png)

5."C++ Linker"配置（此处解决内联文件编译支持c++11的问题），如图所示：

![image](https://github.com/dwjlw1314/DWJ-PROJECT/raw/master/PictureSource/3.6.5.png)

6.链接库与路径

![image](https://github.com/dwjlw1314/DWJ-PROJECT/raw/master/PictureSource/3.6.6.png)

7.指定编译后链接库路径与其他工程文件

![image](https://github.com/dwjlw1314/DWJ-PROJECT/raw/master/PictureSource/3.6.7.png)

<font color=#FF0000 size=5><p align="center">设置Eclipse支持C++ 11</p></font>

1.Properties > C/C++ Build > Setting > GCC C++ Compiler > Miscellaneous，在Other flags原有参数后添加-std=c++0x

![image](https://github.com/dwjlw1314/DWJ-PROJECT/raw/master/PictureSource/3.6.8.jpg)

2.Properties > C/C++ General > Paths and Sysmbols > Symbols > Gun C++，Add添加Name=GXX_EXPERIMENTAL_CXX0X，其余留空

![image](https://github.com/dwjlw1314/DWJ-PROJECT/raw/master/PictureSource/3.6.9.jpg)
