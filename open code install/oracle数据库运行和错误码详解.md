<font color=#FF0000 size=5> <p align="center">ORA-00845</p></font>
```
command: SQL> startup
descript: MEMORY_TARGET not supported on this

oracle的官方解释是：

Starting with Oracle Database 11g, the Automatic Memory Management feature requires more shared memory (/dev/shm)and file descriptors. The size of the shared memory should be at least the greater of MEMORY_MAX_TARGET and MEMORY_TARGET for each Oracle instance on the computer. If MEMORY_MAX_TARGET or MEMORY_TARGET is set to a non zero value, and an incorrect size is assigned to the shared memory, it will result in an ORA-00845 error at startup.

MEMORY_MAX_TARGET的设置不能超过/dev/shm的大小，在oracle11g中新增的内存自动管理的参数MEMORY_TARGET,它能自动调整SGA和PGA，这个特性需要用到/dev/shm共享文件系统，而且要求/dev/shm必须大于MEMORY_TARGET，如果/dev/shm比MEMORY_TARGET小就会报错

解决方案：
[root@dwj ~]# vim /etc/fstab      #修改2个地方
1.swap后面修改成defaults,size=11G
2.tmpfs后面修改成defaults,size=11G

修改完后，需要重新挂载一下，才能生效：
[root@dwj ~]# mount -o remount,size=11G /dev/shm
```

<font color=#FF0000 size=5> <p align="center">EXP-00008</p></font>
```
[root@dwj ~]# exp c##antman/ant@orcl file=~/backup.dmp log=~/backup.log full=y
descript: ORACLE error 1013 encountered

解决方案：
#以DBA身份连接数据库执行两条语句
[oracle@diwj ~]$ sqlplus / as sysdba
SQL> grant execute on SYS.DBMS_DEFER_IMPORT_INTERNAL to c##antman;
SQL> grant execute on SYS.DBMS_EXPORT_EXTENSION to c##antman;
```
