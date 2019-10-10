SSH秘钥(免密登录)
====

配置SSH秘钥，以达成免密登录服务器的效果
#  秘钥
SSH密钥提供一种更为安全的虚拟专有服务器登录机制。

不通过密码登录，而是通过ssh完成登录。使用ssh登录时，实际上客户端会优先使用秘钥
来与服务器进行配对(可以通过在命令中加入`-v`来查看，即`ssh -v name@server`)
，配对成功则直接登录（免密登录），配对失败将会尝试使用密码登录远程服务器。
ssh秘钥几乎无法以暴力方式破解。配置好服务器（公钥）和客户端（秘钥）之后便可。

#  生成秘钥对
在linux下使用命令

```
# 1、开始生成秘钥对
$ ssh-keygen 
# 提示秘钥需要保存的位置以及秘钥的名称，默认回车便可
Generating public/private rsa key pair.
Enter file in which to save the key (/root/.ssh/id_rsa):
Created directory '/root/.ssh'.
# 这里提示的密码是在使用秘钥时是否需要输入秘密，又不是什么军方需要特别秘密的东西，直接回车跳过
Enter passphrase (empty for no passphrase):
Enter same passphrase again:
# 完成后会提示类似的内容
Your identification has been saved in /home/demo/.ssh/id_rsa.
Your public key has been saved in /home/demo/.ssh/id_rsa.pub.
The key fingerprint is:
4a:dd:0a:c6:35:4e:3f:ed:27:38:8c:74:44:4d:93:67 demo@a
The key's randomart image is:
+--[ RSA 2048]----+
|          .oo.   |
|         .  o.E  |
|        + .  o   |
|     . = = .     |
|      = S = .    |
|     o + = +     |
|      . o + o .  |
|           . o   |
|                 |
+-----------------+
```

见到类似上面的输出，说明秘钥已经创建好了，可以在用户目录下找到，移动到用户目录下

```
# 移动到用户目录下
$ cd /home/user/.ssh
# 查看目录下文件
$ ls -al
id_rsa  id_rsa.pub
```

其中`id_rsa`是私钥，绝对不能给任何人，`id_rsa.pub`这个以`.pub`后缀的是
公钥，可以随意告别人。

要使用免密登录需要将公钥交给服务器来保管，以便服务器根据公钥来与客户端
的私钥配对
#  在远程服务器上保存公钥
要将公钥提交到服务器上，一般有两种方法
#  一、ssh-copy-id

```
ssh-copy-id name@server
```

# 二、直接登录服务器完成复制
这里使用`scp`命令将文件复制到服务器上

```
# 先将文件复制到服务器上
$ scp /home/user/.ssh/id_rsa.pub user@server:./.ssh
# 再将文件追加到`authorized_keys`内
$ cat /home/user/.ssh/idrsa.pub >> authorized_keys
```

其实直接查看`id_rsa.pub`文件，复制其内容，再直接粘贴到`authorized_keys`
文件末尾也是可以的

#  配置ssh

#  远程服务器上打开秘钥登录功能
编辑`/etc/ssh/sshd_config`文件，找到以下几项并修改如下

```
RSAAuthentication yes
PubkeyAuthentication yes
```

允许`root`用户使用密码登录

```
PermitRootLogin yes
```

如果需要禁用其他用户登录

```
PasswordAuthentication no
```

最后重启ssh服务

```
service sshd restart
```

# 出现的问题
如果按上文配置好后还是需要密码登录，可以进行以下检查

# 检查客户端ssh配置文件
找到`ssh_config`进行编辑，这个文件位置一般在`/etc/ssh/ssh_config`

```
$ vim /etc/ssh/ssh_conig
```

找到
```
IdentityFile ~/.ssh/id_rsa
```

这行，这行的意思是告诉客户端，使用秘钥登录时的客户端私钥可以在哪里找到。
一般默认是在`~/.ssh/id_rsa`，如果私钥位置有更改，可以根据实际情况增加一行，
比如：

```
IdentityFile ~/.ssh/id_git
```

# 检查远程服务器上的文件使用权限
这里的文件指的是`.ssh`文件夹权限和`authorized_keys`文件。
必须保证这两个文件只有该账户的用户能修改，以免被其他人修改。

所以，这里有两个注意点，一个是文件夹权限，一个是文件的所有者
可以通过命令`ls -al`来查看

```
$ ls -al
drwx------          2   root www-data    4096 Jan 29 15:39 .ssh
$ ls -al .ssh
drwx------          2   root www-data 4096 Jan 29 15:39 .
drwxrwxrwx   10 root root              4096 Feb  3 19:54 ..
-rw-------           1   root www-data 1194 Jan 29 15:39 authorized_keys
```

权限修改为

```
$ chmod 600 authorized_keys
$ chmod 700 .ssh
```

使用`ls -al`可以看到，当前文件的所有者是root用户，需要修改给指定的普通用户。
但是普通用户并没有权限可以创建文件夹（没有'写'权限）。

可以通过两个方法，给普通用户修改权限，或则由有权限的用户创建后再转给普通用户。
这里就只讲将文件所有权改给普通用户

```
$ chown git authorized_keys
$ chown git .ssh
```

#  参考
[Ssh 信任关系建立后仍需要输入密码](http://blog.itpub.net/29500582/viewspace-1251139/)

[配置ssh公钥登录提示还是输入密码](http://blog.51cto.com/zouqingyun/1874410)

[设置 SSH 通过密钥登录](https://www.jianshu.com/p/dc1884601037)

[如何设置SSH密钥](https://blog.csdn.net/zstack_org/article/details/71157842)

[如何更改linux文件的拥有者及用户组(chown和chgrp)](https://blog.csdn.net/hudashi/article/details/7797393)