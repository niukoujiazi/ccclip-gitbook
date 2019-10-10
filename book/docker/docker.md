DOCKER
====
# 参考：
[Docker](https://yeasy.gitbooks.io/docker_practice/)

[DockerHub](https://hub.docker.com/)

[跟陈sir一起玩转docker卷 一](https://www.jianshu.com/p/3ee587517702)

[docker学习5--docker数据卷(volume)](https://blog.csdn.net/dream_broken/article/details/52314993)

[docker中宿主机与容器（container）互相拷贝传递文件的方法](https://blog.csdn.net/dongdong9223/article/details/71425077)

# 删除私有仓库镜像
# 参考
[Docker私有仓库搭建及镜像删除](http://www.louisvv.com/archives/1130.html)

[docker私有仓库删除image](http://blog.51cto.com/302876016/1966816)

[docker 查询或获取私有仓库(registry)中的镜像](http://www.voidcn.com/article/p-toomzlub-brm.html)

[Docker Registry之删除镜像、垃圾回收](https://blog.csdn.net/u010884123/article/details/56838644)

# 服务器配置
服务器上的`register`容器需要修改配置文件，允许其删除镜像。

配置文件名位`config.yml`

因为我们要修改这个配置文件并以此文件来允许`register`容器，所以我们物理机
上必然就没有这个文件，可以先运行一次`register`容器，然后从其中拷出这个
配置文件做修改

```
$ docker run -d -p 5000:5000 -v /opt/data/registry:/var/lib/registry registry
561d1e831f8376684d7d82d3616c6d90135c785cdbfff3fd296a8f4a67be1d70
$ docker cp 561:/etc/docker/registry/config.yml ./
$ vim config.yml
```
打开文件后在其中找到

```
storage:
  cache:
    blobdescriptor: inmemory
  filesystem:
    rootdirectory: /var/lib/registry
```
加入一句配置：

```
storage:
  delete:
      enabled: true
  cache:
    blobdescriptor: inmemory
  filesystem:
    rootdirectory: /var/lib/registry
```
再启动容器便可删除仓库镜像

```
$ docker run -d -v /home/jyt/config.yml:/etc/docker/registry/config.yml \
> -v /opt/data/registry:/var/lib/registry \
> -p 5000:5000 --restart=always --name registry registry
6f425cd3aadb256e25cc37bad0d49b3d9fa614901dd0e6ddc7c121806b9b1b72
```
# 删除镜像
# 查询仓库现有镜像

```
$ curl server:port/v2/_catalog
{"repositories":["alpine","ubuntu"]}
```
# 查询指定镜像在仓库中的现有版本

```
$ curl server:port/v2/ubuntu/tags/list
{"name":"ubuntu","tags":["18.04"]}
```
# 查询指定镜像版本的sha码

```
$ curl --header "Accept: application/vnd.docker.distribution.manifest.v2+json"  -I -XGET server:port/v2/ubuntu/manifests/18.04
HTTP/1.1 200 OK
Content-Length: 1150
Content-Type: application/vnd.docker.distribution.manifest.v2+json
Docker-Content-Digest: sha256:be159ff0e12a38fd2208022484bee14412680727ec992680b66cdead1ba76d19
Docker-Distribution-Api-Version: registry/2.0
Etag: "sha256:be159ff0e12a38fd2208022484bee14412680727ec992680b66cdead1ba76d19"
X-Content-Type-Options: nosniff
Date: Thu, 21 Feb 2019 05:20:54 GMT
```
# 根据`sha`码删除镜像

```
$ curl -XDELETE server:port/v2/ubuntu/manifests/sha256:be159ff0e12a38fd2208022484bee14412680727ec992680b66cdead1ba76d19
```
执行这个操作没有返回结果的，linux的哲学“没有消息，就是好消息”

但这个时候仅仅只是删除了一个“标签”，再次查询仓库，会发现镜像名字任然
在，而查询版本时就找不到指定版本

```
$ curl server:port/v2/ubuntu/tags/list
{"name":"ubuntu","tags":null}
```
# 释放空间
想要释放空间只能进服务器的`register`容器内操作了（以下操作在服务器上进行）

```
$ docker container ls
6f425cd3aadb        registry            "/entrypoint.sh /etc…"   About an hour ago   Up About an hour    0.0.0.0:5000->5000/tcp   registry
$ docker exec -it 6f4 /bin/sh

/ #  cd /var/lib/registry
/ #  du -sh 
33.8M
```
这里可以看到，存储镜像的文件夹大小，33.8M。

这里有个疑问，上传的镜像大小和仓库存储镜像的文件夹大小差太多，存储镜像的文件小很多

```
registry garbage-collect /etc/docker/registry/config.yml
```
以上，便完成了网上所谓的镜像删除。

再次尝试之后发现，将一个镜像删除之后再次`push`上去之后就无法`pull`下来了

```
$ docker pull server:port/ubuntu:18.04
Error response from daemon: manifest for 139.199.9.116:5000/ubuntu:18.04 not found
```
# 错误
# 启动register错误
参考：[docker端口映射或启动容器时报错Error response from daemon: driver failed programming external connectivity on endpoint quirky_allen](https://www.cnblogs.com/hailun1987/p/7518306.html)

```
docker: Error response from daemon: driver failed programming external 
connectivity on endpoint romantic_sanderson (12565e44485be0ef9d5c282
e325d76dc4e8a1e38f71f2764fbf83bbcb18e27be):  (iptables failed: iptables
--wait -t nat -A DOCKER -p tcp -d 0/0 --dport 5000 -j DNAT --to-destination 
172.17.0.2:5000 ! -i docker0: iptables: No chain/target/match by that name.
 (exit status 1)).
```
解决方法：重启docker

```
$ systemctl restart docker
```
# 端口占用
在启动容器时报错

```
Error response from daemon: driver failed programming external connectivity on endpoint redis (6c89629a558b666e5fba1c3ad27d6de3c4b341ee1005dae9ab3ec7b797cfdf1d): Error starting userland proxy: listen tcp 0.0.0.0:6379: bind: address already in use
Error: failed to start containers: redis
```
原因很明显，端口被使用了，解决方法就是释放该端口

```
$ lsof -i:6379
```
很尴尬的事情发生了，`lsof`没有输出任何内容，这代表没有任何进程正在使用
这个端口，但是这个端口又切切实实的被占用着。对于`lsof`的作用可能需要重新
认知一下了，但现在不是搞这个的时候。

好在我们知道这个端口默认是`redis`的服务在使用，于是

```
$ sudo service redis-server stop
```
停止`redis`服务间接实现释放该端口

# 启动容器命令参考
# postgresql
```
$ docker run -dit -p 5432:5432 --name postgresql \
> --mount source=postgresql,target=/var/lib/postgresql \
> -u postgres jyt:v11\
> /usr/lib/postgresql/9.3/bin/postgres \
> -D /var/lib/postgresql/9.3/main \
> -c config_file=/etc/postgresql/9.3/main/postgresql.conf 
```
参考:[使用docker部署PostgreSQL数据库](https://segmentfault.com/a/1190000004518306)
# redis
```
$ docker run -dit --network jyt-net \
> --mount source=redis,target=/var/lib/redis \
> -p 6379:6379 \
> --name redis jyt:v11 \
> /usr/bin/redis-server
```
参考：[Docker 安装 Redis](http://www.runoob.com/docker/docker-install-redis.html)

redis-server --appendonly yes : 在容器执行redis-server启动命令，并打开
redis持久化配置

# 在docker中部署postgresql启动错误
# 1、
```
web_1    | django.db.utils.OperationalError: FATAL:  no pg_hba.conf entry for host "172.18.0.4", user "postgres", database "postgres", SSL on
web_1    | FATAL:  no pg_hba.conf entry for host "172.18.0.4", user "postgres", database "postgres", SSL off
web_1    | 
db-com   | 2019-02-26 12:11:43.925 CST [19] postgres@postgres FATAL:  no pg_hba.conf entry for host "172.18.0.4", user "postgres", database "p
ostgres", SSL on
db-com   | 2019-02-26 12:11:43.928 CST [20] postgres@postgres FATAL:  no pg_hba.conf entry for host "172.18.0.4", user "postgres", database "p
ostgres", SSL off
```
如上提示，由于数据库身份验证失败导致

由于首次安装postgresql时会产生一个`postgres`的Unix用户和一个`postgres`
的数据库用户，Unix用户的密码可以不管，但由于没有设置密码，要执行postgres
命令必须切换到`postgres`用户下。而数据库用户`postgres`没有设置密码就会
导致外部进程无法连接，因为该数据库用户没有密码，连接时不给密码就会提示

```
2019-02-26 14:16:11.230 CST [19] postgres@postgres FATAL:  password authentication failed for user "postgres"
2019-02-26 14:16:11.230 CST [19] postgres@postgres DETAIL:  User "postgres" has no password assigned.
    Connection matched pg_hba.conf line 95: "host  all  all 0.0.0.0/0 md5"
2019-02-26 14:16:11.233 CST [20] postgres@postgres FATAL:  password authentication failed for user "postgres"
2019-02-26 14:16:11.233 CST [20] postgres@postgres DETAIL:  User "postgres" has no password assigned.
    Connection matched pg_hba.conf line 95: "host  all  all 0.0.0.0/0 md5"
```
而随便给个密码又会

```
FATAL:  password authentication failed for user "postgres"
```
解决方法就是修改数据库用户`postgres`的密码，切换到Unix的postgres用户下执行

```
$ su postgres
$ psql --command " ALTER USER postgres WITH PASSWORD '123456';" 
```
在Dockerfile中则是

```
USER postgres
RUN /etc/init.d/postgresql restart \
        && psql --command " ALTER USER postgres WITH PASSWORD '123456';" \
        && /etc/init.d/postgresql stop
```

# 2、
```
db-com   | 2019-02-26 13:41:56.227 CST [11] LOG:  could not open temporary statistics file "/var/run/postgresql/10-main.pg_stat_tmp/global.tmp": No such file or directory
```
按错误日志，创建该目录，并将所有者改为`postgres`

```
$ mkdir -p /var/run/postgresql/10-main.pg_stat_tmp \
$ chown -R postgres:postgres /var/run/postgresql/10-main.pg_stat_tmp
```
# 3、
```
db-com   | 2019-02-26 13:50:14.828 CST [1] LOG:  listening on IPv4 address "127.0.0.1", port 5432
db-com   | 2019-02-26 13:50:14.828 CST [1] LOG:  could not bind IPv6 address "::1": Cannot assign requested address
db-com   | 2019-02-26 13:50:14.828 CST [1] HINT:  Is another postmaster already running on port 5432? If not, wait a few seconds and retry.
db-com   | 2019-02-26 13:50:14.831 CST [1] LOG:  listening on Unix socket "/var/run/postgresql/.s.PGSQL.5432"
db-com   | 2019-02-26 13:50:14.863 CST [6] LOG:  database system was shut down at 2019-02-26 12:28:48 CST
db-com   | 2019-02-26 13:50:14.881 CST [1] LOG:  database system is ready to accept connections
```
但是web服务连接时却出现

```
web_1    | django.db.utils.OperationalError: could not connect to server: Connection refused
web_1    |     Is the server running on host "db" (172.18.0.3) and accepting
web_1    |     TCP/IP connections on port 5432?
```
这个问题比较微妙，我目前已知的原因有三个，目前正在接触的web项目为django，
所以此处列出django的示例，提供参考

# 1、postgresql的`postgresql.conf`
在该文件中找到

```
#  - Connection Settings -

listen_addresses = '127.0.0.1'        #  what IP address(es) to listen on;
                    #  comma-separated list of addresses;
                    #  defaults to 'localhost'; use '*' for all
                    #  (change requires restart)
port = 5432             #  (change requires restart)
max_connections = 100           #  (change requires restart)
# superuser_reserved_connections = 3 #  (change requires restart)
```
将`listen_addresses`改为`listen_addresses = '0.0.0.0'`

这是由于，在容器中启动时`127.0.0.1`总是指向容器本身，导致无法监听来自外部其他
容器的访问。

# 2、postgresql的`pg_hba.conf`
在该文件中中找到

```
 #  Database administrative login by Unix domain socket
 local   all             postgres                                peer
 
 #  TYPE  DATABASE        USER            ADDRESS                 METHOD
 
 #  "local" is for Unix domain socket connections only
 local   all             all                                     trust
 #  IPv4 local connections:
 host    all             all             127.0.0.1/32            md5
 #  IPv6 local connections:
 host    all             all             ::1/128                 md5
```
保证和上面代码内容相同，其中`trust`在较新的版本中可以使用`peer`代替
# 3、django的配置文件`settings.py`
找到连接数据库的配置内容

```
DATABASES = {
    'default': {
        # 'ENGINE': 'django.db.backends.sqlite3',
        # 'NAME': os.path.join(BASE_DIR, 'db.sqlite3'),
        'ENGINE': 'django.db.backends.postgresql_psycopg2',
        'NAME': 'postgres',
        'USER':'postgres',
        'PASSWORD':'123456',
        'HOST':'db',
        'PORT':'5432',
    }
```
其中`  'HOST':'db'`需要指向具体容器的`服务名称`，原因和上面一样，如果使用
`localhost`将会一直指向容器本身导致无法正确访问数据库。也就是django会在
当前容器下查找数据库服务，但是数据库服务却在其他容器提供。

由于我时使用`docker-compose`来管理容器，而`compose`则是`项目`和`服务`
的概念，这里稍微转换一下

`compose`将数个容器提供的服务整个成一起，称为一个`项目`，而每个容器本身提供服务，
故将容器称为`服务`

如果没有使用`compose`的话，比如直接使用docker网桥服务来连接容器将的
通信，则这里的`HOSTS`则指向该容器在网桥中的IP，可以使用

```
docker inspect <container id>
```
来查看具体容器的IP

# 4、
```
PermissionError: [Errno 13] Permission denied: '/code/jimi_commd.log'
```
用更高的用户权限或者修改相关文件的权限和所有者

# 5、
```
no pg_hba.conf entry for host "172.18.0.4", user "postgres", database "postgres", SSL on
```
将`172.18.0.4`添加进`pg_hba.conf`

```
echo "host all all 172.18.0.4/16 md5" >> pg_hba.conf
```
# 启动redis错误
![QQ截图2019022716234920190227.png](/static/file/QQ截图2019022716234920190227.png)

![QQ截图2019022716242920190227.png](/static/file/QQ截图2019022716242920190227.png)

![QQ截图2019022716245920190227.png](/static/file/QQ截图2019022716245920190227.png)

修改配置文件`redis.conf`,找到

```
bind 127.0.0.1
# protected-mode yes
```
改为

```
# bind 127.0.0.1
protected-mode no
```
注释掉`bind`而不是改成`0.0.0.0`就能允许所有主机连接

`protected-mode no` 则是关闭安全模式

修改完这两部之后需要重启`redis`并执行

```
$ redis-server --protected-mode no
```
使配置生效，否则会一直提示

```
redis.exceptions.ResponseError: DENIED Redis is running in protected 
mode because protected mode is enabled, no bind address was specified,
no authentication password is requested to clients. In this mode 
connections are only accepted from the loopback interface. 
If you want to connect from external computers to Redis you may adopt 
one of the following solutions: 1) Just disable protected mode sending the
command 'CONFIG SET protected-mode no' from the loopback interface 
by connecting to Redis from the same host the server is running, however 
MAKE SURE Redis is not publicly accessible from internet if you do so. 
Use CONFIG REWRITE to make this change permanent. 2) Alternatively you
can just disable the protected mode by editing the Redis configuration file, 
and setting the protected mode option to 'no', and then restarting the 
server. 3) If you started the server manually just for testing, restart it with 
the '--protected-mode no' option. 4) Setup a bind address or an 
authentication password. NOTE: You only need to do one of the above 
things in order for the server to start accepting connections from the 
outside.
```
# 解决错误参考：
[docker-compose error # 55](https://github.com/kartoza/docker-postgis/issues/55)

[PostgreSQL学习第五篇--监听地址及端口修改](https://blog.csdn.net/ghostliming/article/details/53312493)

[如何设置PostgreSQL允许被远程访问](http://lazybios.com/2016/11/how-to-make-postgreSQL-can-be-accessed-from-remote-client/)

[如何获取 docker 容器(container)的 ip 地址](https://blog.csdn.net/sannerlittle/article/details/77063800)

[Postgresql: password authentication failed for user “postgres”](https://stackoverflow.com/questions/7695962/postgresql-password-authentication-failed-for-user-postgres)

[Postgres 9.5-main.pg_stat_tmp: No such file or directory](https://serverfault.com/questions/881064/postgres-9-5-main-pg-stat-tmp-no-such-file-or-directory)

[PostgreSQL script fails # 2](https://github.com/torhve/pix/issues/2)

