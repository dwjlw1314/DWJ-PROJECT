<font color=#FF0000 size=4><p align="center">git log 查看所有提交历史</p></font>

查看全部提交历史并展示每次修改的内容
>[root@dwj Desktop] git log -p

查看最近n次提交历史
>[root@dwj Desktop] git log -n

查看最近n次提交历史并展示修改的内容
>[root@dwj Desktop] git log -p -n

列出修改的文件以及每个文件中修改了多少行，是被移除还是添加，并展示摘要内容
>[root@dwj Desktop] git log --stat

查看提交历史，并显示摘要内容（统计并展示修改了多少内容而不显示具体哪些文件做出了修改）
>[root@dwj Desktop] git log --shortstat

用来指定使用不同于默认格式的方式展示提交历史，value取值有：oneline , short , full , fuller等
>[root@dwj Desktop] git log --pretty=value

```
oneline  该参数会把提交历史的commit描述以及校验和显示在同一行，并且省略默认格式下的其他内容
full     该参数输出与默认的格式相比少了Data日期的描述，但是增加了commit提交人信息
short    该参数输出比默认的格式少了Data日期的描述
fuller   该参数全部输出
fromat   该参数后跟指定时间格式
```

仅在默认格式后面展示已经修改的文件
>[root@dwj Desktop] git log --name-only

显示每次提交后新增、修改、删除的文件清单
>[root@dwj Desktop] git log --name-status

仅显示SHA-1的前几个字符，而非全部字符commit id
>[root@dwj Desktop] git log --abbrev-commit

以相对当前的时间展示提交历史
>[root@dwj Desktop] git log --relative-date

在展示提交历史前面加入简单的ASCII图形，区分每次提交历史
>[root@dwj Desktop] git log --graph

筛选选项-S，可以列出那些添加或移除了某些关键字的提交
>[root@dwj Desktop] git log -Sfunction_name

显示指定name的作者相关提交，注意：这里的用户名，是初始化git时传入的name
>[root@dwj Desktop] git log --author=name

显示指定name的提交者相关提交
>[root@dwj Desktop] git log --committer=name

只显示某些文件或者目录的修改，需要用两个短划线--隔开选项和后面限定的路径名
>[root@dwj Desktop] git log -- ./dir

仅显示指定时间之后的提交
>[root@dwj Desktop] git log --since=time/--after=time

仅显示指定时间之前的提交
>[root@dwj Desktop] git log --until=time/--before=time

仅显示含指定关键字的提交
>[root@dwj Desktop] git log --grep=key

>[root@dwj Desktop] git log --pretty=format:"%h - %an, %ar : %s" --since=”2008-10-01” --before=”2008-11-01” <br>

列出了常用选项format的格式占位符写法及其代表的意义：

选项 | 说明
---|---
%H  |提交对象commit的完整哈希字串
%h  |提交对象的简短哈希字串
%T  |树对象tree的完整哈希字串
%t  |树对象的简短哈希字串
%P  |父对象parent的完整哈希字串
%p  |父对象的简短哈希字串
%an |作者author的名字
%ae |作者的电子邮件地址
%ad |作者修订日期,可以用-date=选项定制格式
%ar |作者修订日期，按多久以前的方式显示
%cn |提交者(committer)的名字
%ce |提交者的电子邮件地址
%cd |提交日期
%cr |提交日期，按多久以前的方式显示
%s  |提交说明

<font color=#FF0000 size=4><p align="center">git rebase 合并多个</p></font>

合并1-3条，有两个方法:
1.从HEAD版本开始往过去数3个版本
>[root@dwj Desktop] git rebase -i HEAD~3

2.指名要合并的版本之前的版本号,3a4226b这个版本是不参与合并的，只是一个坐标
>[root@dwj Desktop] git rebase -i 3a4226b

执行了rebase命令之后，会弹出一个窗口，头几行如下：
```
pick commitver '注释*********'
pick commitver '注释*********'
pick commitver '注释*********'
```
将pick改为squash或者s,之后保存并关闭文本编辑窗口即可。改完之后文本内容如下:
```
pick commitver '注释*********'
s commitver '注释*********'
s commitver '注释*********'
```
然后保存退出，Git会压缩提交历史，修改以后要记得敲下面的命令：
```
[root@dwj Desktop] git add .
[root@dwj Desktop] git rebase --continue
如果你想放弃这次压缩的话，执行以下命令：
[root@dwj Desktop] git rebase --abort
```
如果没有冲突，或者冲突已经解决，则会出现如下的编辑窗口：
```
  This is a combination of 4 commits.
  The first commit’s message is:
  注释......
  The 2nd commit’s message is:
  注释......
  The 3rd commit’s message is:
  注释......
  Please enter the commit message for your changes. Lines starting  with ‘’ will be ignored, and an empty message aborts the commit.
```
输入wq保存并退出, 再次输入git log查看commit历史信息，你会发现这两个commit已经合并了

<font color=#FF0000 size=4><p align="center">git config 设置控制Git外观和行为的配置变量</p></font>

