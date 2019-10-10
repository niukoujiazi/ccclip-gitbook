Harbor
====

# 前言

一时踩坑一时爽，一直踩坑电脑会坏掉

# 非常感谢帮助我的大佬们，记下大佬的群号:`774607973`
# 登录时hostname错误
```
Error response from daemon: Get https://server.top/v2/: 
Get https://server.com:80/service/token?account=admin&client_id=docker&offline_token=true&service=harbor-registry: 
http: server gave HTTP response to HTTPS client
```
```

前面`.top`后面变成`.com`，`hostname`填错了。解决方法：

打开`Harbor`的配置文件`harbor.cfg`，找到

```
# The IP address or hostname to access admin UI and registry service.
# DO NOT use localhost or 127.0.0.1, because Harbor needs to be accessed by external clients.
# DO NOT comment out this line, modify the value of "hostname" directly, or the installation will fail.
# hostname = reg.mydomain.com
hostname = server.com
```

将`.com`改为正确的。之后，尤其重要的个步骤是`为Harbor生成配置文件`

```
$ ./prepare
```
之后再重启`docker-compose`

# nginx不停重启
启动`Harbor`之后，执行`docker-compose ps`发现`nginx`在不停的重启。

由于我的日志驱动有问题，就从日志文件`/var/log/harbor/proxy.log`找到部分
问题。

```
Apr  1 19:04:52 172.25.0.1 proxy[22610]: nginx: [emerg] 
PEM_read_bio_X509_AUX("/etc/nginx/cert/server.top.crt") failed 
(SSL: error:25066067:DSO support routine
s:DLFCN_LOAD:could not load the shared library:filename
(libz.so): libz.so: cannot open shared object file: 
No such file or directory error:25070067:DSO support 
routines:DSO_load:could not load the shared library 
error:0906D06C:PEM routines:PEM_read_bio:no start line)
```
```
没找到ssl的`.crt`文件导致，于是我将文件复制到了指定路径`/etc/nginx/cert/`

但是，没能解决问题，后来大佬又指点了我，我的`.crt`文件格式不对，才想起
由于我从阿里云上申请的免费证书后缀是`.pem`我就使用`opssl`转换成了`.crt`，
造成格式不对。其实不用转，直接将`.pem`的后缀改成`.crt`就行了
