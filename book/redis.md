
#  Redis
#  install redis
#  Ubuntu
  

```
sudo apt-get update
sudo apt-get install redis-server
```

启动redis

```
redis-server
```

查看redis是否有好好启动，输入

```
redis-cli
```

如果redis正常启动将会打开redis的shell，在终端显示

```
redis 127.0.0.1:6379>
```

`127.0.0.1`是本机IP，`6379`时redis的默认服务器端口，可以输入命令

```
redis 127.0.0.1:6379> ping
PONG
```

显示`PONG`代表redis安装好了
# 修改Redis数据目录
参考：[Redis 数据存储位置 导出数据](https://blog.csdn.net/CSDNones/article/details/50387256)

[redis修改data目录之后，RDB和AOF写入失败](https://www.jianshu.com/p/2ad2ae9f8446)

打开配置文件查看

```
$ vim /etc/redis/redis.conf
```
找到

```
#  The working directory.
# 
#  The DB will be written inside this directory, with the filename specified
#  above using the 'dbfilename' configuration directive.
# 
#  The Append Only File will also be created inside this directory.
# 
#  Note that you must specify a directory here, not a file name.
dir /var/lib/redis
```

#  出现的错误
#  启动redis报错
参考：[Ubuntu下启动 Redis时， 提示 "Can't open the log file: Permission denied failed"](https://blog.csdn.net/whq19890827/article/details/72842435)

```
$ service redis-server start
Starting redis-server: 
*** FATAL CONFIG FILE ERROR ***
Reading the configuration file, at line 171
>>> 'logfile /var/log/redis/redis-server.log'
Can't open the log file: Permission denied
failed
```

原因：`redis-server.log`执行权限不够

解决方法：

```
$ sudo chmod 777 /var/log/redis/redis-server.log
```
#  redis-cli 报错
```
$ redis-cli
Could not connect to Redis at 127.0.0.1:6379: Connection refused
```
解决方法：修改配置文件

```
$ find / | grep redis.conf
/etc/redis/redis.conf
$ vim /etc/redis/redis.conf
```
找到

```
bind 127.0.0.1 ::1
```
这行，将其改为：

```
bind 127.0.0.1
```
保存后退出，并重启服务

```
$ redis-server /etc/redis/redis.conf
$ redis-cli
```