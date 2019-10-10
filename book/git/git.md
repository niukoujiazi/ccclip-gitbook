git分布式版本控制系统
======

#  参考
[Git教程](https://www.liaoxuefeng.com/wiki/0013739516305929606dd18361248578c67b8067c8c017b000)

[git - 简易指南](http://www.bootcss.com/p/git-guide/)

[七步搭建 Git 服务器](https://segmentfault.com/a/1190000011313567# articleHeader5)

[git从远程库同步到本地仓库](http://scofieldwyq.github.io/2016/02/29/git%E4%BB%8E%E8%BF%9C%E7%A8%8B%E5%BA%93%E5%90%8C%E6%AD%A5%E5%88%B0%E6%9C%AC%E5%9C%B0%E4%BB%93%E5%BA%93/# d1)

[Git 基础 - 远程仓库的使用](https://git-scm.com/book/zh/v1/Git-%E5%9F%BA%E7%A1%80-%E8%BF%9C%E7%A8%8B%E4%BB%93%E5%BA%93%E7%9A%84%E4%BD%BF%E7%94%A8)

[起步 - 安装 Git](https://git-scm.com/book/zh/v1/%E8%B5%B7%E6%AD%A5-%E5%AE%89%E8%A3%85-Git)

[Download for Linux and Unix](https://git-scm.com/download/linux)

[CentOS 7 安装最新的 Git](https://ehlxr.me/2016/07/30/CentOS-7-%E5%AE%89%E8%A3%85%E6%9C%80%E6%96%B0%E7%9A%84-Git/)


#  Git的诞生
不得不提一下Git的诞生，看了几次都觉得牛逼得不行。（摘录自廖雪峰的Git教程）
很多人都知道，Linus在1991年创建了开源的Linux，从此，Linux系统不断发展，已经成为最大的服务器系统软件了。

Linus虽然创建了Linux，但Linux的壮大是靠全世界热心的志愿者参与的，这么多人在世界各地为Linux编写代码，那Linux的代码是如何管理的呢？

事实是，在2002年以前，世界各地的志愿者把源代码文件通过diff的方式发给Linus，然后由Linus本人通过手工方式合并代码！

你也许会想，为什么Linus不把Linux代码放到版本控制系统里呢？不是有CVS、SVN这些免费的版本控制系统吗？因为Linus坚定地反对CVS和SVN，这些集中式的版本控制系统不但速度慢，而且必须联网才能使用。有一些商用的版本控制系统，虽然比CVS、SVN好用，但那是付费的，和Linux的开源精神不符。

不过，到了2002年，Linux系统已经发展了十年了，代码库之大让Linus很难继续通过手工方式管理了，社区的弟兄们也对这种方式表达了强烈不满，于是Linus选择了一个商业的版本控制系统BitKeeper，BitKeeper的东家BitMover公司出于人道主义精神，授权Linux社区免费使用这个版本控制系统。

安定团结的大好局面在2005年就被打破了，原因是Linux社区牛人聚集，不免沾染了一些梁山好汉的江湖习气。开发Samba的Andrew试图破解BitKeeper的协议（这么干的其实也不只他一个），被BitMover公司发现了（监控工作做得不错！），于是BitMover公司怒了，要收回Linux社区的免费使用权。

Linus可以向BitMover公司道个歉，保证以后严格管教弟兄们，嗯，这是不可能的。实际情况是这样的：

Linus花了两周时间自己用C写了一个分布式版本控制系统，这就是Git！一个月之内，Linux系统的源码已经由Git管理了！牛是怎么定义的呢？大家可以体会一下。

Git迅速成为最流行的分布式版本控制系统，尤其是2008年，GitHub网站上线了，它为开源项目免费提供Git存储，无数开源项目开始迁移至GitHub，包括jQuery，PHP，Ruby等等。

历史就是这么偶然，如果不是当年BitMover公司威胁Linux社区，可能现在我们就没有免费而超级好用的Git了。


#  安装Git
#  Linux上安装Git
在终端输入`git`，看看系统有没有安装好

```
$ git
usage: git [--version] [--help] [-C <path>] [-c <name>=<value>]
           [--exec-path[=<path>]] [--html-path] [--man-path] [--info-path]
           [-p | --paginate | --no-pager] [--no-replace-objects] [--bare]
           [--git-dir=<path>] [--work-tree=<path>] [--namespace=<name>]
           <command> [<args>]
```

大多数linux都会友好的告诉你是否安装了

如果没有安装的话就安装一下
先安装依赖：

ubuntu/debian:

```
$  apt-get install libcurl4-gnutls-dev libexpat1-dev gettext \
    libz-dev libssl-dev
```
centos:

```
$ yum install curl-devel expat-devel gettext-devel \
  openssl-devel zlib-devel
```
进行安装：

ubuntu/debian:

```
$ sudo apt-get install git
```
centos

```
$  yum install git
```
#  源码包安装
下载最新版git安装包

```
$ wget https://github.com/git/git/archive/v2.9.2.tar.gz
```
解压到指定目录

```
$ tar -xzvf v2.9.2.tar.gz -C ~/app/
```
安装

```
$ cd git-2.9.2
$ make prefix=/usr/local/git all
$ make prefix=/usr/local/git install
```
#  创建版本库
什么是版本库呢？版本库又名仓库，英文名repository，你可以简单理解成一个目录，这个目录里面的所有文件都可以被Git管理起来，每个文件的修改、删除，Git都能跟踪，以便任何时刻都可以追踪历史，或者在将来某个时刻可以“还原”。

创建版本库只需要你选择一个“适合”的目录，执行`git init`便可，不管是什么目录，
这里我选择新建一个目录给git使用

```
# 创建目录
$ mkdir /home/user/git
# 移动到目录下
$ cd /home/user/git
# 使其变成git仓库
$ git init
```

仓库就建好了，可以使用命令`ls -al`在仓库目录下找到一个隐藏的目录`.git`
#  工作流
你的本地仓库由 git 维护的三棵“树”组成。第一个是你的 `工作目录`，它持有
实际文件；第二个是 `缓存区（Index）`，它像个缓存区域，临时保存你的改动；
最后是`HEAD`，指向你最近一次提交后的结果
![trees20190204.png](/static/file/trees20190204.png)
#  常用命令

#  查看仓库当前状态

```
git status
```

#  将改动添加到缓存区

```
git add <filename>
git add *
```

#  实际提交改动

```
git commit -m '代码提交信息'
```

#  查看文件改动前后不同之处

```
git diff
```

# 查看代码提交日志（由近到远）

```
git log

# 简化日志输出
git log --pretty=oneline
```

需要友情提示的是，你看到的一大串类似1094adb...的是commit id（版本号），和SVN不一样，Git的commit id不是1，2，3……递增的数字，而是一个SHA1计算出来的一个非常大的数字，用十六进制表示，而且你看到的commit id和我的肯定不一样，以你自己的为准。为什么commit id需要用这么一大串数字表示呢？因为Git是分布式的版本控制系统，后面我们还要研究多人在同一个版本库里工作，如果大家都用1，2，3……作为版本号，那肯定就冲突了。


#  版本回退

首先，Git必须知道当前版本是哪个版本，在Git中，用HEAD表示当前版本，也就是最新的提交1094adb...（注意我的提交ID和你的肯定不一样），上一个版本就是HEAD^，上上一个版本就是HEAD^^，当然往上100个版本写100个^比较容易数不过来，所以写成HEAD~100。

```
git reset --hard HEAD^
```

如果回退到之前的版本，却发现不是自己想要的版本，想撤销这次操作，
但是这个时候再使用`git log`你就看不到最新的哪个版本了。毕竟上面已经明确
告诉你了`git log`显示的是由近到远的提交日志。这时候就需要用到`commit id`了。
运气好的话，在哪里打印过日志，就翻上去，找到最新版本的`commit id`，使用
`commit id`来回到未来的某个版本,比如这里看到先前最新的版本`commit id`
为`1094a`，那么就可以这么操作

```
git reset --hard 1094a
```

如果没有记录这个`commit id`却又后悔了回退版本，想撤销这次操作回到先前
未来的某个版本上，这个时候可以使用

```
git reflog
```

git提供`git reflog`来记录你的每一次操作，便可顺藤摸瓜找到先前的`commit id`
来回到指定的版本

#  撤销修改
当想要丢弃本次修改的时候怎么办，git是记录修改的，不是什么记事本，你撤销
回去便会回到原状，在工作区的每一个修改都将被git所记录。

什么是修改呢，创建一个文件夹，新建一个文件，在代码里增加一行，一个符号，都是修改

操作了修改之后，使用`git status`，git将会明确告诉你那些文件被修改了，并且
会提示你接下来要怎么操作

如果你还没有将文件提交到缓存区的话

```
$ git status
On branch master
Changes not staged for commit:
  (use "git add <file>..." to update what will be committed)
  (use "git checkout -- <file>..." to discard changes in working directory)

    modified:   readme.txt

no changes added to commit (use "git add" and/or "git commit -a")
```

根据提示，你可以使用`git checkout -- <filename>`丢弃此次修改

而如果已经将文件添加到缓存区时

```
$ git status
On branch master
Changes to be committed:
  (use "git reset HEAD <file>..." to unstage)

    modified:   readme.txt
```

同样提示你可以使用`git reset HEAD <filename>`来撤销此次添加到缓存区的文件

如果你已经`git commit -m '提交说明'`将修改提交了上去，就直接使用版本回退，
毕竟代码是在自己电脑上修改，都还有救，而但你将代码提交到git服务器上时，
那么，没救了，等死吧

#   删除
在git工作区中删除文件也属于修改。
如果确实需要删除文件，需要使用`git rm <filename>`，并提交上去

```
$ rm test.txt
$ git rm test.txt
$git commit -m '说明'
```

#  分支管理
创建分支

```
git checkout -b dev
# 这个命令相当于,下面两条
git branch dev
git checkout dev
```

`dev`是分支名，是自定义的名字

查看当前分支 `git branch `

切换当前分支 `git checkout <name>`

合并某个分支到当前分支`git merge <name>`

删除分支`git branch -d <name>`

强行删除分支`git btanch -D <name>`

查看分支合并图 `git log --graph`

禁用快速合并(Fast Forward)，并生成一个新的提交

```
git merge --no-ff -m "text" <branch name>
```
创建本地分支与远程分支的联系

```
git branch --set-upstream-to <branchname> origin/<branchnae>
```

#  保存工作现场
[廖雪峰的Git教程](https://www.jianshu.com/p/7b8f06aa6ba3)


    `git stash`
    将当前工作现场“储藏起来”

用`git stash list`查看保存的工作现场

恢复的方式有两种:

```
        git stash apply ‘stash’内容不删除，删除使用git stash drop
        git stash pop，恢复的同时把stash内容也删了
```
多次stash，恢复用git stash list查看，然后恢复指定的stash

```
git stash apply stash@{0}
```
#  搭建git服务器
git是分布式版本控制系统，同一个git仓库可以分布到不同的机器上。
多个机器上互相推送克隆，但为了能够避免其他人的电脑没开机等情况，这个时候
需要一台24小时开机的设备充当git服务器，以便每个人都将自己的代码修改推送
到这台机器上。

同样安装git

```
$ sudo apt-get install git
```

和客户端不同的地方在于，这个是要充当服务器的角色，所以需要一个账户以供给
其他人登录推送代码。所以需要创建一个账号

```
$ sudo adduser git
```

和客户端一样初始化一个目录作为git仓库

```
$ mkdir /gitpro
$ cd /gitpro
$ git init --bare name.git
```

加上`--bare`创建一个名为`name`的裸仓库。裸仓库没有工作区，因为服务器上
的git仓库纯粹是为了共享，同时也避免了用户直接登录到服务器上修改工作区。
通常服务器上的的git仓库会以`.git`后缀。

因为我们现在不是使用上面创建的`git`用户来操作这些，所以这个仓库的所有者
是当前用户（我用root用户操作，所以该文件的所有者是root），需要将所有者
改为`git`用户，这样才能使用`git`用户操作该目录

```
$ sudo chown -R git:git name.git
```

为了更安全的登录git服务器推送代码，需要用到ssh免密登录，这里只需要手机客户端
的公钥，保存在`git`用户目录下的`.ssh/authorized_keys`内便可，具体可以参考本
博客的另一篇文章“SSH秘钥(免密登录)”

需要注意的是，`.ssh`文件夹和`authorized_keys`的文件所有者必须是git用户。
并且，`.ssh`文件夹权限为`700`，`authorized_keys`文件权限为`600`。

最后禁用`git`用户使用shell登录，直接变价`/etc/passwd`文件，找到

```
git:x:1001:1001:jyt,,jyt,:/home/git:/bin/bash
```

将其改为

```
git:x:1001:1001:,,,:/home/git:/usr/bin/git-shell
```

这样，git用户可以正常通过ssh使用git，但无法登录shell，因为我们为git用户指定的git-shell每次一登录就自动退出。

# 克隆远程仓库

```
git clone git@server:/gitpro/name.git
```

上文提到的`server`   `gitpro`   `name.git`记得替换成自己的

# 关联远程仓库
git服务器有了之后，需要在客户端上关联远程仓库，以便从机器推送到服务器上

添加远程仓库
```
git remote add origin git@server:/gitpro/nam.git
```

查看关联的远程仓库名

```
git remote
```

删除关联的远程仓库名

```
git remote remove <name>
```

查看仓库信息

```
git remote show <name>
```

远程仓库重命名

```
git remote rename  <old name> <new name>
```

#  推送数据到远程库

```
git push -u origin master
```

这里的`origin`指的是关联的远程仓库名。`master`指的是要推送的分支。

# 抓取远程仓库并同步到本地仓库

```
# 抓取远程仓库
git fetch origin master
# 比较远程仓库和本地仓库差异
git log master.. origin/master
# 合并远程库
git merge origin/master
```