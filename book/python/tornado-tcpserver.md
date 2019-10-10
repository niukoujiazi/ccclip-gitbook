Tornado TCPServer
=======

#  参考

[https://cuiqingcai.com/6080.html](Python中logging模块的基本用法)

[https://codeday.me/bug/20190216/634978.html](如何将Tornado日志存储到文件？)

[https://www.jianshu.com/p/8c74a3d87567](tornado日志管理)

#  代码参考
```
# !/usr/bin/env python
# codig=utf-8

import tornado.ioloop
import tornado.options
import tornado.httpserver
import pymongo
import motor.motor_tornado
from tornado.log import enable_pretty_logging
import logging

from application import application

from tornado.options import define, options

#  日志
#  定义logger并起名server用于日志配置共享
logger = logging.getLogger("server")
logger.setLevel(level=logging.INFO)
handler = logging.handlers.RotatingFileHandler("/var/log/tornado/tornado.log",
        mode="a", maxBytes=1048000, backupCount=10, encoding="utf-8")
formatter = logging.Formatter('%(asctime)s - %(name)s - %(levelname)s - %(message)s')
handler.setFormatter(formatter)
# tornado的日志记录器，access是web访问信息
app_log = logging.getLogger("tornado.application")
access_log = logging.getLogger("tornado.access")
gen_log = logging.getLogger("tornado.general")
enable_pretty_logging()
app_log.addHandler(handler)
access_log.addHandler(handler)
gen_log.addHandler(handler)
logger.addHandler(handler)


define("port",default = 8000, help = "run on the given port", type = int)

class LogFormatter(tornado.log.LogFormatter):
    """
    重写tornado日志模块的格式
    """
    def __init__(self):
        super(LogFormatter, self).__init__(
                fmt='%(color)s[%(asctime)s %(filename)s:%(funcName)s:%(lineno)d %(levelname)s]%(end_color)s %(message)s',
                datefmt='%Y-%m-%d %H:%M:%S'
                )

def main():
    tornado.options.parse_command_line()
    # [i.setFormatter(LogFormatter()) for i in logging.getLogger().handlers]
    http_server = tornado.httpserver.HTTPServer(application)
    http_server.listen(options.port)

    print("runing at http://127.0.0.1:{}".format(options.port))
    print("Quite the server with Control-C")
    http_server.start(0)

    # 为每个子进程创建一个MotorClient
    application.settings['db'] = motor.motor_tornado.MotorClient("localhost",27017).testdb
    tornado.ioloop.IOLoop.instance().start()

if __name__ == "__main__":
    main()
```

#  日志
#  思路
tornado是使用`logging`模块来进行日志管理的，所以只需要添加一个logging的
处理器(`handler`)，将tornado自己的三个处理器`tornado.application` `tornado.access`
`tornado.access`获取到之后为其 `addHandler` 自己的处理器，就可以实现在
tornado有输出的同时将输出一份到文件

#  重写日志格式
tornado的日志格式是可以修改的。参考上面代码

```
class LogFormatter(tornado.log.LogFormatter):
    """
    重写tornado日志模块的格式
    """
    def __init__(self):
        super(LogFormatter, self).__init__(
                fmt='%(color)s[%(asctime)s %(filename)s:%(funcName)s:%(lineno)d %(levelname)s]%(end_color)s %(message)s',
                datefmt='%Y-%m-%d %H:%M:%S'
                )

def main():
    .........
    [i.setFormatter(LogFormatter()) for i in logging.getLogger().handlers]
    .........
```