环境信息

SSH Secure Shell Client所在的操作系统：Windows7

Linux服务器版本：
>[root@node1 ~]# cat /etc/redhat-release  <br>
Red Hat Enterprise Linux Server release 6.4 (Santiago)

免密码登录的原理:SSH服务中有公钥和私钥的概念(public Key是指公钥，Private Key是指私钥)

认证的过程如下：

Public Key对数据进行加密而且只能用于加密，Private Key也只能对所匹配的Public Key加密过的信息进行解密

把Windows上面生成的Public Key放到Linux服务器上用户家目录下面的.ssh目录中，并添加公钥内容到该目录下面的authorized_keys文件

如果开始从Windows(客户端)上面通过ssh方式远程Linux(服务器)时，此时客户端软件就会向服务器发出请求，请求用密匙进行安全验证。服务器收到请求之后，先在该服务器上的主目录下寻找公匙，然后把它和发送过来的公匙进行比较。如果两个密匙一致，服务器就用公匙加密“质询”并把它发送给客户端软件。客户端软件收到“质询”之后就可以用私匙解密再把它发送给服务器，此时因为密钥能匹配上，所以可以直接登录到Linux服务器

先找到SSH SecureShell的安装路径，然后进入dos界面,切换到ssh目录
>C:\Users\hsql> cd /d "C:\Program Files (x86)\SSH Communications Security\SSH Secure Shell"

生成密钥，要求免密码登录服务器，采用默认值；这里使用rsa的密钥，默认是2048bit(位)，同样可以使用dsa方式的密钥
>C:\Program Files (x86)\SSH Communications Security\SSH Secure Shell> ssh-keygen2 -t rsa

windows环境生成的密钥对位于以下目录
```
Private key saved to C:/Users/hsql/Application Data/SSH/UserKeys/id_rsa_2048_a
Public key saved to  C:/Users/hsql/Application Data/SSH/UserKeys/id_rsa_2048_a.pub
```

接着用密码的方式登录到Linux服务器的用户下，然后在用户的家目录下面创建.ssh目录(如果已经存在就不需要创建)

然后我们将Windows的UserKeys目录下的id_rsa_2048_a.pub文件上传到Linux服务器相应用户的.ssh目录下面

如果是Linux之间信任关系，就直接将公钥的内容添加到authorized_keys文件中即可(失败尝试使用下面导入方式)
>[root@node1 .ssh]# cat id_rsa_2048_a.pub >> authorized_keys

但是如果公钥是在Windows上面生成的，Linux的Openssh不识别，所以需要进行转换后再追加到authorized_keys中
>[root@node1 .ssh]# ssh-keygen -i -f id_rsa_2048_a.pub >> authorized_keys

重新使用SSH Secure Shell客户端，然后重新登录并在登录认证状态栏中选择Public Key方式，这样就可以免密码登录了
