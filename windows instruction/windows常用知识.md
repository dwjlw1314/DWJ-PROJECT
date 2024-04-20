1.判断dll是32位还是64位的简单方法:

notepad++打开exe或dll文件，只需要在第二段中找到PE两个字母，在其后的若是d，则证明该程序是64位；若是L，则证明是32位
```
y爉?my爉?my爉Richy爉        PE  d? PT        ?"
```

2.永久设置 cmd 终端字体颜色
```
C:\Users\dwj> regedit
1. 查找 关键字 DefaultColor
2. 修改对应字体颜色编号即可
3. 默认路径：HKEY_LOCAL_MACHINE\SOFTWARE\Microsoft\Command Processor
```
