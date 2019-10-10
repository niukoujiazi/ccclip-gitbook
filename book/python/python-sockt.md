#  python sockt 网络编程
#  参考
[Python中的socket网络编程(TCP/IP，UDP)讲解](https://blog.csdn.net/csdn15698845876/article/details/78311576)
 
[Python socket编程](https://segmentfault.com/a/1190000002759586)
 
[TCP编程](https://www.liaoxuefeng.com/wiki/001374738125095c955c1e6d8bb493182103fac9270762a000/001386832511628f1fe2c65534a46aa86b8e654b6d3567c000)

[解决socket.error: [Errno 98] Address already in use问题](https://blog.csdn.net/chenyulancn/article/details/8181238)
 
#  实例


```
# /usr/bin/env python
# coding=utf-8

import socket
import threading

def tcplink(sock,addr):
    print('Accept new connectin from %s:%s...' % addr)
    sock.send('welcome'.encode())
        # send()发送TCP数据，将string中的数据发送到连接的套接字
    while True:
        data = sock.recv(1024)
            # recv()接受TCP数据，数据以字符串形式返回，可以指定要接受的最大数据量
        if data == 'exit' or not data:
            break;
        send_data = ('hello %s' % data).encode()
        sock.send(send_data)
    sock.close()
    print('connetction from %s:%s closed' % addr)



s = socket.socket(socket.AF_INET,socket.SOCK_STREAM)
    # AF_INET指定IPv4，SOCK_STREAM指定TCP连接
s.setsockopt(socket.SOL_SOCKET,socket.SO_REUSEADDR,1)
    # 设置给定套接字的选项，（）内容让套接字允许地质重用
s.bind(('127.0.0.1',10001))
    # 绑定定制和端口到套接字

s.listen(5)    # 开始监听，设置操作系统可以挂起的最大连接数，至少为1，一般为5
print('server is waiting connect...')

while True:    # 设置一个无限循环来实现监听
    sock,addr = s.accept()
        # accept()被动接受客户端连接，是阻塞式等待连接
#     socke,addr = s.recvfrom(1024)
    print('-'*20,sock,addr)
    t = threading.Thread(target = tcplink , args = (sock,addr))
        # 将套接字内容丢到线程中执行
    t.start()
        # 启动这个线程

```

#  客户端


```
# usr/bin/env python3
# coding=utf-8

import socket
import threading

s = socket.socket(socket.AF_INET,socket.SOCK_STREAM)

s.connect(('127.0.0.1',10001))
data = s.recv(1024)
print(data)
for i in [b'one',b'tow',b'three']:
    s.send(i)
    data = s.recv(1024)
    print(data)
s.send(b'exit')
s.close()
```
