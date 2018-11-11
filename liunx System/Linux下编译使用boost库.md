一、编译安装<br>
1.先去Boost官网下载最新的Boost版本, 我下载的是boost_1_59_0版本，进行减压
>[root@dwj Desktop]# tar -zxvf boost_1_59_0.tar.gz

2.进行编译的安装
>[root@dwj boost_1_59_0]# ./b2 --prefix=/usr/local/boost-1-59-0 install

该命令把boost的头文件文件夹 include/ 安装在prefix定义的目录中, 并且会编译所有的boost模块, 并将编译好的库文件夹 lib/
也放在prefix定义的目录中. 所有成功编译的的话, prefix目录即  /usr/local/boost-1-59-0  目录会包含 include 和 lib 两个文件夹

二、功能测试

1.先测试只依赖头文件的功能模块，将下面的代码保存为 test.cpp
```c++
    #include<boost/lambda/lambda.hpp>
    #include<iostream>
    #include<iterator>
    #include<algorithm>

    int main()
    {
            using namespace boost::lambda;
            typedef std::istream_iterator<int>in;
            std::for_each(
            in(std::cin), in(), std::cout << (_1 * 3)<< " ");
    }
```

保存后编译
>[root@dwj opt]# g++ test.cpp -o test -I /usr/local/boost-1-59-0/include <br>
-I: 大写的i, 指定头文件搜索目录，执行 ./test 测试, 输入一个数, 返回这个数乘3的值

2.再测试需要用到二进制库的功能模块，将下面的代码保存为 banary.cpp
```c++
    #include <iostream>
    #include <boost/filesystem.hpp>

    using namespace boost::filesystem;

    int main(int argc, char *argv[])
    {
         if (argc < 2)
         {
               std::cout << "Usage:tut1 path\n";
               return 1;
          }
          std::cout << argv[1] << " "<<file_size(argv[1])<<std::endl;
          return 0;
    }
```

编译的时候需要注意
>[root@dwj opt]# g++ banary.cpp -I /usr/local/boost-1-59-0/include -L /usr/local/boost-1-59-0/lib -lboost_system -lboost_filesystem <br>
-L: 后接boost库文件夹，-l: 这是小写的 L, 接源文件编译所需用到的库文件, 注意使用 -l；要注意, 库文件之间也存在依赖关系, 比如这里 boost_filesystem库依赖于boost_system库, 所以boost_filesystem要写在后面, 否则可能会出现符号解析错误，执行 ./a.out，输出Usage:tut1 path
