sed是一个文件处理工具，本身是一个管道命令，主要是以行为单位进行处理，sed的用法:

sed命令行格式为： sed [-nefri] 'command' 输入文本
```
常用选项：
  -n: 使用安静(silent)模式。所有来自STDIN的资料一般都会被列出到萤幕上。加上-n参数后，则只有经过特殊处理的那一行(或者动作)才会被列出来
  -e: 直接在指令列模式上进行 sed 的动作编辑
  -f: 直接将 sed 的动作写在一个档案内， -f filename 则可以执行 filename 内的sed 动作
  -r: sed 的动作支援的是延伸型正规表示法的语法。(预设是基础正规表示法语法)
  -i: 直接修改读取的档案内容，而不是由屏幕输出

常用命令：
  a∶ 新增，a的后面可以接字串，而这些字串会在新的一行出现(目前的下一行)
  c∶ 取代，c的后面可以接字串，这些字串可以取代 n1,n2之间的行
  d∶ 删除，因为是删除啊，所以d后面通常不接任何咚咚
  i∶ 插入，i的后面可以接字串，而这些字串会在新的一行出现(目前的上一行)
  p∶ 列印，亦即将某个选择的资料印出。通常p会与参数sed -n一起运作
  s∶ 取代，可以直接进行取代的工作哩！通常这个s的动作可以搭配正规表示法！例如 1,20s/old/new/g 就是
```
举例：(假设我们有一文件名为ab)

1.删除某行

    [root@localhost ruby] # sed '1d' ab          #删除第一行?
    [root@localhost ruby] # sed '$d' ab          #删除最后一行
    [root@localhost ruby] # sed '1,2d' ab        #删除第一行到第二行
    [root@localhost ruby] # sed '2,$d' ab        #删除第二行到最后一行

2.显示某行

    [root@localhost ruby] # sed -n '1p' ab       #显示第一行
    [root@localhost ruby] # sed -n '$p' ab       #显示最后一行
    [root@localhost ruby] # sed -n '1,2p' ab     #显示第一行到第二行
    [root@localhost ruby] # sed -n '2,$p' ab     #显示第二行到最后一行

3.使用模式进行查询

    [root@localhost ruby] # sed -n '/ruby/p' ab  #查询包括关键字ruby所在所有行
    [root@localhost ruby] # sed -n '/\$/p' ab    #查询包括关键字$所在所有行，使用反斜线\屏蔽特殊含义

4.增加一行或多行字符串

    #第一行后增加字符串"drink tea"
    [root@localhost dwj]# cat ab
      Hello!
      dwj is me,welcome to my blog.
      end
    [root@localhost dwj] # sed '1a drink tea' ab
      Hello!
      drink tea
      dwj is me,welcome to my blog.
      end

    #第一行到第三行后增加字符串"drink tea"
    [root@localhost dwj] # sed '1,3a drink tea' ab
      Hello!
      drink tea
      dwj is me,welcome to my blog.
      drink tea
      end
      drink tea

    #第一行后增加多行，使用换行符\n
    [root@localhost dwj] # sed '1a drink tea\nor coffee' ab
      Hello!
      drink tea
      or coffee
      dwj is me,welcome to my blog.
      end

5.代替一行或多行

    #第一行代替为Hi
    [root@localhost dwj] # sed '1c Hi' ab
      Hi
      dwj is me,welcome to my blog.
      end
    #第一行到第二行代替为Hi
    [root@localhost dwj] # sed '1,2c Hi' ab
      Hi
      end

6.替换一行中的某部分  <br>
格式：sed 's/要替换的字符串/新的字符串/g（要替换的字符串可以用正则表达式）

    [root@localhost dwj] # sed -n '/dwj/p' ab | sed 's/dwj/bird/g #把dwj替换为bird
    [root@localhost dwj] # sed -n '/dwj/p' ab | sed 's/dwj//g     #删除dwj

7.插入

    #在文件ab中最后一行直接输入"bye"
    [root@localhost dwj] # sed -i '$a bye' ab
    [root@localhost dwj]# cat ab
      Hello!
      dwj is me,welcome to my blog.
      end
      bye

8.删除匹配行  <br>
sed -i '/匹配字符串/d' filename（注：若匹配字符串是变量，则需要“”，而不是‘’）

替换匹配行中的某个字符串  <br>
sed -i '/匹配字符串/s/替换源字符串/替换目标字符串/g' filename

sed -i "s# PATH=.*#&:`pwd`#" /etc/profile       //追加数据
