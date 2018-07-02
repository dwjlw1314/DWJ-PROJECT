Qt安装包主要包含以下几部分：
* Qt Library：也就是Qt的库，这是Qt的核心
* Qt Creator：官方开发的一款轻量级、跨平台的IDE，更换平台不需要重新学习
* Qt Designer：Qt程序的UI设计器。拖拽就可以开发简单的GUI程序，并且可以及时预览程序界面（无需编译）
* Qt Assistant：Qt帮助工具，包含了Qt教程、示例、类参考手册、模块介绍等，是Qt的官方资料，类似MSDN
* Qt Linguist：Qt语言包，是Qt的国际化工具，借助它可以很方便让程序支持多种语言，面向全球用户

<font color=#FF0000>在Windows下，GUI解决方案比较多，基于C++的有Qt、MFC、WTL、wxWidgets、DirectUI、Htmlayout，基于C#的有WinForm、WPF，基于Java的有AWT、Swing，基于Pascal的有Delphi，最新的aardio；有Web开发经验，可以基于Webkit或Chromium将网页转换为桌面程序，总起来说，Qt主要用于桌面程序开发和嵌入式开发</font> <br>

Qt是一个跨平台的框架。跨平台GUI通常有三种实现策略：

1. API 映射：API 映射是说，界面库使用同一套 API，将其映射到不同的底层平台上面。大体相当于将不同平台的 API 提取公共部分。比如说，将 Windows 平台上的按钮控件和 Mac OS 上的按钮组件都取名为 Button。当你使用 Button 时，如果在 Windows 平台上，则编译成按钮控件；如果在 Mac OS 上，则编译成按钮组件。优点是，所有组件都是原始平台自有的，外观和原生平台一致；缺点是，编写库代码的时候需要大量工作用于适配不同平台，并且，只能提取相同部分的 API。比如 Mac OS 的文本框自带拼写检测，但是 Windows 上面没有，则不能提供该功能。这种策略的典型代表是 wxWidgets。这也是一个标准的 C++ 库，和 Qt 一样庞大。它的语法看上去和 MFC 类似，有大量的宏

2. API 模拟：前面提到，API 映射会“缺失”不同平台的特定功能，而 API 模拟则是解决这一问题。不同平台的有差异 API，将使用工具库自己的代码用于模拟出来。按照前面的例子，Mac OS 上的文本框有拼写检测，但是 Windows 的没有。那么，工具库自己提供一个拼写检测算法，让 Windows 的文本框也有相同的功能。API 模拟的典型代表是 wine 是Linux上面的 Windows 模拟器。它将大部分 Win32 API 在Linux上面模拟了出来，让 Linux 可以通过 wine 运行 Windows 程序。由此可以看出，API 模拟最大优点是，应用程序无需重新编译，即可运行到特定平台上。一个例子是微软提供的 DirectX，这个开发库将屏蔽掉不同显卡硬件所提供的具体功能。使用这个库，你无需担心硬件之间的差异，如果有的显卡没有提供该种功能，SDK 会使用软件的方式加以实现

3. GUI 模拟：任何平台都提供了图形绘制函数，例如画点、画线、画面等。有些工具库利用这些基本函数，在不同绘制出自己的组件，这就是 GUI 模拟。GUI 模拟的工作量无疑是很大的，因为需要使用最基本的绘图函数将所有组件画出来；并且这种绘制很难保证和原生组件一模一样。但是，这一代价带来的优势是，可以很方便的修改组件的外观——只要修改组件绘制函数即可。很多跨平台的 GUI 库都是使用的这种策略，例如 Swing、Qt、gtk+（这是一个 C 语言的图形界面库。使用 C 语言很优雅地实现了面向对象程序设计。不过，这也同样带来了一个问题——使用大量的类型转换的宏来模拟多态，并且它的函数名一般都比较长，使用下划线分割单词，看上去和 Linux 如出一辙。gtk+ 并不是模拟的原生界面，而有它自己的风格，所以有时候就会和操作系统的界面格格不入）

<font color=#FF0000 size=4> <p align="center">signal和slot</p></font>
在Qt5中，QObject::connect() 有五个重载,返回值都是 QMetaObject::Connection：
```c++
QMetaObject::Connection connect(const QObject *, const char *,
                                const QObject *, const char *,
                                Qt::ConnectionType);
QMetaObject::Connection connect(const QObject *, const QMetaMethod &,
                                const QObject *, const QMetaMethod &,
                                Qt::ConnectionType);
QMetaObject::Connection connect(const QObject *, const char *,
                                const char *, Qt::ConnectionType) const;
QMetaObject::Connection connect(const QObject *, PointerToMemberFunction,
                                const QObject *, PointerToMemberFunction,
                                Qt::ConnectionType)
QMetaObject::Connection connect(const QObject *, PointerToMemberFunction, Functor);
--可以将每个函数看做是 QMetaMethod 的子类, Functor 参数是可以接受 static 函数、全局函数以及 Lambda 表达式
```
最常用的形式: connect(sender, signal, receiver, slot); <br>
>第一个是发出信号的对象，第二个是发送对象发出的信号，第三个是接收信号的对象，第四个是接收对象在接收到信号之后所需要调用的函数。也就是说，当 sender 发出了 signal 信号之后，会自动调用 receiver 的 slot 函数

1. 自定义类只有继承了 QObject 类，才具有信号槽的能力。凡是继承 QObject 类，都应该在第一行代码写上 Q_OBJECT宏。这个宏的展开将为我们的类提供信号槽机制、国际化机制以及 Qt 提供的不基于 C++ RTTI 的反射能力，这个宏将由 moc（可以将其理解为一种预处理器，是比 C++ 预处理器更早执行的预处理器） 做特殊处理，不仅仅是宏展开这么简单。moc 只会读取标记了 Q_OBJECT 的<头文件内容>，生成以 moc_ 为前缀的文件，例如 moc_file.cpp。如果类放到了cpp文件中，我们就需要手动调用 moc 工具处理 main.cpp，并且将 main.cpp 中的 include "header.h"改为 include "moc_header.h" 就可以了

2. 类中 signals 块所列出的一个个的函数名就是该类的信号，返回值是 void（因为无法获得信号的返回值，所以也就无需返回任何值），参数是传递到slot的数据。信号作为函数名，不需要在 cpp 函数中添加任何实现（Qt 程序能够使用普通的 make 进行编译。没有实现的函数名怎么会通过编译？这里 moc 会帮我们实现信号函数所需要的函数体。emit 是 Qt 对 C++ 的扩展，是一个关键字（其实也是一个宏）

3. 自定义信号槽需要注意的5个事项：1.发送者和接收者都需要是 QObject 的子类（当然，槽函数是全局函数、Lambda 表达式等无需接收者的时候除外）; 2.使用 signals 标记信号函数，信号是一个函数声明，返回 void，不需要实现函数代码; 3.槽函数是普通的成员函数，作为成员函数，会受到访问控制符的影响; 4.使用 QObject::connect() 函数连接信号和槽; 5.emit 在恰当的位置发送信号
