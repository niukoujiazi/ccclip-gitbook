Python 调用DLL
====

#  参考
[Python的学习（十五）---- 调用windows下DLL详解](https://blog.csdn.net/linda1000/article/details/8232421)

[python ctype 导入 dll 文件报错 invalid ELF header 有人遇到过吗](https://www.v2ex.com/t/244511)

[python ctypes 调用.dll and .so](https://blog.csdn.net/xc_zhou/article/details/84959594)
#  ctypes模块
加载DLL
加载的时候要根据你将要调用的函数是符合什么调用约定的
stdcall调用约定： 两种加载方式

```
Objdll = ctypes.windll.LoadLibrary("dllpath")
Objdll = ctypes.WinDLL("dllpath")
```
cdesl调用约定： 两种加载方式

```
Objdll = ctypes.cdll.LoadLibrary("dllpath")
Objdll = ctypes.CDLL("dllpath")
```
其实 windll 和 cdll 分别是WinDLL类和CDLL类的对象


