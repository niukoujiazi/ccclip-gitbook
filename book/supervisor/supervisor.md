#  [supervisor](http://supervisord.org)
#  简介
Supervisor是一个客户端/服务器系统，允许其用户在类UNIX操作系统上控制许多进程。

> 注意：
> Supervisor是使用`python2`进行开发，运行依赖于python2；
> 故要求默认python版本必须为python2
> 使用以下命令可的到类似输出
> ```
> $ python version
> Python 2.7.5
> ```


> 建议切换到 `root` 用户操作
> ```
> $ sudo su 
> ```
#  安装
```
sudo pip install supervisor
```
#  配置
配置文件有两种,一般放在`/etc/supervisor/`目录下
- supervisor
    └ `supervisord`程序的启动配置文件
- *.conf
    └ `supervisor`通过该配置文件管理进程(启动/关闭)
#  supervisor.conf
```conf
; supervisor config file

[unix_http_server]
file=/var/run/supervisor.sock   ; (the path to the socket file)
chmod=0700                       ; sockef file mode (default 0700)

[supervisord]
logfile=/var/log/supervisor/supervisord.log ; (main log file;default $CWD/supervisord.log)
pidfile=/var/run/supervisord.pid ; (supervisord pidfile;default supervisord.pid)
childlogdir=/var/log/supervisor            ; ('AUTO' child log dir, default $TEMP)

; the below section must remain in the config file for RPC
; (supervisorctl/web interface) to work, additional interfaces may be
; added by defining them in separate rpcinterface: sections
[rpcinterface:supervisor]
supervisor.rpcinterface_factory = supervisor.rpcinterface:make_main_rpcinterface

[supervisorctl]
serverurl=unix:///var/run/supervisor.sock ; use a unix:// URL  for a unix socket

; The [include] section can just contain the "files" setting.  This
; setting can list multiple files (separated by whitespace or
; newlines).  It can also contain wildcards.  The filenames are
; interpreted as relative to this file.  Included files *cannot*
; include files themselves.

[include]
files = /etc/supervisor/conf.d/*.conf
```
也可以使用以下指令生成配置文件
```
echo_supervisord_conf > /etc/supervisord.conf
```
#  *.conf
```
[program:django]
directory=/root/gpsenv/gpsenv
command=/root/gpsenv/gpsenv/bin/python /root/gpsenv/manage.py runserver 0.0.0.0:8000
autostart=true
autorestart=true
startretries=10
redirect_stderr=true
stdout_logfile=/var/log/supervisor/django.log
environment=ASPNETCORE_ENVIRONMENT="Development"
```
#  其他文件创建
```
sudo mkdir supervisord
sudo touch supervisord/supervisord.log
sudo touch supervisord/stdout.log
```

#  启动supervisor

```
supervisord -c /etc/supervisor/supervisord.conf
```
具体路径请根据实际`supervisord.conf`文件所在位置进行配置

#  管理supervisor
```
- 1、查看状态
supervisorctl status
- 2、停止一个进程
supervisorctl stop <name>
- 3、启动一个进程
supervisorctl start <name>
- 4、重启一个进程
supervisorctl restart <name>
- 5、删除一个进程
supervisorctl remove <name>
- 6、更新一个进程的配置文件
supervisorctl update <name>
```
`<name>`为进程名，请按需替换




#  报错
#  使用`python3`启动supervisor
系统python环境不是python2的话，由于supervisor是python2的环境下运行，
如果版本不对将会导致启动错误。
可以通过修改`/etc/bin/supervisord`和`/etc/bin/supervisorctl`的启动版本来解决

先找到这两个文件，通过`find`命令查找

```
find / |grep supervisorctl

find / |grep supervisord
```
找打文件，一般都在`/etc/bin/`目录下可以看到这两个文件
直接进入修改

```
vim /etc/bin/supervisord
vim /etc/bin/supervisorctl
```

进入文件会看到第一行有

```
# !/usr/bin/python
```

将其修改为

```
# !/usr/bin/python2
```

便可使用python2环境启动supervisor

#  文件权限报错
保证几个文件夹的文件有足够的执行权限

```
chmod 777 /var/run/supervisord.sock
chmod 777 /var/log
```

如果报错

```
error: <class 'socket.error'>, [Errno 2] No such file or directory: file: /usr/lib64/python2.7/socket.py line: 224
```

则可以尝试将进程杀死再重新启动
```
ps aux | grep supervisord
# 根据pid杀死进程
kill pid
# 重新启动superviosr
supervisord -c /etc/supervisor/supervisord.conf
supervisorctl -c /etc/supervisor/supervisord.conf
```
#  supervisor.sock 文件报错
```
unix:///tmp/supervisor.sock no such file
# 或则
unix:///var/run/supervisor.sock no such file
```

说明没在对应位置找到`supevisor.sock`文件

解决方法是直接到指定目录下创建文件，修改文件权限，并取消过时的套接字

