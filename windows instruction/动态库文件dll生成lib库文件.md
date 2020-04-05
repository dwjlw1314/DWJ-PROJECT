1.编写bat批处理文件nmake.bat，做如下设置：
```
setpath=%path%;"C:\ProgramData\Microsoft\Visual Studio 2019\Visual Studio Tools\VC\bin"
cmd.exe /kvcvars32.bat
```

2.启动nmake.bat，进入dll库所在目录：
> cd /d D:\eclipse_project\OpenBLAS-0.3.8\build-vs2019\bin

3.使用dumpbin导出函数列表到某个文件中：
>  dumpbin -exports libopenblas.dll > lib.txt

4.生成的lib.txt中包含了dll文件的导出函数信息，如下:
```
Microsoft(R) COFF Binary File Dumper Version 6.00.8447
Copyright(C) Microsoft Corp 1992-1998. All rights reserved.


Dump of file libopenblas.dll

File Type: DLL

  Section contains thefollowing exports for libopenblas.dll

           0 characteristics
    53A0ED46 time date stamp Wed Jun 1809:37:10 2014
        0.00 version
           1 ordinal base
        7417 number of functions
        7417 number of names

    ordinal hint RVA      name

          3   0 000012C0 CAXPY
       3084   1 00248C30 CBBCSD
       2070   2 00158A30 CBDSQR
       …
       5017 1CF7 004303C0 zupmtr
       5018 1CF8 004303C0 zupmtr_

  Summary

        1000 .CRT
      10E000 .bss
      …
      80D000 .text
        1000 .tls
```

5.将lib.txt中“ordinal hintRVA name”一行之前以及下面“Summary”之后的内容全部删除，然后使用UtriEdit等带有列编辑功能的文本编辑器打开lib.txt，切换到列模式：

![image](https://github.com/dwjlw1314/DWJ-PROJECT/raw/master/PictureSource/5.5.1.jpg)

6.在最前面一列加入@（选择第一列，然后输入@即可）

7.将name列移动到@前面；

8.删除hint和RVA两列；

9.在文件的前面添加两行，最后改造成下面这样：

![image](https://github.com/dwjlw1314/DWJ-PROJECT/raw/master/PictureSource/5.5.2.jpg)

10.将上面改造完成后的文件重命名为libopenblas.def，然后使用lib命令，生成lib文件，如下：

```
对于32位的机器，使用：
> lib /machine:i386 /def:libopenblas.def

对于64位的机器，使用：
> lib /machine:X64 /def:libopenblas.def
```

11.至此，我们需要的lib库文件生成了

====================================

MinGW使用def文件生成MinGW可以使用的后缀为.a的lib库的步骤：
> libtool –d libopenblas.def –l libopenblas.a

命令执行完毕后，生成的libopenblas.a文件就是MinGW的lib库文件

另外Mingw实际上提供了一个工具用于生成def文件
> pexports libopenblas.dll > libopenblas.def

如果Mingw提示没有pexports命令，我们只需要使用Mingw-get安装一下即可：
> mingw-get install pexports
