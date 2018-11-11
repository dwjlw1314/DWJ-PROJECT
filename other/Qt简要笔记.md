Qt安装包主要包含以下几部分：
```
* Qt Library：也就是Qt的库，这是Qt的核心
* Qt Creator：官方开发的一款轻量级、跨平台的IDE，更换平台不需要重新学习
* Qt Designer：Qt程序的UI设计器。拖拽就可以开发简单的GUI程序，并且可以及时预览程序界面（无需编译）
* Qt Assistant：Qt帮助工具，包含了Qt教程、示例、类参考手册、模块介绍等，是Qt的官方资料，类似MSDN
* Qt Linguist：Qt语言包，是Qt的国际化工具，借助它可以很方便让程序支持多种语言，面向全球用户
```

<font color=#FF0000>在Windows下，GUI解决方案基于C++的有Qt、MFC、WTL、wxWidgets、DirectUI、Htmlayout，基于C#的有WinForm、WPF，基于Java的有AWT、Swing，基于Pascal的有Delphi，最新的aardio；有Web开发经验，可以基于Webkit或Chromium将网页转换为桌面程序，总起来说，Qt主要用于桌面程序开发和嵌入式开发</font>

Qt是一个跨平台的框架。跨平台GUI通常有三种实现策略：

1.API 映射：API 映射是说，界面库使用同一套 API，将其映射到不同的底层平台上面。大体相当于将不同平台的 API 提取公共部分。比如说，将 Windows 平台上的按钮控件和 Mac OS 上的按钮组件都取名为 Button。当你使用 Button 时，如果在 Windows 平台上，则编译成按钮控件；如果在 Mac OS 上，则编译成按钮组件。优点是，所有组件都是原始平台自有的，外观和原生平台一致；缺点是，编写库代码的时候需要大量工作用于适配不同平台，并且，只能提取相同部分的 API。比如 Mac OS 的文本框自带拼写检测，但是 Windows 上面没有，则不能提供该功能。这种策略的典型代表是 wxWidgets。这也是一个标准的 C++ 库，和 Qt 一样庞大。它的语法看上去和 MFC 类似，有大量的宏

2.API 模拟：前面提到，API 映射会“缺失”不同平台的特定功能，而 API 模拟则是解决这一问题。不同平台的有差异 API，将使用工具库自己的代码用于模拟出来。按照前面的例子，Mac OS 上的文本框有拼写检测，但是 Windows 的没有。那么，工具库自己提供一个拼写检测算法，让 Windows 的文本框也有相同的功能。API 模拟的典型代表是 wine 是Linux上面的 Windows 模拟器。它将大部分 Win32 API 在Linux上面模拟了出来，让 Linux 可以通过 wine 运行 Windows 程序。由此可以看出，API 模拟最大优点是，应用程序无需重新编译，即可运行到特定平台上。一个例子是微软提供的 DirectX，这个开发库将屏蔽掉不同显卡硬件所提供的具体功能。使用这个库，你无需担心硬件之间的差异，如果有的显卡没有提供该种功能，SDK 会使用软件的方式加以实现

3.GUI 模拟：任何平台都提供了图形绘制函数，例如画点、画线、画面等。有些工具库利用这些基本函数，在不同绘制出自己的组件，这就是 GUI 模拟。GUI 模拟的工作量无疑是很大的，因为需要使用最基本的绘图函数将所有组件画出来；并且这种绘制很难保证和原生组件一模一样。但是，这一代价带来的优势是，可以很方便的修改组件的外观——只要修改组件绘制函数即可。很多跨平台的 GUI 库都是使用的这种策略，例如 Swing、Qt、gtk+（这是一个 C 语言的图形界面库。使用 C 语言很优雅地实现了面向对象程序设计。不过，这也同样带来了一个问题——使用大量的类型转换的宏来模拟多态，并且它的函数名一般都比较长，使用下划线分割单词，看上去和 Linux 如出一辙。gtk+ 并不是模拟的原生界面，而有它自己的风格，所以有时候就会和操作系统的界面格格不入）

Qt 5 模块分为 Essentials Modules 和 Add-on Modules 两部分。前者是基础模块，适用所有平台；后者是扩展模块，建立在基础模块之上

