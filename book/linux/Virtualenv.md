
Virtualenv
=====
#  参考：
[Python开发必备神器之一：virtualenv](http://codingpy.com/article/virtualenv-must-have-tool-for-python-development/)

#  基本用法
先创建一个虚拟环境以供使用

```
virtualenv ENV
# 这个命令将会在当前目录下创建一个和虚拟环境名同名的目录`ENV`
# 如果希望第三方包不会一起复制过来，可以使用`--no-site-packages`来得到
# 干净的没有第三方包的虚拟环境
virtualenv --no-site-packages ENV 
# 如果需要指定python版本
virtualenv -p /usr/bin/python2.7 venv
```

附上常用命令：

```
lsvirtualenv -b #  列出虚拟环境
workon [虚拟环境名称] #  切换虚拟环境
lssitepackages #  查看环境里安装了哪些包

cdvirtualenv [子目录名] #  进入当前环境的目录

cpvirtualenv [source] [dest] #  复制虚拟环境

deactivate #  退出虚拟环境

rmvirtualenv [虚拟环境名称] #  删除虚拟环境
```
