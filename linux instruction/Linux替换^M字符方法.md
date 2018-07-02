替换^M字符，在Linux下使用vim来查看一些在Windows下创建的文本文件，行尾会有一些 "^M" 字符，以下方法可以处理

1.使用dos2unix命令，可以去掉行尾的^M，一般的分发版本中都带有这个小工具，使用起来很方便:
>[root@dwj /]# dos2unix myfile.txt

2.使用vim的替换功能，前提是vim编辑界面有'^M'字符。启动vim，进入命令模式，输入以下任意命令即可:
```
:%s/^M$//g   #去掉行尾的^M
:%s/^M//g    #去掉所有的^M
:%s/^M/\r/g  #将^M替换成回车
:%s/^M/[ctrl-v]+[enter]/g   #将^M替换成回车
```

3.使用sed命令。和vim的用法相似：
>[root@dwj /]# sed -i 's/^M/\n/g' myfile.txt

4.使用tr命令重定向到newfile
>[root@dwj /]# cat filename |tr -d '^M' > newfile

5.在vim的/etc/vimrc文件中最后添加 set fileformats=unix,dos 就可以了

注意：这里的"^M"要使用"CTRL-V CTRL-M"生成，而不是直接键入"^M"
```
vim版本大于7.1，用dos显示和保存，如下语句：
:e ++ff=dos
vim版本小于等于7.1，用dos格式显示和保存，如下语句：
:set ff=dos
vim版本小于等于7.1，用unix格式显示和保存，如下语句：
:set ff=unix
替换^M可以使用上面5种方法
```