这些变量存储在三个不同的位置：
1. /etc/gitconfig 文件: 包含系统上每一个用户及他们仓库的通用配置。从此文件读写配置变量使用--system选项
2. ~/.gitconfig 文件：只针对当前用户。可以传递--global选项让读写此文件
3. .git/config 文件：当前使用仓库的 Git 目录中的 config 文件，针对该仓库

列出所有Git可配置的字段
>[root@dwj /] git config --list

列出指定配置的字段
>[root@dwj /] git config user.name

配置默认文本编辑器
>[root@dwj /] git config --global core.editor notepad++

设置你的用户名称与邮件地址
>[root@dwj /] git config --global user.name "dwjlw1314" <br>
>[root@dwj /] git config --global user.email dwjlw1314@qq.com

可以通过git config来轻松地为每一个完整的git命令设置一个别名
>[root@dwj /] git config --global alias.last 'log -1 HEAD'
#这样，可以用last轻松地看到最后一次提交
>[root@dwj /] git last

<font color=#FF0000 size=4><p align="center">git status 工作区和暂存区文件状态简览</p></font>

>[root@dwj /] git status -s   状态报告输出如下：

```
 M README
MM Rakefile
A lib/git.rb
M lib/simplegit.rb
?? LICENSE.txt
```
新添加的未跟踪文件前面有 ?? 标记，新添加到暂存区中的文件前面有 A 标记，修改过的文件前面有 M 标记。你可能注意到了 M 有两个可以出现的位置，出现在右边的 M 表示该文件被修改了但是还没放入暂存区，出现在靠左边的 M 表示该文件被修改了并放入了暂存区。 例如，上面的状态报告显示： README 文件在工作区被修改了但是还没有将修改后的文件放入暂存区,lib/simplegit.rb 文件被修改了并将修改后的文件放入了暂存区。 而Rakefile 在工作区被修改并提交到暂存区后又在工作区中被修改了，所以在暂存区和工作区都有该文件被修改了的记录

<font color=#FF0000 size=4><p align="center">git commit 提交暂存区文件</p></font>

命令会将暂存区中的文件尝试重新提交,快照会保持不变，而修改的只是提交信息和commit id
>[root@dwj /] git commit --amend

<font color=#FF0000 size=4><p align="center">git remote  远程仓库的使用</p></font>

查看某一个远程仓库的更多信息
>[root@dwj /] git remote show [remote-name]

显示需要读写远程仓库使用的 Git 保存的简写与其对应的 URL
>[root@dwj /] git remote -v

添加一个新的远程 Git 仓库，同时指定一个你可以轻松引用的简写 shortname
>[root@dwj /] git remote add shortname url

拉取远程的 shortname 仓库中有但你没有的信息
>[root@dwj /] git fetch [remote-name]

将 master 分支推送到 origin 服务器远程仓库
>[root@dwj /] git push origin master

远程仓库的移除与重命名
>[root@dwj /] git remote rename old-shortname new-shortname <br>
>[root@dwj /] git remote rm shortname

<font color=#FF0000 size=4><p align="center">git tag 标签的使用</p></font> <br>
Git主要有两种类型的标签：轻量标签 lightweight 与附注标签 annotated，默认push不会将标签上传到远程仓库服务器上

基本和特定模式查找标签
>[root@dwj /] git tag  <br>
>[root@dwj /] git tag -l 'V1.1*'

创建附注标签使用 -a 参数，-m 选项指定了一条将会存储在标签中的信息
>[root@dwj /] git tag -a v1.4 -m 'my version 1.4'  <br>
>[root@dwj /] git show v1.4  显示打标签者的信息、日期时间、附注信息，具体的提交信息

创建轻量标签本质上是将提交校验和存储到一个文件中没有保存任何其他信息
>[root@dwj /] git tag v1.5-dwj

本地删除tag和删除远程仓库的tag的方法
>[root@dwj /] git tag -d v1.5-dwj <br>
>[root@dwj /] git push origin :refs/tags/v1.5-dwj

对已经提交的commit id=9fceb02打标签
>[root@dwj /] git tag -a v1.2 9fceb02

显式地推送标签到origin远程服务器上
>[root@dwj /] git push origin [tagname]

一次性推送所有不在origin远程仓库服务器上的标签
>[root@dwj /] git push origin --tags

检出标签，Git 中你并不能真的检出一个标签，需要在特定的标签上创建一个新分支，如果再进行了一次提交那么 branchname 分支就会和
tagname 标签就不同了，需要注意
>[root@dwj /] git checkout -b [branchname] [tagname]

<font color=#FF0000 size=4><p align="center">git branch 分支操作</p></font> <br>
Git在当前 commit 对象上新建一个分支指针,该命令仅仅是建立了一个新的分支，但不会自动切换到这个分支中去，所以我们依然还在 master 分支里工作
>[root@dwj /] git branch lgl_branch

切换到lgl_branch分支，可以执行 git checkout 命令，这样 HEAD 就指向了 lgl_branch 分支
>[root@dwj /] git checkout lgl_branch <br>
>[root@dwj /] git checkout -b lgl_branch  这相当于执行了上面两条命令
