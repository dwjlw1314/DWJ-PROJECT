* /etc/profile  <br>
#此文件为系统的每个用户设置环境信息,当用户第一次登录时,该文件被执行.并从/etc/profile.d目录的配置文件中搜集shell的设置

* /etc/bashrc
#为每一个运行bash shell的用户执行此文件.当bash shell被打开时,该文件被读取

* ~/.bash_profile
#每个用户都可使用该文件输入专用于自己的shell信息,当用户登录时,该文件仅执行一次.默认他设置一些环境变量,执行用户的.bashrc文件

* ~/.bashrc
#该文件包含专用于你的bash shell的bash信息,当登录时以及每次打开新的shell时,该文件被读取

* ~/.bash_logout
#当每次退出系统(退出bash shell)时,执行该文件

* ~/.bash_history
#是bash shell的历史记录文件，里面记录了你在bash shell中输入的所有命令，可通过HISSIZE环境变量设置在历史记录文件里保存记录的条数

#/etc/profile 中设定的变量(全局)的可以作用于任何用户,而~/.bashrc等中设定的变量(局部)只能继承/etc/profile中的变量,他们是"父子"关系

~/.bash_profile 是交互式login方式进入bash运行的; ~/.bashrc 是交互式non-login方式进入bash运行的;所以通常前者会调用后者

设置生效：可以重启生效，也可以使用命令：
>[root@dwj etc]# source /etc/profile

<font color=#FF0000 size=5>用户环境配</font>

.bash_profile、.bashrc、和.bash_logout

* 上面这三个文件是bash shell的用户环境配置文件，位于用户的主目录下。其中.bash_profile是最重要的一个配置文件，它在用户每次登录系统时被读取，里面的所有命令都会被bash执行。.profile(由Bourne Shell和Korn Shell使用)和.login(由C Shell使用)两个文件是.bash_profile的同义词，目的是为了兼其他Shell。在Debian中使用.profile文件代 替.bash_profile文件

* .bashrc文件会在bash shell调用另一个bash shell时读取，也就是在shell中再键入bash命令启动一个新shell时就会去读该文件。这样可有效分离登录和子shell所需的环境。但一般来说都会在.bash_profile里调用.bashrc脚本以便统一配置用户环境

* .bash_logout在退出shell时被读取。所以我们可把一些清理工作的命令放到这文件中。

在/etc目录的bashrc和profile是系统级（全局）的配置文件，当在用户主目录下找不到.bash_profile 和.bashrc时，就会读取这两个文件

<font color=#FF0000 size=5>选项</font>

bash shell中的选项可控制shell的行为和功能，我们可以通过shopt命令来设置。使用set命令也可以，但它已被shopt替代，但为了向下兼容，set命令还是可以使用的。使用不带参数的shopt命令可以列出当前shell中只能由shopt设置的选项，用shopt -o可列出可由set命令设置的选项

下面是部分可由set命令基本的选项，默认是关闭的
```
emacs           进入emacs编辑模式
vi              进入vi编辑模式
ignoreeof       不允许单独使用Ctrl_D退出的用法，要使用exit。与IGNOREEOF=10等价
noclobber       不允许重定向覆盖已存在文件
noglob          不允许扩展文件名通配符
nounset         使用未定义的变量时给出错误
```
下面是部分只能由shopt命令设置的选项
```
cdspell          自动改正cd命令参数中的小错误
hostcomplete     以@开头时，按tab键可进行主机名的自动完成
dotgblob         以点开始的文件名被包含在路径名扩展中
mailwarn         显示邮件警告信息
```
shopt命令的选项如下：
```
-p               显示可设置选项及当前取值
-s               设置每一选项为on
-u               设置每一选项为off
-q               不输出信息
-o               输出由set操作的选项
```

<font color=#FF0000 size=5>登录Linux时要执行文件的过程如下</font>

在刚登录Linux时，首先启动/etc/profile文件，然后再启动用户目录下的~/.bash_profile|~/.bash_login|~/.profile文件中的其中一个(根据不同的linux操作系统的不同，命名不一样)

执行的顺序为：~/.bash_profile、 ~/.bash_login、 ~/.profile; 如果 ~/.bash_profile文件存在的话，一般还会执行 ~/.bashrc文件

因为在 ~/.bash_profile文件中一般会有下面的代码：
```
if [ -f ~/.bashrc ]; then
    . ~/.bashrc
fi

```
~/.bashrc中，一般还会有以下代码：
```
if [ -f /etc/bashrc ]; then
    . /etc/bashrc
fi
```
所以~/.bashrc会调用/etc/bashrc文件。最后，在退出shell时，还会执行~/.bash_logout文件

执行顺序为：/etc/profile -> (~/.bash_profile | ~/.bash_login | ~/.profile) -> ~/.bashrc -> /etc/bashrc -> ~/.bash_logout

/etc/profile和/etc/environment等各种环境变量设置文件的用处:

先将export LANG=zh_CN加入/etc/profile ,退出系统重新登录，登录提示显示英文

将/etc/profile中的export LANG=zh_CN删除，将LNAG=zh_CN加入/etc/environment，退出系统重新登录，登录提示显示中文

/etc/environment是设置整个系统的环境，而/etc/profile是设置所有用户的环境，前者与登录用户无关，后者与登录用户有关

系统应用程序的执行与用户环境可以是无关的，但与系统环境是相关的，所以当你登录时，你看到的提示信息，象日期、时间信息的显示格式与系统环境的LANG是相关的，缺省LANG=en_US，如果系统环境LANG=zh_CN，则提示信息是中文的，否则是英文的

用户环境建立的过程中总是先执行/etc/profile然后在读取/etc/environment

对于用户的SHELL初始化而言是先执行/etc/profile,再读取文件/etc/environment.对整个系统而言执行顺序如下：

/etc/enviroment --> /etc/profile --> $HOME/.profile -->$HOME/.env (如果存在)

登陆系统时shell读取的顺序应该是
/etc/profile ->/etc/enviroment -->$HOME/.profile -->$HOME/.env

如果同一个变量在用户环境(/etc/profile)和系统环境(/etc/environment)有不同的值那应该是以用户环境为准了
