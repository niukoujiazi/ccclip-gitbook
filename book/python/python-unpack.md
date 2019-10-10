Pyinstaller 打包 Django
=========

# 参考

[pyinstaller简洁教程](http://legendtkl.com/2015/11/06/pyinstaller/)

[用PyInstaller把Python代码打包成单个独立的exe可执行文件](https://www.crifan.com/use_pyinstaller_to_package_python_to_single_executable_exe/comment-page-1/)

[很重要：pyinstaller 打包总结](https://blog.csdn.net/qq_34769196/article/details/83028955)

[Bundling data files with PyInstaller (--onefile)](https://stackoverflow.com/questions/7674790/bundling-data-files-with-pyinstaller-onefile)

[pyinstaller打包django项目](https://blog.csdn.net/qq_34809033/article/details/81873896)

[Python3.5 django1.11用pyinstaller打包的一些操作问题](https://www.jianshu.com/p/986bc1f89a15)

[Pyinstaller](https://pyinstaller.readthedocs.io/en/stable/)

[Using spec File](https://pyinstaller.readthedocs.io/en/stable/spec-files.html)

[pyinstaller 打包django1.8](https://my.oschina.net/ayy/blog/543953)
# 快速使用
安装
linux or mac
你可以从PyPi上下载安装，当然也可以使用pip或者easy_install来安装。

1
2
3
pip install pyinstaller
or
easy_install pyinstaller
版本更新

1
2
3
pip install --upgrade pyinstaller
or
easy_install --upgrade pyinstaller
windows
Windows上运行PyInstaller还需要PyWin32或者pypiwin32，其中pypiwin32在你安装PyInstaller的时候会自动安装。

使用PyInstaller
PyInstaller分析你的python程序，找到所有的依赖项。然后将依赖文件和python解释器放到一个文件夹下或者一个可执行文件中。

打包成一个文件夹
当使用PyInstaller打包的时候，默认生成一个文件夹，文件夹中包含所有依赖项，以及可执行文件。打包成文件夹的好处就是debug的时候可以清楚的看到依赖项有没有包含。另一个好处是更新的时候，只需要更新可执行文件就可以了。当然缺点也很明显，不方便，不易管理。

1
pyinstaller script.py
那么它是如何工作的呢？PyInstaller的引导程序是一个二进制可执行程序。当用户启动你的程序的时候，PyInstaller的引导程序开始运行，首先创建一个临时的Python环境，然后通过Python解释器导入程序的依赖，当然他们都在同一个文件夹下。

打包成一个文件
我们可以用onefile参数将所有文件打包到一个可执行文件中。

1
pyinstaller --onefile script.py
打包成一个文件相对于文件夹更容易管理。坏处运行相对比较慢。这个文件中包含了压缩的依赖文件拷贝（.so文件）。

当程序运行时，PyInstaller的引导程序会新建一个临时文件夹。然后解压程序的第三方依赖文件到临时文件夹中。这也是为什么一个可执行文件比文件夹中执行的时间要长的原因。剩下的就和上面的一样了。

spec 文件
当你执行下面命令

1
pyinstaller options..script.py
PyInstaller首先建一个sepc(specification)文件：script.spec。这个文件的存放地址可以使用参数–specpath= 来定义，默认放在当前文件夹下。

spec文件的作用是什么呢？它会告诉PyInstaller如何处理你的py文件，它会将你的py文件名字和输入的大部分参数进行编码。PyInstaller通过执行spec文件中的内容来生成app，有点像makefile。正常使用中我们是不需要管spec文件的，但是下面几种情况需要修改spec文件：

需要打包资源文件
需要include一些PyInstaller不知道的run-time库
为可执行文件添加run-time 选项
多程序打包
可以通过下面命令生成spec文件

1
pyi-makespec options script.py [other scripts ...]
修改完spec文件，就可以通过下面命令来生成app文件了

1
pyinstaller options script.spec
当通过spec文件来生成app文件的时候只有下面几个参数是有用的：

–upx-dir=
–distpath=
–noconfirm=
–ascii
spec 文件解析
下面是一个spec文件的例子。

1
2
3
4
5
6
7
8
9
10
11
12
13
14
block_cipher = None
a = Analysis(['minimal.py'],
        pathex=['/Developer/PItests/minimal'],
        binaries=None,
        datas=None,
        hiddenimports=[],
        hookspath=None,
        runtime_hooks=None,
        excludes=None,
        ciper=block_cipher)
pyz = PYZ(a.pure, a.zipped_data,
        cipher=block_cipher)
exe = EXE(pyz, ...)
coll = COLLECT(...)
spec文件中主要包含4个class: Analysis, PYZ, EXE和COLLECT.

Analysis以py文件为输入，它会分析py文件的依赖模块，并生成相应的信息
PYZ是一个.pyz的压缩包，包含程序运行需要的所有依赖
EXE根据上面两项生成
COLLECT生成其他部分的输出文件夹，COLLECT也可以没有
修改spec文件
我们上面说过有时候PyInstaller自动生成的spec文件并不能满足我们的需求，最常见的情况就是我们的程序依赖我们本地的一些数据文件，这个时候就需要我们自己去编辑spec文件来添加数据文件了。
上面的spec文件解析中Analysis中的datas就是要添加到项目中的数据文件，我们可以编辑datas.

1
2
3
4
5
6
a = Analysis(
    ...
    datas = [('you/source/file/path','file_name_in_project'),
    ('source/file2', 'file_name2')]
    ...
    )
可以认为datas是一个List,每个元素是一个二元组。元组的第一个元素是你本地文件索引，第二个元素是拷贝到项目中之后的文件名字。除了上面那种写法，也可以将其提出来。

1
2
3
4
5
6
7
added_files = [...]

a = Analysis(
    ...
    datas = added_files,
    ...
    )
其他的二进制文件添加方法类似。

总结
最后简单来说，我们要通过PyInstaller生成可执行的文件主要下面两步。

1
pyinstaller [option] mypython.py
option为空生成文件夹，选择onefile，生成一个文件。
如果项目有一些依赖的数据文件，上面生成的二进制文件是无法运行的，这个时候可以通过修改spec文件，让后再用pyinstaller运行spec文件。

1
pyinstaller [option] mypython.spec
当然也按上文那样先生成spec文件。



# 出现的问题
# not module name ...
打包时出线找不到某些模块，使用`pyinstaller`打包时似乎无法打包进

```
from module import name
```
这种格式的模块，所以如果在打包时提示没有找到模块的话，最简单有效的手段
就是将找不到的模块`import`进文件。

例如我在打包`django`的主程序`manage`时出现

`not module name 'xadmin.plugins'`

最快捷的解决方法是编辑`mange.py`文件，加上

```
import xadmin.migrations
```
就能解决问题。

但是，有些模块，直接`import`进入主文件可能会导致很多莫名其妙的问题。
无厘头，我也不明白为什么`import`就会直接出错。

这个时候，直接在主文件中`import`就无法解决问题了，因为`import`直接报错。

对我可行的解决方法是找到缺失模块的文件，在该文件中`import`

比如`xadmin.plugins`直接`import`将会出错，于是检查上面报错`no module name`
部分的信息，找到是哪个文件使用的，在该文件中`import xadmin.plugins`，便可解决问题。

顺便一体，我找到我的`django/myapp/urls.py`文件中使用了`from xadmin.plugins import ...`。
于是直接加上`import`问题便解决了。

另外网上还提供一种解释及解决方法：

`pyinstaller`无法打包隐式引入的模块（`from ... impor ...`)，所以需要手动告知
`pyinstaller`打包这文件。
方法是修改`.spec`文件。关于这个文件的详细解释可以看一下
[官方文档](https://pyinstaller.readthedocs.io/en/stable/spec-files.html)
的详细说明

操作是修改`hiddenimports`，将需要打包的模块写入。比如

```
hiddenimport =['rest_framework.authentication','xadmin.plugins']
```
# TemplateDoesNotExist as /
找不到模板文件

解决方法是将文件复制到制定位置，具体需要复制到的路径可以在报错页面内找到
`Template-loader postmortem`处将提示`Django`查找文件的路径及顺序。
将需要的文件复制进去便可。

但是这只是对于打包成文件而言可以这么做，而打包成单文件的话，就只能在打
包时将静态文件复制进去。这时就需要用到`.spec`文件的`datas`部分。

`datas`是二元组列表格式，二元组格式为

```
datas=[
    ('需要复制的文件的绝对路径','文件需要复制进打包后的文件的位置'),
    ('','')
]
```
这样，在打包后文件将复制到对应位置，解决`TemplateDoesNotExist`问题。

# 静态文件找不到(404)
打包之后的`django`项目，一切正常，就是静态文件找不到。

这里说的静态文件指图片,css,html等。可以尝试上面的添加`datas`二元组，将静态文件
一并打包进去。具体需要放置的位置就只能根据实际情况放了，我的django是放
在了`myapp`下面。

# django的/admin页面静态文件无法找到
关于这个页面邪门的不行！

通常，django渲染此页面的css及js会使用python的目录
`\Lib\site-packages\django\contrib\admin\static\admin`下的文件。

但是，使用`pyinstaller`打包时，`pyinstaller`将会把`django`的模块复制进包内。
最坑的地方就在这里，我们访问`/admin`页面所需的静态文件，打包后的项目
不会在`python`的安装目录里找，也不会在包内的`./django`模块目录内找。而是会
在你打包前的原目录里的`myapp/`文件夹里找！！！

巨坑！解决方法就是将`\Lib\site-packages\django\contrib\admin\static\admin`
下的文件全复制到`myapp/static/admin/`目录下。

