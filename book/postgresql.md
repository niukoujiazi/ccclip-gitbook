PostgreSQL install and use
====

#  参考：
[postgresql 数据导入导出](https://blog.csdn.net/allen_oscar/article/details/9061573)

[ubuntu 下 PostgreSQL 使用小记](https://blog.wenzhixin.net.cn/2014/01/12/hello_postgresql/)

[PostgreSQL新手入门](http://www.ruanyifeng.com/blog/2013/12/getting_started_with_postgresql.html)

[修改postgres密码](https://www.cnblogs.com/kaituorensheng/p/4735191.html)

[PostgreSQL 9.3.1 中文手册](http://www.postgres.cn/docs/9.3/app-psql.html)
#  安装PostgreSQL(debian)
安装PostgreSQL的服务端和客户端：

```
$ sudo apt-get install postgresql postgresql-client
```

初次安装后会默认生成一个名为"`postgres`"的Linux用户、名为"`postgres`"的
数据库和名为"`postgres`"的数据库用户。

在使用Postgresql时，`psql`将会默认使用和当前Linux用户名同名的数据库用户
名登录数据库
#  服务

```
#  查看状态
sudo /etc/init.d/postgresql status

#  启动
sudo /etc/init.d/postgresql start

#  停止
sudo /etc/init.d/postgresql stop

#  重启
sudo /etc/init.d/postgresql restart
```
#  错误
#  启动错误
安装后启动出现：

```
psql: could not connect to server: No such file or directory
    Is the server running locally and accepting
    connections on Unix domain socket "/var/run/postgresql/.s.PGSQL.5432"?
```
参考：[Why psql can't connect to server?](https://stackoverflow.com/questions/31645550/why-psql-cant-connect-to-server)

找到`pg_hba.conf`配置文件，选摘：

```
#  Database administrative login by Unix domain socket
local   all             postgres                                peer

#  TYPE  DATABASE        USER            ADDRESS                 METHOD

#  "local" is for Unix domain socket connections only
local   all             all                                     peer
#  IPv4 local connections:
host    all             all             127.0.0.1/32            md5
#  IPv6 local connections:
host    all             all             ::1/128                 md5
```
其中：

```
#  "local" is for Unix domain socket connections only
local   all             all                                     peer
```
将`peer`改为`trust`，并重启服务

```
sudo /etc/init.d/postgresql restart
```
#  启动服务错误
```
 * Restarting PostgreSQL 10 database server                                      * Error: Config owner (postgres:101) and data owner (root:0) do not match, and config owner is not root
                                                                         [fail]
```
某种原因导致一些目录和文件的所有者“不正确”，解决方法：

修改目录/文件所有者，一般只要改`/var/lib/postgresql/`文件夹下的便可

```
$ chown -R postgresql:postgresql /var/lib/postgresql/
```

#  在docker中部署postgresql启动错误
#  1、
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

#  2、
```
db-com   | 2019-02-26 13:41:56.227 CST [11] LOG:  could not open temporary statistics file "/var/run/postgresql/10-main.pg_stat_tmp/global.tmp": No such file or directory
```
按错误日志，创建该目录，并将所有者改为`postgres`

```
$ mkdir -p /var/run/postgresql/10-main.pg_stat_tmp \
$ chown -R postgres:postgres /var/run/postgresql/10-main.pg_stat_tmp
```
#  3、
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

#  1、postgresql的`postgresql.conf`
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

#  2、postgresql的`pg_hba.conf`
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
#  3、django的配置文件`settings.py`
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

#  4、
```
PermissionError: [Errno 13] Permission denied: '/code/jimi_commd.log'
```
用更高的用户权限或者修改相关文件的权限和所有者

#  5、
```
no pg_hba.conf entry for host "172.18.0.4", user "postgres", database "postgres", SSL on
```
将`172.18.0.4`添加进`pg_hba.conf`

```
echo "host all all 172.18.0.4/16 md5" >> pg_hba.conf
```