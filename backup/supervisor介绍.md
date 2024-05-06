supervisor是一个用于控制程序后台启动的脚本，拿python写的挺亲切的。
但是目前看到的教程和现在的版本不太一致，所以来个简单的教程。

首先安装：sudo apt-get install supervisor
其次配置文件：/etc/supervisor/supervisord.conf
这里我们一般什么都不需要修改，因为在这个文件最后一行包含了：
[include]
files = /etc/supervisor/conf.d/*.conf
我们只需要写子配置文件即可

在这个目录的conf.d下创建*.conf结尾的文件，内容如下
```
[program:contract]
directory=/home/foxing/contract_collector
command=python3 -u contract_collector.py
autostart=true
autorestart=false
startsecs=1
user = foxing
stderr_logfile=/tmp/contract_collector_err.log
stdout_logfile=/tmp/contract_stdout.log
redirect_stderr = true
stdout_logfile_maxbytes = 20000000
stdout_logfile_backups = 3
```
大概介绍贴一下：
```
#项目名
[program:blog]
#脚本目录
directory=/opt/bin
#脚本执行命令
command=/usr/bin/python /opt/bin/test.py
#supervisor启动的时候是否随着同时启动，默认True
autostart=true
#当程序exit的时候，这个program不会自动重启,默认unexpected，设置子进程挂掉后自动重启的情况，有三个选项，false,unexpected和true。如果为false的时候，无论什么情况下，都不会被重新启动，如果为unexpected，只有当进程的退出码不在下面的exitcodes里面定义的
autorestart=false
#这个选项是子进程启动多少秒之后，此时状态如果是running，则我们认为启动成功了。默认值为1
startsecs=1
#脚本运行的用户身份 
user = test
#日志输出 
stderr_logfile=/tmp/blog_stderr.log 
stdout_logfile=/tmp/blog_stdout.log 
#把stderr重定向到stdout，默认 false
redirect_stderr = true
#stdout日志文件大小，默认 50MB,这里改了是int，不能带M单位。
stdout_logfile_maxbytes = 20000000
#stdout日志文件备份数
stdout_logfile_backups = 20
```
之后使用supervisorctl update 可以重现加载配置文件
敲supervisorctl status可以看到我们进程组的状态
```
root@debian:/home/foxing/contract_collector# supervisorctl 
contract                         RUNNING   pid 22889, uptime 0:00:17
```