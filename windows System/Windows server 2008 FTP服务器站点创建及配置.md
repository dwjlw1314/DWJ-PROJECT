1.创建专属于FTP的用户  计算机-管理-本地用户和组-用户-新建用户 –创建FTP用户

2.安装IIS组件(以 WindowsServer2008 R2为例)

控制面板--程序和功能--打开或关闭Windows服务进入图1界面--添加角色服务进入图2界面勾选FTP选项,完成FTP站点服务添加

![image](https://github.com/dwjlw1314/DWJ-PROJECT/raw/master/PictureSource/2.3.1.jpg)
![image](https://github.com/dwjlw1314/DWJ-PROJECT/raw/master/PictureSource/2.3.2.jpg)

3.新建文件FTP文件夹(名字自定义),上传的文件存在该文件中.

4.点击IIS左边主页，需要设置的有：服务器证书（SSL），FTP  SSL设置，FTP身份验证，FTP授权规则

![image](https://github.com/dwjlw1314/DWJ-PROJECT/raw/master/PictureSource/2.3.3.jpg)

服务器证书（SSL）:双击进入----创建自签名证书-取个名字---OK

![image](https://github.com/dwjlw1314/DWJ-PROJECT/raw/master/PictureSource/2.3.4.png)

 FTP  SSL设置：双击进入---选择上一步创建的证书-点击：允许SSL 链接—ok

![image](https://github.com/dwjlw1314/DWJ-PROJECT/raw/master/PictureSource/2.3.5.jpg)

 FTP身份验证：进入--------启用“基本身份验证”-----OK

![image](https://github.com/dwjlw1314/DWJ-PROJECT/raw/master/PictureSource/2.3.6.jpg)

 FTP授权规则：进入----添加允许规则—选择指定用户（添写最初创建的FTP用户）---权限读写----OK

![image](https://github.com/dwjlw1314/DWJ-PROJECT/raw/master/PictureSource/2.3.7.jpg)

 5.点击网站—右键---添加FTP站点---输入一个名称---选择路径(最初为FTP创建的文件夹)

![image](https://github.com/dwjlw1314/DWJ-PROJECT/raw/master/PictureSource/2.3.8.jpg)

6.绑定和SSL设置：

下一步：IP地址指定本机地址，端口默认为21

SSL:勾选“允许”，选择之前创建的SSL证书---OK

![image](https://github.com/dwjlw1314/DWJ-PROJECT/raw/master/PictureSource/2.3.9.jpg)

7.身份验证和授权信息：

身份验证选基本----授权---指定的用户(添加最初创建的)

![image](https://github.com/dwjlw1314/DWJ-PROJECT/raw/master/PictureSource/2.3.10.jpg)

完成。重启IIS，通过IP进行访问验证。如果无法访问，就可能是防火墙的问题
