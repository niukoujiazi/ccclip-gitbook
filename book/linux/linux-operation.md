#  Linux操作
#  zip
使用zip命令解压文件到制定目录

```
unzip filename.zip -d indexname
```

其中`unzip`时zip解包命令，`-d`用于指定要解压到的目录
#  unar
解压windows系统下压缩的zip文件时，会产生中文乱码现象

```
$ sudo apt-get install unar
$ unar your.zip
```

#  centos 端口
[centos7 firewalld(防火墙)和systemctl(开机启动服务)相关](https://github.com/cytggit/Map-openlayers/wiki/centos7--firewalld(%E9%98%B2%E7%81%AB%E5%A2%99)%E5%92%8Csystemctl(%E5%BC%80%E6%9C%BA%E5%90%AF%E5%8A%A8%E6%9C%8D%E5%8A%A1)%E7%9B%B8%E5%85%B3)

centos上防火墙的端口设置
centos7默认是没有`iptables`，所以通过`iptables`命令来配置是不可以的
centos采用了`firewalld`防火墙
查看80端口是否开启

```
firewall-cmd --query-port=80/tcp
```

开启80端口

```
firewall-cmd --zone=public --add-port=80/tcp --permanent
```

命令含义：
`--zone`  作用域
`--add-port=80/tcp`  添加端口，格式为：端口号/通讯协议
`--permanent ` 永久生效
添加后要重启才能生效
`firewall-cmd --reload`
查看已开启的端口
`firewall-cmd --list-ports`

关闭80端口

```
firewall-cmd --remove-port=80/tcp --permanent
```

`unix`环境下小于1024的端口不能被用户直接使用(可以试一下sudo)
#  文件传输
从服务器上下载文件目录
需要先在服务器上安装`vsftpd`
检查是否安装过

```
rmp -q vsftpd
```

如果没有安装的话，使用

```
yum -y install vsftpd
```

启动vsftpd

```
server vsftpd start
```

检查`vsftpd`是否开启

```
ps -e | grep vsftpd
```

检查21端口是否被监听

```
netstat -an | grep 21
```

注意：ftp协议是使用21号端口，所以21号端口必须开启


#  其实也可以不用那么麻烦
要下载目录直接在本地使用wget命令就行

```
wget ftp://45.77.95.163:21/ --ftp-user=junhan --ftp-password=@Jun8196285 -r
```

ftp跟上要连接的服务器ip:端口号，之后跟上登录ftp的用户名和密码
这个用户名和密码直接可以使用登录服务器的用户名和密码，或则可以创建单独作为ftp使用的用户名和密码，这个暂时没有去研究怎么创建

可以使用scp命令上传和下载
上传单个文件

```
scp -p port source_dictionary_file user@serverip:target_dictionary_file
```

port位置输入端口
`source_dictionary_file` 源文件的名字/位置
`target_dictionary_file` 目标地点的地址，必须是在服务器上存在的目录
或则使用另一个

```
scp /user/hellowood/wechar.jpg root@ip:/home/hellword/wechat.jpg
```

和上面一样，源文件地址和目标地址一定要正确

上传整个文件夹

```
scp -p port -r source_dictionary user@serverip:target_dictionary
```

或则
```
scp -r /userfile/file root@serverip:/home/file
```

对来，只要上面的用户名和密码正确，就会要求输入密码

下载

```
scp -p user@serverip:source_dictionary_file target_dictionary
```

或则

```
scp root@serverip:/home/file.jpg /home/wechat.jpg
```

下载整个文件夹

```
scp -p port -r user@serverip:/home/file /home/fiele2
```
#  Linux系统错误
`/etc/porfileh`这个文件不能乱改一旦出错都将导致整个命令行的命令都无法使用

如果发生这种情况可以使用

```
export PATH=/usr/local/sbin:/usr/local/bin:/sbin:/bin:/usr/sbin:/usr/bin:/root/bin
```
这句来修改系统配置，就可重新使用，之后就将porfile文件改正确
#  定义别名
```
# 示例：查看80端口
$ alias ps8='ps -aux | grep 80'
# 之后只需要执行ps8便可
```
# 批量删除有相同字符的文件
参考：[linux 删除所有后缀名相同的文件](https://blog.csdn.net/liulangdeshusheng/article/details/43152957)

```
$ find . -name '*.log' -exec rm -rf {} \;
```
`find`查找文件

`-name`根据某个字符查找，支持正则匹配

`-exec`后面跟一个指令，表示将前面得到的所有结果都一一执行该指令

`{}`占位符，会将前面匹配到的内容替换过来

`\;`不知道啥东西，可能是格式吧

#  netstat
```
　-t : 指明显示TCP端口
　-u : 指明显示UDP端口
　-l : 仅显示监听套接字(所谓套接字就是使应用程序能够读写与收发通讯协议(protocol)与资料的程序)
　-p : 显示进程标识符和程序名称，每一个套接字/端口都属于一个程序。
　-n : 不进行DNS轮询，显示IP(可以加速操作)
```
使用

```
$ netstat -tulpn |grep 10000
```
查询`10000`端口的使用情况
#  批量替换
sed -i "s/查找字段/替换字段/g" `grep 查找字段 -rl 路径`  

或者  grep -rl 查找字段  路径|xargs sed -i "s/查找字段/替换字段/g"

```
grep print.* -rl ./ | xargs sed -i "s/print\(.*\)/print(\1)/g"
```