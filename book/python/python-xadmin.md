Python 模块 xadmin 安装问题
=====

# 参考
[Django2.1集成xadmin管理后台所遇到的错误集锦，解决填坑](https://www.cnblogs.com/xingfuggz/p/10142388.html)

[cannot import name 'password_reset_confirm'](https://blog.csdn.net/sunfellow2009/article/details/81477368)

[django2.x报错No module named 'django.core.urlresolvers'](https://blog.csdn.net/weixin_35757704/article/details/78977753)
# 前言
将项目的`django`升级到2.1之后，`xadmin`爆炸了，网上资料比较齐全，这里多此一举
整合一下
# 手动安装xadmin模块
# 下载
从[这里](https://github.com/sshwsfc/xadmin)下载
# 手动安装
```
$ pip install xadmin-master.zip
```
如果安装出现问题，可以尝试使用一个空文件重命名为`README.rst`，替换掉
zip包内的`README.rst`文件

# 解决兼容问题
# ModuleNotFoundError: No module named 'django.core.urlresolvers' 
原因为`django2.0`将包`django.core.urlresolvers`重命名为`django.urls`。

因此，只要将所有引用该包的地方全部改成`django.urls`，比如

```
from django.core.urlresolvers  import reverse
# 改为
from django.urls import reverse
```
# ForeignKey引发的错误
将`所有`使用`models.ForeignKey`的地方，在后面补上引用外键`on_delete=models.CASCADE`

```
user = models.ForeignKey(AUTH_USER_MODEL)
# 改为
user = models.ForeignKey(AUTH_USER_MODEL,on_delete=models.CASCADE)
```
# dashboard.py文件出现参数数量问题
```
File "...\xadmin\views\dashboard.py",line 285 ,in __init__ ....
TypeError: __init__() takes 1 positional argument but 6 were given
```
解决方法：

```
forms.Field.__init__(self, required, widget, label, initial, help_text, *args, **kwargs)
# 改为
forms.Field.__init__(self)
```
# ImportError: cannot import name 'login' from 'django.contrib.auth.views'
```
# 将
from django.contrib.auth.views import login
from django.contrib.auth.views import logout
# 改为
from django.contrib.auth import authenticate, login, logout
```
# cannot import name 'password_reset_confirm'
```
# 将
from django.contrib.auth.views import password_reset_confirm
# 改为
from django.contrib.auth.views import PasswordResetForm
```
# ImportError: cannot import name 'QUERY_TERMS' from 'django.db.models.sql.query'
```
#  django2.1.1版本将xadmin\plugins\filters.py文件中的
from django.db.models.sql.query import LOOKUP_SEP, QUERY_TERMS
 
#  修改为
from django.db.models.sql.query import LOOKUP_SEP, Query
 
 
#  在Django2.0版本中把
from django.db.models.sql.query import LOOKUP_SEP, QUERY_TERMS
 
#  修改为：
from django.db.models.sql.query import LOOKUP_SEP
from django.db.models.sql.constants import QUERY_TERMS

```
# ModuleNotFoundError: No module named 'django.contrib.formtools'
```
#  卸载旧版本
pip uninstall django-formtools
 
#  安装新版本
pip install django-formtools
```

