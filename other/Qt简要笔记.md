Qt安装包主要包含以下几部分：
* Qt Library：也就是Qt的库，这是Qt的核心
* Qt Creator：官方开发的一款轻量级、跨平台的IDE，更换平台不需要重新学习
* Qt Designer：Qt程序的UI设计器。拖拽就可以开发简单的GUI程序，并且可以及时预览程序界面（无需编译）
* Qt Assistant：Qt帮助工具，包含了Qt教程、示例、类参考手册、模块介绍等，是Qt的官方资料，类似MSDN
* Qt Linguist：Qt语言包，是Qt的国际化工具，借助它可以很方便让程序支持多种语言，面向全球用户

<font color=#FF0000>在Windows下，GUI解决方案基于C++的有Qt、MFC、WTL、wxWidgets、DirectUI、Htmlayout，基于C#的有WinForm、WPF，基于Java的有AWT、Swing，基于Pascal的有Delphi，最新的aardio；有Web开发经验，可以基于Webkit或Chromium将网页转换为桌面程序，总起来说，Qt主要用于桌面程序开发和嵌入式开发</font> <br>

Qt是一个跨平台的框架。跨平台GUI通常有三种实现策略：

1. API 映射：API 映射是说，界面库使用同一套 API，将其映射到不同的底层平台上面。大体相当于将不同平台的 API 提取公共部分。比如说，将 Windows 平台上的按钮控件和 Mac OS 上的按钮组件都取名为 Button。当你使用 Button 时，如果在 Windows 平台上，则编译成按钮控件；如果在 Mac OS 上，则编译成按钮组件。优点是，所有组件都是原始平台自有的，外观和原生平台一致；缺点是，编写库代码的时候需要大量工作用于适配不同平台，并且，只能提取相同部分的 API。比如 Mac OS 的文本框自带拼写检测，但是 Windows 上面没有，则不能提供该功能。这种策略的典型代表是 wxWidgets。这也是一个标准的 C++ 库，和 Qt 一样庞大。它的语法看上去和 MFC 类似，有大量的宏

2. API 模拟：前面提到，API 映射会“缺失”不同平台的特定功能，而 API 模拟则是解决这一问题。不同平台的有差异 API，将使用工具库自己的代码用于模拟出来。按照前面的例子，Mac OS 上的文本框有拼写检测，但是 Windows 的没有。那么，工具库自己提供一个拼写检测算法，让 Windows 的文本框也有相同的功能。API 模拟的典型代表是 wine 是Linux上面的 Windows 模拟器。它将大部分 Win32 API 在Linux上面模拟了出来，让 Linux 可以通过 wine 运行 Windows 程序。由此可以看出，API 模拟最大优点是，应用程序无需重新编译，即可运行到特定平台上。一个例子是微软提供的 DirectX，这个开发库将屏蔽掉不同显卡硬件所提供的具体功能。使用这个库，你无需担心硬件之间的差异，如果有的显卡没有提供该种功能，SDK 会使用软件的方式加以实现

3. GUI 模拟：任何平台都提供了图形绘制函数，例如画点、画线、画面等。有些工具库利用这些基本函数，在不同绘制出自己的组件，这就是 GUI 模拟。GUI 模拟的工作量无疑是很大的，因为需要使用最基本的绘图函数将所有组件画出来；并且这种绘制很难保证和原生组件一模一样。但是，这一代价带来的优势是，可以很方便的修改组件的外观——只要修改组件绘制函数即可。很多跨平台的 GUI 库都是使用的这种策略，例如 Swing、Qt、gtk+（这是一个 C 语言的图形界面库。使用 C 语言很优雅地实现了面向对象程序设计。不过，这也同样带来了一个问题——使用大量的类型转换的宏来模拟多态，并且它的函数名一般都比较长，使用下划线分割单词，看上去和 Linux 如出一辙。gtk+ 并不是模拟的原生界面，而有它自己的风格，所以有时候就会和操作系统的界面格格不入）

Qt 5 模块分为 Essentials Modules 和 Add-on Modules 两部分。前者是基础模块，适用所有平台；后者是扩展模块，建立在基础模块之上 <br>
Qt 基础模块分有以下几个：
* Qt Core，提供核心的非 GUI 功能，所有模块都需要这个。这个模块的类包括了动画框架、定时器、各个容器类、时间日期类、事件、IO、JSON、插件机制、智能指针、图形（矩形、路径等）、线程、XML 等。所有这些类都可以通过 <QtCore> 头文件引入
* Qt Gui，提供 GUI 程序的基本功能，包括与窗口系统的集成、事件处理、OpenGL 和 OpenGL ES 集成、2D 图像、字体、拖放等。这些类一般由 Qt 用户界面类内部使用，当然也可以用于访问底层的 OpenGL ES 图像 API。Qt Gui 模块提供的是所有图形用户界面程序都需要的通用功能
* Qt Multimedia，提供视频、音频、收音机以及摄像头等功能。这些类可以通过 <QtMultimedia> 引入，而且需要在 pro 文件中添加 QT += multimedia
* Qt Network，提供跨平台的网络功能。这些类可以通过 <QtNetwork> 引入，而且需要在 pro 文件中添加 QT += network
* Qt Qml，提供供脚本语言 QML 使用的 C++ API。这些类可以通过 <QtQml> 引入，而且需要在 pro 文件中添加 QT += qml
* Qt Quick，允许在 Qt/C++ 程序中嵌入 Qt Quick (一种基于 Qt 的高度动画的用户界面，适合于移动平台开发)，这些类可以通过 <QtQuick> 引入，而且需要在 pro 文件中添加 QT += quick
* Qt SQL，允许使用 SQL 访问数据库。这些类可以通过 <QtSql> 引入，而且需要在 pro 文件中添加 QT += sql
* Qt Test，提供 Qt 程序的单元测试功能。这些类可以通过 <QtTest> 引入，而且需要在 pro 文件中添加 QT += testlib
* Qt Webkit，基于 WebKit2 的实现以及一套全新的 QML API（顺便说一下，Qt 4.8 附带的是 QtWebkit 2.2）
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

