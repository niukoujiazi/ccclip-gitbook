#   centos 配置
#   ssh连接时间
因为ssh一般会默认几分钟内五操作就自动断开连接，而我
平时切出去查点资料就自己断开了，挺难受的，就改了配置文件
修改配置文件`/etc/ssh/ssh_config`(服务器和本地的位置一样)

```
sudo vim /etc/ssh/ssh_config
```

本地linux配置

```
ServerAliveInterval 50
ServerAliveCountMax 3
```

远程服务器上配置

```
ClientAliveInterval 50
ClientAliveCountMax 3
```

#   在linux上添加新用户

```
useradd username  使用useradd添加用户，username是要建立用户的名字
passwd username      使用passwd来修改用户的密码
su username    切换到指定用户
```

设置允许普用用户使用sudo命令，必须使用root权限的帐号执行visudo,直接修改/etc/sudoers文件也可做到，但visudo可以对输入进行语法检查，因为sudoers文件语法错误回导致所有sudo命令都无法使用
具体设置：

```
visudo
```

执行这个命令后会打开文件
找到 `root ALL=(ALL) ALL`这行，在其下新增一行
`username ALL=(ALL) ALL`注意这里的空格是`tab`，为防止出错建议直接复制root那行下来再做修改
保存并退出就可
 
