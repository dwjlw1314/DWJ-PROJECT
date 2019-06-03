windows10系统中，在vmware workstation中安装虚拟机的时候遇到提示“WMware Workstation与Hyper-v不兼容。请先从系统中移除Hyper-v角色，然后再运行VMware Workstation”。

遇到这个提示可以把Hyper-v功能关闭，然后再打开vmware

![image](https://github.com/dwjlw1314/DWJ-PROJECT/raw/master/PictureSource/2.5.1.png)

方法/步骤
```
打开“控制面板”
在控制面板中，选择查看方式为大图标或者小图标，然后点击"程序和功能"
在打开的窗口中，点击右边菜单的“启用或关闭windows功能”
找到hyper-v的选项
```
![image](https://github.com/dwjlw1314/DWJ-PROJECT/raw/master/PictureSource/2.5.2.png)

取消勾选，然后点击确定，使设置生效

![image](https://github.com/dwjlw1314/DWJ-PROJECT/raw/master/PictureSource/2.5.3.png)

系统会配置hyper-v，配置成功之后提示重启电脑，需要重启电脑使设置生效