Qt 4 也分成若干模块，但是这些模块与 Qt 5 有些许多不同。下面是 Qt 4 的模块：
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

下面是 Qt 4 的一些工具模块：
* QQtDesigner，用于扩展 Qt Designer
* QQtUiTools，用于在自己的引用程序中处理 Qt Designer 生成的 form 文件
* QQtHelp，联机帮助
* QQtTest，单元测试

下面是专门供 Windows 平台的模块：
* QAxContainer，用于访问 ActiveX 控件
* QAxServer，用于编写 ActiveX 服务器

下面是专门供 Unix 平台的模块：
* QtDBus，使用 D-Bus 提供进程间交互

<font color=#FF0000 size=4> <p align="center">qtcreator支持的编译器以及对应调试器</p></font>
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

1. 自定义类只有继承了 QObject 类，才具有信号槽的能力。凡是继承 QObject 类，都应该在第一行代码写上 Q_OBJECT宏。这个宏的展开将为我们的类提供信号槽机制、国际化机制以及 Qt 提供的不基于 C++ RTTI 的反射能力，这个宏将由 moc（可以将其理解为一种预处理器，比 C++ 预处理器更早执行）做特殊处理，moc 只会读取标记了 Q_OBJECT 的<头文件内容>，生成以 moc_ 为前缀的文件

2. 类中 signals 块所列出的一个个的函数名就是该类的信号，返回值是 void（因为无法获得信号的返回值，所以也就无需返回任何值），参数是传递到slot的数据。信号作为函数名，不需要在 cpp 函数中添加任何实现（Qt 程序能够使用普通的 make 进行编译。没有实现的函数名怎么会通过编译？这里 moc 会帮我们实现信号函数所需要的函数体。emit 是 Qt 对 C++ 的扩展，是一个关键字（其实也是一个宏）

3. 自定义信号槽需要注意的5个事项：1.发送者和接收者都需要是 QObject 的子类（当然，槽函数是全局函数、Lambda 表达式等无需接收者的时候除外）; 2.使用 signals 标记信号函数，信号是一个函数声明，返回 void，不需要实现函数代码; 3.槽函数是普通的成员函数，作为成员函数，会受到访问控制符的影响; 4.使用 QObject::connect() 函数连接信号和槽; 5.emit 在恰当的位置发送信号

<font color=#FF0000 size=4> <p align="center">Qt调试关联工具</p></font>
qt内存泄露检查：
```
Linux ,Mac OS X ： Valgrind
Windows： Visual Leak Detector for Visual C++ 2008-2015 (VLD, Open-source)
```

<font color=#FF0000 size=4> <p align="center">Qt moc</p></font>
实际在使用标准 C++ 编译器编译 Qt 源程序之前，Qt 扩展了标准 C++，先使用一个叫做 moc（Meta Object Compiler，元对象编译器）的工具，先对 Qt 源代码进行一次预处理（注意，这个预处理与标准 C++ 的预处理有所不同。Qt 的 moc 预处理发生在标准 C++ 预处理器工作之前，并且 Qt 的 moc 预处理不是递归的），生成标准 C++ 源代码，然后再使用标准 C++ 编译器进行编译。信号函数是不需要编写实现代码的，moc为信号函数这样的语法进行了处理，这样就可以通过标准 C++ 的编译了。类通过继承 QObject 类，可以很方便地获得这些特性。当然，这些特性都是由 moc 帮助实现的。moc 其实实现的是一个叫做元对象系统（meta-object system）的机制，正如上面所说，这是一个标准 C++ 的扩展，更适合于进行 GUI 编程。虽然利用模板可以达到类似的效果，但是 Qt 没有选择使用模板。按照 Qt 官方的说法，模板虽然是内置语言特性，但是其语法实在是复杂，并且由于 GUI 是动态的，利用静态的模板机制有时候很难处理。而使用 moc 生成代码更为灵活，虽然效率有些降低（一个信号槽的调用大约相当于四个模板函数调用），现代计算机上这点性能损耗实在是可以忽略的

Qt 使用 moc，为标准 C++ 增加了一些特性：
* 信号槽机制，用于解决对象之间的通讯，可以认为是 Qt 最明显的特性之一
* 可查询，并且可设计的对象属性
* 强大的事件机制以及事件过滤器
* 基于上下文的字符串翻译机制（国际化），也就是 tr() 函数
* 复杂的定时器实现，用于在事件驱动的 GUI 中嵌入能够精确控制的任务集成
* 层次化的可查询的对象树，提供一种自然的方式管理对象关系
* 智能指针（QPointer），在对象析构之后自动设为 0，防止野指针
* 能够跨越库边界的动态转换机制

<font color=#FF0000 size=4> <p align="center">Qt对象树</p></font>
QObjects 是以对象树的形式组织起来的。当创建一个 QObject 对象时，会看到 QObject 的构造函数接收一个 QObject 指针作为参数，这个参数就是 parent，也就是父对象指针。这相当于，在创建 QObject 对象时，可以提供一个其父对象，创建的这个 QObject 对象会自动添加到其父对象的 children() 列表。当父对象析构的时候，这个列表中的所有对象也会被析构
```
标准 C++ （ISO/IEC 14882:2003）要求，局部对象的析构顺序应该按照其创建顺序的相反过程
```

<font color=#FF0000 size=4> <p align="center">Qt对话框分类</p></font>
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