Qt 基础模块分有以下几个：
```
* Qt Core，提供核心的非 GUI 功能，所有模块都需要这个。这个模块的类包括了动画框架、定时器、各个容器类、时间日期类、事件、IO、JSON、插件机制、智能指针、图形（矩形、路径等）、线程、XML 等。所有这些类都可以通过 <QtCore> 头文件引入
* Qt Gui，提供 GUI 程序的基本功能，包括与窗口系统的集成、事件处理、OpenGL 和 OpenGL ES 集成、2D 图像、字体、拖放等。这些类一般由 Qt 用户界面类内部使用，当然也可以用于访问底层的 OpenGL ES 图像 API。Qt Gui 模块提供的是所有图形用户界面程序都需要的通用功能
* Qt Multimedia，提供视频、音频、收音机以及摄像头等功能。这些类可以通过 <QtMultimedia> 引入，而且需要在 pro 文件中添加 QT += multimedia
* Qt Network，提供跨平台的网络功能。这些类可以通过 <QtNetwork> 引入，而且需要在 pro 文件中添加 QT += network
* Qt Qml，提供供脚本语言 QML 使用的 C++ API。这些类可以通过 <QtQml> 引入，而且需要在 pro 文件中添加 QT += qml
* Qt Quick，允许在 Qt/C++ 程序中嵌入 Qt Quick (一种基于 Qt 的高度动画的用户界面，适合于移动平台开发)，这些类可以通过 <QtQuick> 引入，而且需要在 pro 文件中添加 QT += quick
* Qt SQL，允许使用 SQL 访问数据库。这些类可以通过 <QtSql> 引入，而且需要在 pro 文件中添加 QT += sql
* Qt Test，提供 Qt 程序的单元测试功能。这些类可以通过 <QtTest> 引入，而且需要在 pro 文件中添加 QT += testlib
* Qt Webkit，基于 WebKit2 的实现以及一套全新的 QML API(顺便说一下，Qt 4.8 附带的是 QtWebkit 2.2)
Qt 扩展模块：
* Qt 3D，提供声明式语法，在程序中可以简单地嵌入 3D 图像。为 Qt Quick 添加了 3D 内容渲染。Qt 3D 提供了 QML 和 C++ 两套 API，用于开发 3D 程序
* Qt Bluetooth，提供用于访问蓝牙无线设备的 C++ 和 QML API
* Qt Contacts，用于访问地址簿或者联系人数据库的 C++ 和 QML API
* Qt Concurrent，封装了底层线程技术的类库，方便开发多线程程序
* Qt D-Bus，这是一个仅供 Unix 平台使用的类库，用于利用 D-Bus 协议进行进程间交互
* Qt Graphical Effects，提供一系列用于实现图像特效的类，比如模糊、锐化等
* Qt Image Formats，支持图片格式的一系列插件，包括 TIFF、MNG、TGA 和 WBMP
* Qt JS Backend，该模块没有公开的 API，是 V8 JavaScript 引擎的一个移植。这个模块仅供 QtQml 模块内部使用
* Qt Location，提供定位机制、地图和导航技术、位置搜索等功能的 QML 和 C++ API
* Qt OpenGL，方便在 Qt 应用程序中使用 OpenGL。从 Qt 4 移植到 Qt 5 保留下来，在 Qt 5 程序需要使用的是 Qt Gui 模块中的 QOpenGL
* Qt Organizer，使用 QML 和 C++ API 访问组织事件（organizer event）。organizer API 是 Personal Information Management API 的一部分，用于访问 Calendar 信息。通过 Organizer API 可以实现：从日历数据库访问日历时间、导入 iCalendar 事件或者将自己的事件导出到 iCalendar
* Qt Print Support，提供对打印功能的支持
* Qt Publish and Subscribe，为应用程序提供对项目值的读取、导航、订阅等的功能
* Qt Quick 1，从 Qt 4 移植过来的 QtDeclarative 模块，用于提供与 Qt 4 的兼容。如果你需要开发新的程序，需要使用 QtQuick 模块
* Qt Script，提供脚本化机制。这也是为提供与 Qt 4 的兼容性，如果要使用脚本化支持，请使用 QtQml 模块的 QJS* 类
* Qt Script Tools，为使用了 Qt Script 模块的应用程序提供的额外的组件
* Qt Sensors，提供访问各类传感器的 QML 和 C++ 接口
* Qt Service Framework，提供客户端发现其他设备的服务。Qt Service Framework 为在不同平台上发现、实现和访问服务定义了一套统一的机制
* Qt SVG，提供渲染和创建 SVG 文件的功能
* Qt System Info，提供一套 API，用于发现系统相关的信息，比如电池使用量、锁屏、硬件特性等
* Qt Tools，提供了 Qt 开发的方便工具，包括 Qt CLucene、Qt Designer、Qt Help 以及 Qt UI Tools
* Qt Wayland，仅用于 Linux 平台，用于替代 QWS，包括 Qt Compositor API（server）和 Wayland 平台插件（clients）
* Qt WebKit，从 Qt 4 移植来的基于 WebKit1 和 QWidget 的 API
* Qt Widgets，使用 C++ 扩展的 Qt Gui 模块，提供了一些界面组件，比如按钮、单选框等
* Qt XML，SAX 和 DOM 的 C++ 实现。该模块已经废除，请使用 QXmlStreamReader/Writer
* Qt XML Patterns，提供对 XPath、XQuery、XSLT 和 XML Schema 验证的支持
```
Qt 4 也分成若干模块，但是这些模块与 Qt 5 有些许多不同。下面是 Qt 4 的模块：
```
* QQtCore，Qt 提供的非 GUI 核心类库，这一部分与 Qt 5 大致相同，只不过 Qt 4 的 core 类库中并不包含 JSON、XML 处理等
* QQtGui，图形用户界面组件，这个模块相当于 Qt 5 的 QtGui 与 QtWidgets 两个模块的总和
* QQtMultimedia，多媒体支持，类似 Qt 5 的相关部分
* QQtNetwork，网络支持，类似 Qt 5
* QQtOpenGL，提供对 OpenGL 的支持。在 Qt 5 中，这部分被移植到 QtGui 模块
* QQtOpenVG，提供对 OpenVG 的支持
* QQtScript，提供对 Qt Scripts 的支持。Qt Script 是一种类似于 JavaScript 的脚本语言。在 Qt 5 中，推荐使用 QtQml 的 JavaScript 部分
* QQtScriptTools，为 Qt Script 提供的额外组件
* QQtSql，提供对 SQL 数据库的支持
* QQtSvg，提供对 SVG 文件的支持
* QQtWebKit，提供显示和编辑 Web 内容
* QQtXml，XML 处理，这部分在 Qt 5 中被添加到了 QtCore
* QQtXmlPatterns，提供对 XQuery、XPath 等的支持
* QQtDeclarative，用于编写动画形式的图形用户界面的引擎
* QPhonon，多媒体框架
* QQt3Support，Qt 3 兼容类库
```
下面是 Qt 4 的一些工具模块：
```
* QQtDesigner，用于扩展 Qt Designer
* QQtUiTools，用于在自己的引用程序中处理 Qt Designer 生成的 form 文件
* QQtHelp，联机帮助
* QQtTest，单元测试
```
下面是专门供 Windows 平台的模块：
```
* QAxContainer，用于访问 ActiveX 控件
* QAxServer，用于编写 ActiveX 服务器
```
下面是专门供 Unix 平台的模块：

