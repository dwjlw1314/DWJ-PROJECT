/dev/mapper/VolGroup-lv_root: unexpected inconsistency; run fsck manually

 解决方法：

1：输入 root 密码；<br>
2：输入 fsck -c<br>
3: 输入 Yes<br>
3: 输入 fsck -t ext4 /dev/sda1<br>

//可以取消这一步<br>
--4: 输入fsck -y /dev/sda1; (# df -h ; # mount | grep ''on /''; # fsck -y /dev/your_partition)

5：reboot  重启
