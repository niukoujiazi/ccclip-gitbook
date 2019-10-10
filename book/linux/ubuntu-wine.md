Ubuntu环境安装wine
=======

#  在ubuntu环境下安装wine QQ轻聊版
参考
 
[最新wineQQ 完美解决方案](https://blog.csdn.net/LeonSUST/article/details/82156960)
 
需要先安装`deepin-wine`环境
安装`git`来克隆github项目

```
sudo apt-get install git
```

安转号`git`后使用其克隆项目

```
git clone "https://gitee.com/wszqkzqk/deepin-wine-for-ubuntu.git"
```
下载好之后进入下载好的文件夹并将其一键安装

```
cd deepin-wine-for-ubuntu
sudo sh ./install.sh
```

安装好环境之后就需要安转应用容器,到这个网址下载

[http://mirrors.aliyun.com/deepin/pool/non-free/d/](http://mirrors.aliyun.com/deepin/pool/non-free/d/)

推荐：

```
QQ：http://mirrors.aliyun.com/deepin/pool/non-free/d/deepin.com.qq.im/
TIM：http://mirrors.aliyun.com/deepin/pool/non-free/d/deepin.com.qq.office/
QQ轻聊版：http://mirrors.aliyun.com/deepin/pool/non-free/d/deepin.com.qq.im.light/
微信：http://mirrors.aliyun.com/deepin/pool/non-free/d/deepin.com.wechat/
Foxmail：http://mirrors.aliyun.com/deepin/pool/non-free/d/deepin.com.foxmail/
百度网盘：http://mirrors.aliyun.com/deepin/pool/non-free/d/deepin.com.baidu.pan/
360压缩：http://mirrors.aliyun.com/deepin/pool/non-free/d/deepin.cn.360.yasuo/
```
#  中文乱码（宋体）
#  参考
[彻底消除wine中文乱码](https://blog.csdn.net/williamyi96/article/details/79841690)

[如何解决wine中文乱码问题](https://jingyan.baidu.com/article/1974b28953c881f4b0f7744c.html)

[第一杯红酒：Wine让Windows程序运行在Linux上](https://lado.me/2017/04/27/wine-run-windows-program-on-linux/)

[simsun.ttc](https://github.com/sonatype/maven-guide-zh/blob/master/content-zh/src/main/resources/fonts/simsun.ttc)
#  下载字体
```
$ wget https://github.com/sonatype/maven-guide-zh/raw/master/content-zh/src/main/resources/fonts/simsun.ttc
```
将字体复制到指定文件夹

```
$ cp simsun.ttc /home/username/.wine/drive_c/windows/Fonts/
```
`Font`文件夹下，该文件夹由`wine`生成类似windows的目录，如果找不到可以

```
$ find /|grep 'windows/Fonts'
```
之后创建一个文件`zh.reg`，这是windows的注册表脚本，因为需要在windows
环境下执行，所以也必须使用wine
#  zh.reg
```
REGEDIT4 
[HKEY_LOCAL_MACHINE\Software\Microsoft\Windows NT\CurrentVersion\FontSubstitutes] 
“Arial”=”simsun” 
“Arial CE,238”=”simsun” 
“Arial CYR,204”=”simsun” 
“Arial Greek,161”=”simsun” 
“Arial TUR,162”=”simsun” 
“Courier New”=”simsun” 
“Courier New CE,238”=”simsun” 
“Courier New CYR,204”=”simsun” 
“Courier New Greek,161”=”simsun” 
“Courier New TUR,162”=”simsun” 
“FixedSys”=”simsun” 
“Helv”=”simsun” 
“Helvetica”=”simsun” 
“MS Sans Serif”=”simsun” 
“MS Shell Dlg”=”simsun” 
“MS Shell Dlg 2”=”simsun” 
“System”=”simsun” 
“Tahoma”=”simsun” 
“Times”=”simsun” 
“Times New Roman CE,238”=”simsun” 
“Times New Roman CYR,204”=”simsun” 
“Times New Roman Greek,161”=”simsun” 
“Times New Roman TUR,162”=”simsun” 
“Tms Rmn”=”simsun”
```
执行文件

```
$ wine regedit zh.reg
# 由于我是使用deepin-wine,所以我是这么做
$ /usr/bin/deepin-wine regedit zh.reg
```
#  winetricks
[Winetricks](https://wiki.winehq.org/Winetricks)

[winetricks wineserver non trovato!](https://neo2shyalien.eu/en/winetricks-wineserver-not-found.html)

# 安装
```
$ wget https://raw.githubusercontent.com/Winetricks/winetricks/master/src/winetricks
$ chmod +x winetricks
```
#  启动
```
$ sh winetricks <packname>
```
如果在没有参数的情况下运行，winetricks将显示一个包含可用包列表的GUI。
# 错误
```
------------------------------------------------------
wineserver not found!
------------------------------------------------------
```
解决方法：找到`winserver`制作软链接

```
$ locate wineserver
/usr/lib/i386-linux-gnu/deepin-wine/wineserver
/usr/lib/i386-linux-gnu/deepin-wine/wineserver.real
$ sudo ln -s /usr/lib/i386-linux-gnu/deepin-wine/wineserver /usr/local/bin/wineserver
```
#  其他
文章里还介绍了一个桌面图标样式的东西

```
sudo apt-get install gnome-shell-extension-top-icons-plus gnome-tweaks
```

安装好使用命令`gnome-tweaks`开启这个扩展
gnome可以对直接ubuntu的皮肤进行修改，其中可以自己安装一些皮肤

[How To Install the Adapta GTK Theme on Ubuntu](https://www.omgubuntu.co.uk/2016/10/install-adapta-gtk-theme-on-ubuntu)

[ ubuntu18.04 初体验 -- 自定义主题和配置](https://segmentfault.com/a/1190000014667195)