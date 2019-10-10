 Tcpdump 抓包工具
 =======

[抓包工具 tcpdump](http://blog.51cto.com/dngood/1084796)
 
[抓包神器 tcpdump 使用介绍](https://cizixs.com/2015/03/12/tcpdump-introduction/)
 
[CentOS下使用tcpdump网络抓包](http://blog.51cto.com/51eat/1887783)
 
[tcpdump常用参数说明](https://www.centos.bz/2018/01/tcpdump%E5%B8%B8%E7%94%A8%E5%8F%82%E6%95%B0%E8%AF%B4%E6%98%8E/)



----


监听指定端口

```

$ tcpdump -i eth0 -nnA ‘port 80’

```