* QtDBus，使用 D-Bus 提供进程间交互

<font color=#FF0000 size=5> <p align="center">Qt creator支持的编译器以及对应调试器</p></font>

windows系统下主要的调试器：
```
CDB    只能调试用户程序,只有控制台界面，以命令行形式工作
NTSD   能调试用户程序,只有控制台界面，以命令行形式工作
KD     主要用于内核调试，有时候也用于用户态调试,只有控制台界面，以命令行形式工作
WinDbg 在用户态、内核态下都能够发挥调试功能,采用了可视化的用户界面
```

Platform | Compiler	| Native Debugger
---|---|---
Linux |	GCC/ICC	| GDB, LLDB (experimental)
Unix	|GCC/ICC |	GDB
macOS |	GCC/Clang	| LLDB,FSF GDB(experimental)
Windows/MinGW |	GCC	 | GDB
Windows/MSVC  |	Microsoft Visual C++ Compiler |	Debugging Tools for Windows/CDB

Qt 可视化组件的继承关系图

![image](https://github.com/dwjlw1314/DWJ-PROJECT/raw/master/PictureSource/7.2.1.png)

<font color=#FF0000 size=5> <p align="center">exec()、signal和slot</p></font>

```
标准 C++ 语言的限制：参数默认值只能使用在直接地函数调用中
当使用函数指针取其地址的时候，默认参数是不可见的，我们不能在函数指针中使用函数参数的默认值
```

exec() 函数是开始 Qt 的事件循环。当事件发生时，Qt 将创建一个事件对象。所有事件类都继承于 QEvent。事件对象创建完后，将这个事件对象传递给 QObject 的 event() 函数，该函数并不直接处理事件，而是按照事件对象的类型分派给特定的事件处理函数event handler

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
最常用的形式: connect(sender, signal, receiver, slot);

第一个是发出信号的对象，第二个是发送对象发出的信号，第三个是接收信号的对象，第四个是接收对象在接收到信号之后所需要调用的函数。也就是说，当 sender 发出了 signal 信号之后，会自动调用 receiver 的 slot 函数

1. 声明一个信号要使用 signals 关键字，在 signals 前面不能使用权限限定符，因为只有定义该信号的类及子类才可以发射该信号。而且信号只用声明，不能进行定义实现，同时信号没有返回值，只能是void。自定义类只有继承了 QObject 类，才具有信号槽的能力。凡是继承 QObject 类，都应该在第一行代码写上 Q_OBJECT 宏。这个宏的展开将为我们的类提供信号槽机制、国际化机制以及 Qt 提供的不基于 C++ RTTI 的反射能力，这个宏将由 moc（可以将其理解为一种预处理器，比 C++ 预处理器更早执行）做特殊处理，moc 只会读取标记了 Q_OBJECT 的<头文件内容>，生成以 moc_ 为前缀的文件

2. 类中 signals 块所列出的一个个的函数名就是该类的信号，返回值是 void（因为无法获得信号的返回值，所以也就无需返回任何值），参数是传递到slot的数据。信号作为函数名，不需要在 cpp 函数中添加任何实现（Qt 程序能够使用普通的 make 进行编译。没有实现的函数名怎么会通过编译？这里 moc 会帮我们实现信号函数所需要的函数体。发射信号使用 emit 关键字（其实也是一个宏），emit 是对 C++ 的扩展

3. 自定义信号槽需要注意的5个事项：1.发送者和接收者都需要是 QObject 的子类（当然，槽函数是全局函数、Lambda 表达式等无需接收者的时候除外）; 2.使用 signals 标记信号函数，信号是一个函数声明，返回 void，不需要实现函数代码; 3.槽函数是普通的成员函数，作为成员函数，会受到访问控制符的影响; 4.使用 QObject::connect() 函数连接信号和槽; 5.emit 在恰当的位置发送信号

4. 声明一个槽函数要使用 slots 关键字。一个槽可以是private、public或者是protected类型，槽函数也可以声明为虚函数，与普通函数一样，也可以像调用普通函数一样调用槽函数

connect 函数最后的参数 ConnectionType 常用值：

常量 | 描述
---|---
Qt::AutoConnection | 如果信号和槽在不同的线程中，同Qt::QueuedConnection；如果信号和槽在同一个线程中，同Qt::DirectConnection
Qt::DirectConnection | 发射完信号后立即执行槽，只有槽执行完成返回后，发射信号处后面的代码才可以执行
Qt::QueuedConnection | 接收部件所在线程的事件循环返回后再执行槽，无论槽执行与否，发射信号处后面的代码都会立即执行
Qt::BlockingQueuedConnection | 类似Qt::QueuedConnection，只能用在信号和槽在不同的线程情况下
Qt::UniqueConnection | 类似Qt::AutoConnection,但是两个对象间的相同的信号和槽只能有唯一的关联
Qt::AutoCompatconnection | 类似Qt::AutoConnection,它是Qt3中的默认类型

还有一种是信号和槽自动关联方式，eg：on_pushButton_clicked()由on，部件的objectName和信号3部分组成，中间用下划线隔开
需要在部件定义之后调用 QMetaObject::connectSlotsByName(this); 这样才能正确使用自动关联

<font color=#FF0000 size=5> <p align="center">Qt调试关联工具</p></font>

qt内存泄露检查：
```
Linux ,Mac OS X ： Valgrind
Windows： Visual Leak Detector for Visual C++ 2008-2015 (VLD, Open-source)
```

<font color=#FF0000 size=5> <p align="center">Qt moc</p></font>

实际在使用标准 C++ 编译器编译 Qt 源程序之前，Qt 扩展了标准 C++，先使用一个叫做 moc（Meta Object Compiler，元对象编译器）的工具，先对 Qt 源代码进行一次预处理（注意，这个预处理与标准 C++ 的预处理有所不同。Qt 的 moc 预处理发生在标准 C++ 预处理器工作之前，并且 Qt 的 moc 预处理不是递归的），生成标准 C++ 源代码，然后再使用标准 C++ 编译器进行编译。信号函数是不需要编写实现代码的，moc为信号函数这样的语法进行了处理，这样就可以通过标准 C++ 的编译了。类通过继承 QObject 类，可以很方便地获得这些特性。当然，这些特性都是由 moc 帮助实现的。moc 其实实现的是一个叫做元对象系统（meta-object system）的机制，正如上面所说，这是一个标准 C++ 的扩展，更适合于进行 GUI 编程。虽然利用模板可以达到类似的效果，但是 Qt 没有选择使用模板。按照 Qt 官方的说法，模板虽然是内置语言特性，但是其语法实在是复杂，并且由于 GUI 是动态的，利用静态的模板机制有时候很难处理。而使用 moc 生成代码更为灵活，虽然效率有些降低（一个信号槽的调用大约相当于四个模板函数调用），现代计算机上这点性能损耗实在是可以忽略的

Qt 使用 moc，为标准 C++ 增加了一些特性：
```
* 信号槽机制，用于解决对象之间的通讯，可以认为是 Qt 最明显的特性之一
* 可查询，并且可设计的对象属性
* 强大的事件机制以及事件过滤器
* 基于上下文的字符串翻译机制（国际化），也就是 tr() 函数
* 复杂的定时器实现，用于在事件驱动的 GUI 中嵌入能够精确控制的任务集成
* 层次化的可查询的对象树，提供一种自然的方式管理对象关系
* 智能指针（QPointer），在对象析构之后自动设为 0，防止野指针
* 能够跨越库边界的动态转换机制
```

<font color=#FF0000 size=5> <p align="center">Qt事件机制</p></font>

Qt 中有很多种事件：鼠标事件、键盘事件、大小改变的事件、位置移动的事件等等。处理这些事件有两种选择:
```
1. 所有事件对应一个事件处理函数，在这个事件处理函数中用一个很大的分支语句进行选择，其代表作就是 win32 API 的 WndProc() 函数
2. 每一种事件对应一个事件处理函数。Qt 就是使用这种机制,具有多种事件处理函数，需要有一个分发函数 event()对事件进行处理
```
Qt 提供了另外一种解决方案：事件过滤器，事件的调用最终都会追溯到 QCoreApplication::notify() 函数，最大的控制权实际上是重写该函数

Qt 的事件处理，实际上是有五个层次：
```
1. 重写 mousePressEvent() 等事件处理函数。这是最普通、最简单的形式，同时功能也最简单
2. 重写 event() 函数。该函数是所有对象的事件入口，QObject 和 QWidget 中的实现，默认是把事件传递给特定的事件处理函数
3. 在特定对象上面安装事件过滤器。该过滤器仅过滤该对象接收到的事件
4. 在 QCoreApplication::instance() 上面安装事件过滤器。该过滤器将过滤所有对象的所有事件，但是它更灵活，因为可以安装多个过滤器。全局的事件过滤器可以看到 disabled 组件上面发出的鼠标事件。全局过滤器有一个问题：只能用在主线程
5. 重写 QCoreApplication::notify() 函数。这是最强大的，和全局事件过滤器一样提供完全控制，并且不受线程的限制。但是全局范围内只能有一个被使用（因为 QCoreApplication 是单例的）
```

<font color=#FF0000 size=5> <p align="center">Qt对象树</p></font>

QObjects 是以对象树的形式组织起来的。当创建一个 QObject 对象时，会看到它的构造函数接收一个 QObject 指针作为参数，这个参数就是 parent，也就是父对象指针。这相当于，在创建 QObject 对象时，可以提供一个其父对象，创建的这个 QObject 对象会自动添加到其父对象的 children() 列表。当父对象析构的时候，这个列表中的所有对象也会被析构
```
标准 C++ （ISO/IEC 14882:2003）要求，局部对象的析构顺序应该按照其创建顺序的相反过程
```

<font color=#FF0000 size=5> <p align="center">Qt元对象系统</p></font>

```
1. QObject::metaObject()函数可以返回一个类的元对象，它是 QMetaObject 类的对象
2. QMetaObject::className()可以在运行时以字符串形式返回类名，而不是需要C++编辑器原生的运行时类型信息(RTTI)的支持
3. QObject::inherits()函数返回一个对象是否是 QObject 继承树上一个类的实例信息
4. QObject::tr()和QObject::trUtf8() 进行字符串翻译来实现国际化
5. QObject::setProperty()和QObject::property()通过名字来动态设置或者获取对象属性
6. QMetaObject::newInstance()构造该类的一个新实例
7. qobject_cast()函数对QObject类进行动态类型转换,正确返回一个非零的指针,例如：
      QObject *obj = new MyWidget；
      QWidget *widget = qobject_cast< QWidget* >(obj);
```

<font color=#FF0000 size=5> <p align="center">Qt对话框分类</p></font>

对话框分为模态对话框和非模态对话框。所谓模态对话框，就是会阻塞同一应用程序中其它窗口的输入，其中，Qt 有两种级别的模态对话框：应用程序级别的模态和窗口级别的模态，默认是应用程序级别的模态，应用程序级别的模态是指，当该种模态的对话框出现时，必须首先对对话框进行交互，直到关闭对话框，然后才能访问程序中其他的窗口，窗口级别的模态是指，该模态仅仅阻塞与对话框关联的窗口，但是依然允许与程序中其它窗口交互。窗口级别的模态尤其适用于多窗口模式
```
QDialog::exec() 实现应用程序级别的模态对话框
QDialog::open() 实现窗口级别的模态对话框
QDialog::show() 实现非模态对话框
```
Qt 的内置对话框大致分为以下几类：

类名 | 描述
---|---
QColorDialog | 颜色选择
QFileDialog  | 文件或者目录选择
QFontDialog  | 字体选择
QInputDialog | 允许用户输入一个值，并将其值返回
QMessageBox  | 模态对话框，用于显示信息、询问问题等
QPageSetupDialog | 为打印机提供纸张相关的选项
QPrintDialog | 打印机配置
QPrintPreviewDialog | 打印预览
QProgressDialog | 显示操作过程

<font color=#FF0000 size=5> <p align="center">Qt绘图系统</p></font>

整个绘图系统基于 QPainter，QPainterDevice 和 QPaintEngine 三个类
```
QPainter 用于进行绘制的实际操作；
QPaintDevice 是那些能够让 QPainter 进行绘制的二维空间的抽象层;其子类有 QWidget、QPixmap、QPicture、QImage 和 QPrinter QPaintEngine 提供供 QPainter 使用的用于在不同设备上绘制的统一的接口
```
绘图系统定义了两个关键属性：画刷和画笔

QBrush 定义了 QPainter 的填充模式，具有样式、颜色、渐变以及纹理等画刷属性
```
1. style() 定义了填充的样式，使用 Qt::BrushStyle 枚举，默认值是 Qt::NoBrush，也就是不进行任何填充
2. color() 定义了填充模式的颜色。这个颜色可以是预定义的颜色常量，也就是 Qt::GlobalColor，也可以是任意 QColor 对象
3. gradient() 定义了渐变填充。渐变可以由 QGradient 对象表示。提供了三种渐变：线性渐变（QLinearGradient）、辐射渐变（QRadialGradient）和角度渐变（QConicalGradient），它们都是 QGradient 的子类
4. texture() 定义了用于填充的纹理。当你调用 setTexture() 函数时，QBrush 会自动将 style() 设置为 Qt::TexturePattern
```
QPen 定义了用于 QPainter 应该怎样画线或者轮廓线。画笔具有样式、宽度、画刷、笔帽样式和连接样式等画笔属性
```
1. 画刷 style() 定义了线的样式
2. 画刷 brush() 用于填充画笔所绘制的线条
3. 笔帽样式 capStyle() 定义了使用 QPainter 绘制的线的末端
4. 连接样式 joinStyle() 则定义了两条线如何连接起来
5. 画笔宽度 width() 或 widthF() 定义了画笔的宽。宽度至少是 1 像素
```
光栅图形显示器上绘制非水平、非垂直的直线或多边形边界时，或多或少会呈现锯齿状外观。这是因为直线和多边形的边界是连续的，而光栅则是由离散的点组成。在光栅显示设备上表现直线、多边形等，必须在离散位置采样。由于采样不充分重建后造成的信息失真，就叫走样；用于减少或消除这种效果的技术，就称为反走样，用以防止“锯齿”现象的出现

基于像素的设备上(比如显示器)，坐标的默认单位是像素，在打印机上则是点(1/72 英寸)

Qt 提供了四种坐标变换：平移 translate，旋转 rotate，缩放 scale 和扭曲 shear

Graphics View Framework 有三个主要部分：
```
QGraphicsScene：提供了图形视图框架中的场景,是图形项 QGraphicsItem 对象的容器,一个场景分3层,
  图形项层、前景层、背景层,场景的绘制顺序是背景层-图形项层-前景层。拥有以下功能：
 --提供了用于管理大量图形项的快速接口
 --传播事件给每一个图形项
 --管理图形项的状态,例如选择和焦点处理
 --提供无变换的渲染功能,主要用于打印
QGraphicsItem：能够被添加到场景的图形项,拥有以下功能：
 --鼠标按下、移动、释放、双击、悬停、滚轮和右键菜单事件
 --键盘输入焦点和键盘事件
 --拖放事件
 --分组,使用 QGraphicsItemGroup 通过 parent-child 关系来实现
 --碰撞检测
QGraphicsView：提供了视图部件,使场景中的内容可视化,是一个可以滚动的区域,提供一个滚动条浏览大的场景。
  默认提供了一个 QWidget 作为视口部件,可以调用 QGraphicsView::setViewport() 来进行 OpenGL 渲染
  并设置 QGLWidget 作为视口
```

<font color=#FF0000 size=5> <p align="center">Qt文件读写与IO系统</p></font>

Qt 通过QIODevice类提供了对 I/O 设备的抽象，使其具有读写字节块的能力。下面是QT5 I/O 设备的类图：

![image](https://github.com/dwjlw1314/DWJ-PROJECT/raw/master/PictureSource/7.2.2.png)

途中所涉及的类及其用途简要说明如下：

类名 | 作用
---|---
QIODevice	 | 所有 I/O 设备类的父类，提供了字节块读写的通用操作以及基本接口
QBuffer    | 读写QByteArray
QFlie	     | 访问本地文件或者嵌入资源
QProcess   | 运行外部程序，处理进程间通讯
QUdpSocket | 传输 UDP 报文
QTcpSocket | TCP协议网络数据传输
QSslSocket | 使用 SSL/TLS 传输数据
QFileDevice| Qt5新增加的类，提供了有关文件操作的通用实现
QTemporaryFile  | 创建和访问本地文件系统的临时文件
QAbstractSocket | 所有套接字类的父类

open()函数中打开方式的区别：

枚举值 | 描述
---|---
QIODevice::NotOpen   | 未打开
QIODevice::ReadOnly  | 以只读方式打开
QIODevice::WriteOnly | 以只写方式打开
QIODevice::ReadWrite | 以读写方式打开
QIODevice::Append    | 以追加的方式打开，新增加的内容将被追加到文件末尾
QIODevice::Truncate  | 以重写的方式打开，在写入新的数据时会将游标设置在文件开头，并不是将文件内容清空
QIODevice::Text	     | 在读取时，将行结束符转换成 \n；在写入时，将行结束符转换成本地格式，例如 Win32 平台上是 \r\n
QIODevice::Unbuffered| 忽略缓存

方便起见，QTextStream 同 std::cout 一样提供了很多描述符，被称为 stream manipulators，这些描述符只是一些函数的简写

描述符 | 等价于
---|---
bin	            | setIntegerBase(2)
oct	            | setIntegerBase(8)
dec	            | setIntegerBase(10)
hex	            | setIntegerBase(16)
showbase	      | setNumberFlags(numberFlags() ↑ ShowBase)
forcesign	      | setNumberFlags(numberFlags() ↑ ForceSign)
forcepoint	    | setNumberFlags(numberFlags() ↑ ForcePoint)
noshowbase      | setNumberFlags(numberFlags() & ~ShowBase)
noforcesign     | setNumberFlags(numberFlags() & ~ForceSign)
noforcepoint  	| setNumberFlags(numberFlags() & ~ForcePoint)
uppercasebase	  | setNumberFlags(numberFlags() ↑ UppercaseBase)
uppercasedigits	| setNumberFlags(numberFlags() ↑ UppercaseDigits)
lowercasebase	  | setNumberFlags(numberFlags() & ~UppercaseBase)
lowercasedigits	| setNumberFlags(numberFlags() & ~UppercaseDigits)
fixed	          | setRealNumberNotation(FixedNotation)
scientific	    | setRealNumberNotation(ScientificNotation)
left	          | setFieldAlignment(AlignLeft)
right	          | setFieldAlignment(AlignRight)
center	        | setFieldAlignment(AlignCenter)
endl	          | operator<<(‘\n’) and flush()
flush	          | flush()
reset	          | reset()
ws	            | skipWhiteSpace()
bom             |	setGenerateByteOrderMark(true)

<font color=#FF0000 size=5> <p align="center">Qt容器类</p></font>

```
存储容器(containers)有时候也被称为集合(collections), 且是线程安全的
Qt 顺序存储容器：QList，QLinkedList，QVector，QStack 和 QQueue
Qt 关联容器：QMap，QMultiMap，QHash，QMultiHash 和 QSet。带有“Multi”字样的容器支持在一个键上面关联多个值
Hash容器提供了基于散列函数的更快的查找，而非Hash容器则是基于二分搜索的有序集合
QCache 和 QContiguousCache 提供了在有限缓存空间中的高效 hash 查找
```
Qt 各类容器总结:
```
QList<T>：这是至今为止提供的最通用的容器类。它将给定的类型 T 的对象以列表的形式进行存储，与一个整型的索引关联。QList 在内部使用数组实现，同时提供基于索引的快速访问。我们可以使用 QList::append() 和 QList::prepend() 在列表尾部或头部添加元素，也可以使用 QList::insert() 在中间插入。相比其它容器类，QList 专门为这种修改操作作了优化。QStringList 继承自 QList<QString>

QLinkedList<T>：类似于 QList，除了它是使用遍历器进行遍历，而不是基于整数索引的随机访问。对于在中部插入大量数据，它的性能要优于 QList。同时具有更好的遍历器语义（只要数据元素存在，QLinkedList 的遍历器就会指向一个合法元素，相比而言，当插入或删除数据时，QList 的遍历器就会指向一个非法值）

QVector<T>：用于在内存的连续区存储一系列给定类型的值。在头部或中间插入数据可能会非常慢，因为这会引起大量数据在内存中的移动

QStack<T>：这是 QVector 的子类，提供了后进先出（LIFO）语义。相比 QVector，它提供了额外的函数：push()，pop() 和 top()

QQueue<T>：这是 QList 的子类，提供了先进先出（FIFO）语义。相比 QList，它提供了额外的函数：enqueue()，dequeue() 和 head()

QSet<T>：提供单值的数学上面的集合，具有快速的查找性能

QMap<Key, T>：提供了字典数据结构（关联数组），将类型 T 的值同类型 Key 的键关联起来。通常，每个键与一个值关联。QMap 以键的顺序存储数据；如果顺序无关，QHash 提供了更好的性能

QMultiMap<Key, T>：这是 QMap 的子类，提供了多值映射：一个键可以与多个值关联

QHash<Key, T>：该类同 QMap 的接口几乎相同，但是提供了更快的查找。QHash 以字母顺序存储数据

QMultiHash<Key, T>：这是 QHash 的子类，提供了多值散列
````

算法复杂度描述，引入大写字母 O
```
常量时间：O(1)，如果一个函数的运行时间与容器中数据量无关，我们说这个函数是常量时间的。QLinkedList::insert() 就是常量时间
对数时间：O(log n)，如果一个函数的运行时间是容器数据量的对数关系，我们说这个函数是对数时间的。qBinaryFind() 就是对数时间
线性时间：O(n)，如果一个函数的运行时间是容器数据量的线性关系，与数量相关，这个函数是线性时间的。QVector::insert() 就是线性时间
线性对数时间：O(n log n)，线性对数时间要比线性时间慢，但是要比平方时间快
平方时间：O(n²)，平方时间与容器数据量的平方关系
```
Qt 顺序容器的算法复杂度：

容器类名 | 查找	| 插入 | 前方添加 | 后方追加
---|---|---|---|---
QLinkedList<T> | O(n) | O(1) | O(1) | O(1)
QList<T> | O(1) | O(n) | 统计O(1) | 统计 O(1)
QVector<T> | O(1) | O(n) | O(n) | 统计O(1)

Qt 关联容器的算法复杂度：

容器类名 | 查找键平均 | 查找键最坏 | 插入平均 | 插入最坏
---|---|---|---|--- |
QMap<Key, T> | O(log n) | O(log n) | O(log n) | O(log n)
QMultiMap<Key, T> | O(log n) | O(log n) | O(log n) | O(log n)
QHash<Key, T> | 统计O(1) | O(n) | O(1) | 统计O(n)
QSet<Key, T> | 统计O(1) | O(n) | O(1) | 统计O(n)

Java 风格的遍历器：一种只读访问，一种读写访问：

容器 | 只读遍历器 | 读写遍历器
---|---|---
QList<T>, QQueue<T> | QListIterator<T> | QMutableListIterator<T>
QLinkedList<T> | QLinkedListIterator<T> | QMutableLinkedListIterator<T>
QVector<T>, QStack<T> | QVectorIterator<T> | QMutableVectorIterator<T>
QSet<T> | QSetIterator<T> | QMutableSetIterator<T>
QMap<Key, T>, QMultiMap<Key, T> | QMapIterator<T> | QMutableMapIterator<T>
QHash<Key, T>, QMultiHash<Key, T> | QHashIterator<T> | QMutableHashIterator<T>

STL 风格的遍历器：一种只读访问，一种读写访问：

容器 | 只读遍历器 | 读写遍历器
---|---|---
QList<T>, QQueue<T> | QList<T>::const_iterator | QList<T>::iterator
QLinkedList<T> | QLinkedList<T>::const_iterator | QLinkedList<T>::iterator
QVector<T>, QStack<T> | QVector<T>::const_iterator | QVector<T>::iterator
QSet<T> | QSet<T>::const_iterator | QSet<T>::iterator
QMap<Key, T>, QMultiMap<Key, T> | QMap<Key, T>::const_iterator | QMap<Key, T>::iterator
QHash<Key, T>, QMultiHash<Key, T> | QHash<Key, T>::const_iterator | QHash<Key, T>::iterator

<font color=#FF0000 size=5> <p align="center">Qt Linguist</p></font>

Qt 翻译应用程序整个过程：
```
1.在 Qt 中编写代码时需要显示的字符串调用 tr() 函数
2.更改项目 *.pro 文件,在文件中指定生成的 .ts 文件. eg: 添加代码 TRANSLATIONS += myI8N_zh_CN.ts
3.在项目文件列表中 *.pro 文件右击,在弹出的菜单选择 “在此弹出命令提示” 在命令行中输入 lupdate *.pro
工具从 C++ 源代码中提取要翻译的文本，这时会生成一个 .ts 文件,它是 XML 格式,记录了字符串位置和是否已经被翻译等信息
4.启动 Qt Linguist 程序,单击界面左上角 open 图标,然后打开对应 .ts 文件,界面会自动显示源语言和目标语言，直到完成翻译工作
5.使用 lrelease 生成 .qm 文件.在命令行中输入 lrelease *.pro 即可完成; 也可以在 Linguist 程序中使用 文件->发布 功能
6.使用 .qm 文件，在 main.cpp 添加头文件 #include<QTranslator>; 然后在 QApplication a(argc,argv) 下添加代码
  QTranslator translator;
  translator.load("../*.qm");
  a.installTranslator(&translate);
```

<font color=#FF0000 size=5> <p align="center">定制Qt Assistant</p></font>

```
1.创建 HTML 格式的帮助文档
2.创建 Qt 帮助项目(Qt help project).qhp文件,该文件是 XML 格式,用来组织 HTML 文档,使其可以在 Qt Assistant 中使用
3.生成 Qt 压缩帮助(Qt compressed help).qch文件,由 *.qhp 文件生成的二进制格式文件
4.创建 Qt 帮助集合项目(Qt help collection project).qhcp文件,该文件是 XML 格式,用来生成二进制 *.qhc 文件
5.生成 Qt 帮助集合(Qt help collection).qhc文件,可以使 Qt Assistant 只显示一个应用程序帮助文档,也可以定制外观和功能
6.启动程序中的 Qt Assistant
```

<font color=#FF0000 size=5> <p align="center">创建应用程序插件过程</p></font>

```
@@创建一个插件步骤如下：
1.定义一个插件类,需要同时继承自 QObject 类和该插件所提供的功能对应的接口类
2.Q_PLUGIN_METADATA(IID "string") 宏在 Qt 的元对象系统中注册该接口类 (QT_VERSION >= 0x050000)
3.使用 Q_INTERFACES() 宏在 Qt 的元对象系统中注册该接口类
4.使用 Q_EXPORT_PLUGIN2() 宏导出该插件 (QT_VERSION < 0x050000)
5.使用合适的 .pro 文件构建该插件
@@一个应用程序通过插件进行扩展步骤:
1.定义一组接口(只有纯虚函数的抽象类)
2.使用 Q_DECLARE_INTERFACE() 宏在 Qt 的元对象系统中注册该接口类
3.在应用程序中使用 QPluginLoader 来加载插件
4.使用 qobject_cast() 来测试插件是否实现了给定的接口
```

<font color=#FF0000 size=5> <p align="center">Qt函数</p></font>

```
QString("[%1, %2]").arg(x, y); 语句将会使用 x 替换 %1，y 替换 %2，最终生成的 QString 为 [x, y];
QPainter::translate(x, y); 函数意思是将坐标系的原点设置到 (x, y) 点，原有坐标变成(-x，-y);
QTimer::singleShot(0, this, SLOT(f())); QTimer处理是将f()调用放到事件列表中，等到下一次事件循环开始时立刻调用槽函数
QVector<T>、QHash<Key, T>、QSet<T>、QString 和 QByteArray 3个成员函数含义：
capacity()：返回实际已经分配内存的元素数目(对于 QHash 和 QSet，则是散列表中桶的个数);
reserve(size)：为指定数目的元素显式地预分配内存，通过调用来减少内存占用;
squeeze()：释放那些不需要真实存储数据的内存空间,释放所有未使用的预分配空间;
QStyleFactory::keys(); 获取当前系统所支持的样式风格;
QLocale::system().name(); 获取本地系统的语言环境,返回QString类型的"语言_国家"格式,语言用小写字母,国家用大写字母
```