[linux下添加用户并赋予root权限](https://blog.csdn.net/stormbjm/article/details/9086163)
 
修改 /etc/passwd 文件，找到如下行，把用户ID修改为 0 ，如下所示：
```
tommy:x:0:33:tommy:/data/webroot:/bin/bash
```

#   Ubuntu安装WPS
[Ubuntu 18.04 LTS 安装WPS](https://www.jianshu.com/p/3407c18e1337)
  
[linux版WPS地址](https://link.jianshu.com/?t=http%3A%2F%2Flinux.wps.cn%2F)
  
先到官网下载最新版，运行命令安装

```
运行安装包
sudo dpkg -i wps-office_10.1.0.5672~a21_amd64.deb
运行命令，修复依赖
sudo apt install -f

安装需要的软件包并重新安装wps
wget http://kr.archive.ubuntu.com/ubuntu/pool/main/libp/libpng/libpng12-0_1.2.54-1ubuntu1_amd64.deb
sudo dpkg -i libpng12-0_1.2.54-1ubuntu1_amd64.deb
sudo dpkg -i wps-office_10.1.0.5672~a21_amd64.deb

```

字体缺失问题：
[国内下载地址（百度云）](https://link.jianshu.com/?t=http%3A%2F%2Fpan.baidu.com%2Fs%2F1mh0lcbY)
 
```
#  a. 将得到文件复制到/usr/share/fonts

sudo cp * /usr/share/fonts

#  b. 执行以下命令,生成字体的索引信息

sudo mkfontscale

sudo mkfontdir

#  c. 更新字体缓存

sudo fc-cache
```


#   MySQL安装
先下载源

```
wget 'https://dev.mysql.com/get/mysql80-community-release-el7-1.noarch.rpm
```

再使用`yum localinstall`命令将`mysql`源添加进`/etc/yum.repos.d`

```
sudo yum localinstall mysql80-community-release-el7-1.noarch.rpm
```

之后就是安装

```
sudo yum install mysql-community-sever
```

如果启动mysql时报错


```
Job for mysqld.service failed because the control process exited with error code. See "systemctl status mysqld.service" and "journalctl -xe" for details.
```

有可能是`/var/lib/mysql`的目录不是空的，将这个目录删除在重新建一个空目录来启动`mysql`

由于新版本的Mysql没有默认密码，而是在一开始安装完成后产生一个随机密码，
需要在启动日志中找到这个密码才能登录MySQL
首先，先运行MySQL来时日志产生
centos下启动命令是

```
systemctl start mysqld.service
```

而ubuntu下则是

```
service mysqld start
```

运行一次之后就可以在日志文件`/var/log/mysql.log`中找到日志文件，
但因为日志就算只是启动一次依旧产生了很多，要找到一串随机密码还是很累的
可以使用

```
grep "temporary password" /var/log/mysqld.log
```

直接定位到那段密码

之后就需要使用获得的随机密码登录后并修改Mysql的密码

```
GRANT ALL PRIVILEGES ON *.* TO 'root'@localhost IDENTIFIED BY 'PASSWORD';
或则
alter user 'root'@'localhost' identified by '123';
```

#   安装Python3
因为现在趋势就是新的python3，而python2正在逐渐被替换，所以还是升级以下服务器上
python的版本

首先，安装以下依赖

```
yum install zlib-devel bzip2-devel openssl-devel ncurses-devel sqlite-devel readline-devel tk-devel gcc make
```

添加epel扩展

```
yum -y install epel-release
```

使用`wget`下载python3.7

```
wget https://www.python.org/ftp/python/3.7.0/Python-3.7.0.tar.xz
```

解压缩包

```
xz -d Python-3.7.0.tar.xz
tar -xf Python-3.7.0.tar
```

进入解压缩后的目录

```
cd python-3.7.0
```

现在就可以准备配置文件，但在这之间可以先安装`gcc`编译环境

```
sudo yum install -y gcc
```

执行手动编译将python3.7安装到`/usr/local`

```
./configure prefix=/usr/local/python3
```

之后需要标记，标记需要使用`root`帐号

```
sudo su
```
切换到`root`帐号后

```
make && make install
```

如果没有报错就说明正确安装了，可以在`/usr/local`下看到python目录

如果出现错误

```
ModuleNotFoundError: No module named '_ctypes'
```

可能是由于python3.7需要一个新的依赖包，安装一下

```
yum install libffi-devel -y
```

在重新mark之后就行了

之后就可以使用`python3`来使用python3.7，而python2直接`python`就额可以
但我们现在主要用的是python3，所以可以添加软链接

先将原来的python备份

```
mv /usr/bin/python /usr/bin/python.bak
```

添加python3的软连接

```
sudo ln -s /usr/local/python3/bin/python3.7 /usr/bin/python
```

完成后测试一下

```
python -V
```

输入python3.7就说明成功了

最后修改`yum`的软连接，让其指向python3

```
sudo vi /usr/bin/yum
将开头的#  ! /usr/bin/python 改为 # ! /usr/bin/python2
sudo vi /usr/libexec/urlgrabber-ext-down
同样将开头的改为 #  ! /usr/bin/python2
由于防火墙也是使用python2,所以防火墙的配置也要修改
sudo vim /usr/bin/firewall-cmd
将#  !/usr/bin/python -Es 改为 # !/usr/bin/python2
sudo vim /usr/bin/firewall-offline-cmd
将#  !/usr/bin/python -Es 改为 # !/usr/bin/python2
/usr/sbin/firewalld通用做修改
以免防火墙报错，找不到gi模块
vim /usr/sbin/firewalld
```

#   安装pip

```
yum -y install epel-release
yum install python-pip
pip install --upgrade pip
```

#   安装vim
先检查一些vim是否安装，缺了哪些

```
rpm -qa|grep vim
```

如果输出的内容中
`vim-common`,`vim-enhanced`,`vim-minimal`，`vimfilesystem`
四个包都有表示安装成功，缺哪个就安装哪个，比如缺少`vim-enhaced`

```
yum -y install vim-enhanced
```

之后可以对vim做一些配置，以便其使用，vim很强，我就用一点的就够用了

在用户目录下新建文件`.vimrc`

```
touch .vimrc
```

进入配置

```
vim .vimrc
```
输入配置内容

```
set nu         // 这是设置显示行号
   set  showmode   //设置在命令行界面最下面显示当前模式等。
   set   ruler     // 在右下角显示光标所在的行数等信息
   set autoindent   // 设置每次单击Enter键后，光标移动到下一行时与上一行的起始字符对齐
   syntax on    // 即设置语法检测，当编辑C或者Shell脚本时，关键字会用特殊颜色显示
  set tabstop=4 //设置tab键缩进为4
  set shiftwidth=4 //设置每一级的缩进长度
//解决中文乱码
set fileencodings=utf-8,gb2312,gb18030,gbk,ucs-bom,cp936,latin1
set enc=utf8
set fencs=utf8,gbk,gb2312,gb18030
```
#   方向键乱码问题（`^[[A^[[B^[[C^[[D`）
参考：
 
[ssh 连上服务器之后，终端的上下左右键变成乱码，着急哦～～～！！](https://segmentfault.com/q/1010000000521318)

[在Linux中，按上下左右键为什么变成`^[[A^[[B^[[C^[[D？`](https://www.zhihu.com/question/31429658)

造成这种情况可能是创建用户是没有指定环境
需要将用户环境改为 `/bin/bash`

```
sudo vim /etc/passwd
```

找到需要修改的用户在后面加上`/bin/bash`

或者，在终端中输入

```
>> bash
>> chsh
```

在`Login Shell [*]`后输入：

```
/bin/bash
```




