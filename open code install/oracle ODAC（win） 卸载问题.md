1.  使用自带的uninstall工具，路径 app/client_1/oui/bin/setup.exe

2. 手动删除过程
```
删除注册表，包括HKEY_LOCAL_MACHINE\SOFTWARE和HKEY_LOCAL_MACHINE\SYSTEM\CurrentControlSet\Services下的oracle目录和服务
删除c:\program file\oracle目录
重启系统，删除oracle文件所在的目录报错,提示oci.dll文件无法删除
```
分析dll文件是在系统启动时被自动加载的，oracle的服务已经在注册表中删除了，那么就不是服务调用的了
想了下原来是环境变量的原因，将PATH中的oracle的bin目录删重启后，oci.dll就可以删除了
```
1.删除path中oracle的环境变量                                --不删除会出现oci.dll无法删除
2.删除注册表
  a.HKEY_LOCAL_MACHINE\SOFTWARE                            --oracle软件的目录
  b.HKEY_LOCAL_MACHINE\SYSTEM\CurrentControlSet\Services   --oracle对应的系统服务
3.删除c:\program file\oracle目录                            --安装oracle时产生的日志，安装成功就可以删除
4.重启系统后删除oracle文件主目录                             --卸载完成
```

2. 数据导出文件格式
```
CSV  格式(逗号分隔值)
TSV  格式(制表符分隔值)
HTML 格式
XML  格式
SQL  格式生成一个SQL文件，内容包含针对表或视图的查询语句
```