```
$ touch /var/run/supervisor.sock
$ chmod 777 /var/run/supervisor.sock
$ sudo unlink /var/run/supervisor.sock
```
#  问题一
```
Unlinking stale socket /var/run/supervisor/supervisor.sock
```
参考：
[supervisor 从安装到使用](https://www.jianshu.com/p/3658c963d28b)
[supervisor安装和自启动的一些问题](https://www.jianshu.com/p/b211a7bb528a)

解决方法：

```
$ sudo unlink /var/run/supervisor/supervisor.sock
```
#  问题二
```
Error: Another program is already listening on a port that one of our HTTP servers is configured to use.  Shut this program down first before starting supervisord.
For help, use /bin/supervisord -h
```
如字面意思，supervisor已经在运行了，但可能不是通过正确的方式启动，以至于
supervisor无法在shell中使用命令启动。

解决方法：

找出supervisor进程pid后`kill`后重新启动

```
$ ps -aux | grep supervisor
root      3306  0.0  3.4 223840 17212 ?        Ss   2月16   0:31 /usr/bin/python /bin/supervisord -c /etc/supervisor/supervisord.conf
root     19818  0.0  3.2 223548 16200 ?        Ss   12:13   0:00 /usr/bin/python /bin/supervisord -c /etc/supervisord.conf
clip     19870  0.0  0.1 112724   992 pts/0    S+   12:17   0:00 grep --color=auto supervisor
```
将找到的进程强行`kill`

```
$ sudo kill 3306
$ sudo kill 19818
$ sudo supervisord -c /etc/supervisor/supervisord.conf
```
#  问题三
```
django: ERROR (no such process)
```

修改`supervisord.conf`文件，找到

```
file=/var/run/supervisor/supervisor.sock   ; (the path to the socket file)
chmod=0700                 ; sockef file mode (default 0700)
```
将`700`改成`777`，再重启服务便可

#  可能会用到的操作
#  启动Virtualenv虚拟环境中的python程序
要启动`Virtualenv`中的python进程，需要对`supervisor`的配置文件做修改，
普通的配置文件是无法直接启动其程序的，需要在`command`命令上标注好启动
程序的环境。
其中`directory`文件目录指向`Virtualenv`的虚拟目录，

```
/root/gpsenv/gpsenv
# 第二个gpsenv为Virtualenv的环境目录
```

`commnd`运行命令处需要指向其运行目录的`/bin/`目录下，其后的运行命令直接
指向程序文件便可，

```
/root/gpesenv/gpsenv/bin/
# 第二个gpsenv为Virtualenv的环境目录
```

`Virtualenv`有时会使用同一个程序文件管理多个虚拟环境中的python程序，
这时只需要在程序文件中指定号启动文件便可，比如使用绝对路径

```
python /root/gpsenv/gpsenv/manage.py runserver 0.0.0.0:8000
# 其后的 `runserver 0.0.0.0:8000`指定了其运行的地址和端口
```

具体配置参考如下：

```
[program:django]
directory=/root/gpsenv/gpsenv
command=/root/gpsenv/gpsenv/bin/python /root/gpsenv/manage.py runserver 0.0.0.0:8000
autostart=true
autorestart=true
startretries=10
redirect_stderr=true
stdout_logfile=/var/log/supervisor/django.log
environment=ASPNETCORE_ENVIRONMENT="Development"
```
#  centos下开机启动supervisor
在`/usr/lib/systemd/system/`下新增/修改文件`supervisord.service`

```
#  supervisord service for systemd (CentOS 7.0+)
#  by ET-CS (https://github.com/ET-CS)
[Unit]
Description=Supervisor daemon

[Service]
Type=forking
ExecStart=/usr/bin/supervisord -c /etc/supervisord.conf
ExecStop=/usr/bin/supervisorctl $OPTIONS shutdown
ExecReload=/usr/bin/supervisorctl $OPTIONS reload
KillMode=process
Restart=on-failure
RestartSec=42s

[Install]
WantedBy=multi-user.target
```
其中`ExecStart=/usr/bin/supervisord -c /etc/supervisord.conf`为启动配置文件目录

设置开机启动

```
systemctl enable supervisord.service
```


#  配置环境变量
在进程配置文件中加入`environment`以配置环境变量，格式为

`;environment=KEY="value"     
; 这个是用来设置环境变量的，supervisord在linux中启动默认继承了linux的 环境变量，在这里可以设置supervisord进程特有的其他环境变量。
supervisord启动子进程时，子进程会拷贝父进程的内存空间内容。 所以设置的这些环境变量也会被子进程继承。
小例子：environment=name="haha",age="hehe"
 默认为不设置。。。非必须设置`

#  注意
如果正常写入配置重启之后还是读取不到环境变量，可能需要将`supervisord.conf`
配置文件做修改，找到

```
environment=DB_HOSTS_ENV=localhost,REDIS_HOSTS_ENV=localhost,ALLOWED_HOSTS_ENV=120.79.23.89       ; (key value pairs to add to environment)
```
请根据实际情况写入，具体逻辑可以理解为将环境变量写入`supervisord`的环
境，而进程的配置文件中环境变量的设置则声明了可以怎么获取，从`supervisord`
的进程中取得的环境变量
#  参考

[使用supervisor管理进程](http://liyangliang.me/posts/2015/06/using-supervisor/)
[CentOS7——supervisor安装配置实战](https://blog.csdn.net/qq_27754983/article/details/78782866)
[supervisor 永不挂掉的进程 安装以及使用](https://segmentfault.com/a/1190000015768529)
[centos安装supervisor 辅助队列任务](https://iluoy.com/articles/122)

#  闲言碎语
一开始误以为我的环境是python3,所以要安装python3版本的supervisor
但是

```
sudo pip3 install supervisor
```

总是安装不上去，google查了一下发现supervisor4.0适配python3的版本还处于
测试中，暂无法使用。

这就很困扰了，又查了半天，打算弄个环境来运行supervisor时，大佬又指点了我
以下，这个supervisor只是个管理进程的软件，与你使用的语言无关

一语点醒梦中人，我现在是python3和python2并存的状态，python3装不上去可以使用python2来安装使用啊


