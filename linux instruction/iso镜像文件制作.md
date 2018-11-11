制作iso镜像的方法：把/dev/cdrom目录制作为镜像，名字为/root/dwj.iso
```
方法1：[root@dwj ~]# dd if=/dev/cdrom of=/root/dwj.iso
方法2：[root@dwj ~]# cat /dev/cdrom > /root/dwj.iso
方法3：[root@dwj ~]# mkisofs -r -o dwj.iso /dev/cdrom
方法4：[root@dwj ~]# cp -r /home/user dwj.iso
```
>[root@dwj ~]# mkisofs  -r -J -V library32 -0 /opt/library32.iso /opt/library32

该命令是将/opt/library32目录下的所有文件和文件夹制作成光盘镜像文件library32.iso，光盘卷标是library32

参数名 | 描述
--- | ---
-a或--all | mkisofs通常不处理备份文件。使用此参数可以把备份文件加到镜像文件中
-A<应用程序ID>或-appid<应用程序ID>  | 指定光盘的应用程序ID
-abstract<摘要文件> |  指定摘要文件的文件名
-b<开机映像文件>或-eltorito-boot<开机映像文件>  | 指定在制作可开机光盘时所需的开机映像文件
-biblio<ISBN文件> |  指定ISBN文件的文件名，ISBN文件位于光盘根目录下，记录光盘的ISBN
-c<开机文件名称>  | 制作可开机光盘时，mkisofs会将开机映像文件中的全部内容作成一个文件
-C<盘区编号，盘区编号>  | 将许多节区合成一个映像文件时，必须使用此参数
-copyright<版权信息文件>  | 指定版权信息文件的文件名
-d或-omit-period |  省略文件后的句号
-D或-disable-deep-relocation | ISO9660最多只能处理8层的目录，使用-D参数可关闭此功能
-f或-follow-links  | 忽略符号连接
-h  | 显示帮助
-hide<目录或文件名>  | 使指定的目录或文件在ISO 9660或Rock RidgeExtensions的系统中隐藏
-hide-joliet<目录或文件名> |  使指定的目录或文件在Joliet系统中隐藏
-J或-joliet |  使用Joliet格式的目录与文件名称
-l或-full-iso9660-filenames  | 使用ISO 9660 32字符长度的文件名
-L或-allow-leading-dots |  允许文件名的第一个字符为句号
-log-file<记录文件>  | 在执行过程中若有错误信息，预设会显示在屏幕上
-m<目录或文件名>或-exclude<目录或文件名> |  指定的目录或文件名将不会房入映像文件中
-M<映像文件>或-prev-session<映像文件> |  与指定的映像文件合并
-N或-omit-version-number  | 省略ISO 9660文件中的版本信息
-o<映像文件>或-output<映像文件>  | 指定映像文件的名称
-p<数据处理人>或-preparer<数据处理人>  | 记录光盘的数据处理人
-print-size  | 显示预估的文件系统大小
-quiet  | 执行时不显示任何信息
-r或-rational-rock  | 使用Rock Ridge Extensions，并开放全部文件的读取权限
-R或-rock  | 使用Rock Ridge Extensions
-sysid<系统ID>  | 指定光盘的系统ID
-T或-translation-table |  建立文件名的转换表，适用于不支持Rock Ridge Extensions的系统上
-v或-verbose |  执行时显示详细的信息
-V<光盘ID>或-volid<光盘ID>  | 指定光盘的卷册集ID
-volset-size<光盘总数> |  指定卷册集所包含的光盘张数
-volset-seqno<卷册序号>  | 指定光盘片在卷册集中的编号
-x<目录>  | 指定的目录将不会放入映像文件中
-z  | 建立通透性压缩文件的SUSP记录，此记录目前只在Alpha机器上的Linux有效
