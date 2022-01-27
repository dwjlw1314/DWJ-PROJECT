Gitlab分为社区版Gitlab CE 和企业版Gitlab EE

Gitlab服务主要构成:
```
nginx:静态web服务器
gitlab-shell：用于处理Git命令和修改authorized keys列表
gitlab-workhorse:轻量级的反向代理服务器
logrotate：日志文件管理工具
postgresql：数据库
redis：缓存数据库
sidekiq：用于在后台执行队列任务（异步执行）
unicorn：An HTTP server for Rack applications
GitLab Rails应用是托管在这个服务器上面的
```

Gitlab官方安装文档
https://about.gitlab.com/installation/#centos-7

GitLab国内源下载地址
https://mirrors.tuna.tsinghua.edu.cn/gitlab-ce/yum/el7/

Gitlab安装
>[root@dwj ~]# wget https://mirrors.tuna.tsinghua.edu.cn/gitlab-ce/yum/el7/gitlab-ce-11.2.3-ce.0.el7.x86_64.rpm

利用yum安装本地指定的rpm包，好处是自动解决依赖问题
>[root@dwj ~]# yum localinstall gitlab-ce-11.2.3-ce.0.el7.x86_64.rpm -y

初始化GitLab，只需要执行一次
>[root@dwj ~]# gitlab-ctl reconfigure

查看gitlab启动状态，有多个进程启动
>[root@dwj ~]# gitlab-ctl status

查看GitLab版本号
>[root@dwj ~]# cat /opt/gitlab/embedded/service/gitlab-rails/VERSION

修改Gitlab配置文件/etc/gitlab/gitlab.rb
```
external_url 'http://10.3.9.152:8666'
nginx['listen_port'] = 8666
unicorn['port'] = 8667
gitlab_workhorse['auth_backend'] = "http://localhost:8667"
```

重新配置GitLab
>[root@dwj ~]# gitlab-ctl reconfigure

重启动GitLab
>[root@dwj ~]# gitlab-ctl restart

浏览器访问
http://10.3.9.152:8666
