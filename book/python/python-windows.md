Python注册windows服务
======

# 参考

[使用Python写Windows Service服务程序](https://www.bbsmax.com/A/kjdwAno2JN/)

[python实现windows Service服务程序](https://www.cnblogs.com/shhnwangjian/p/6844547.html)

[python打包成exe，并将exe注册到windows服务中](https://blog.csdn.net/Panda_813/article/details/86686473)

[Windows Server 2012的服务管理自动化 -启动类型设置，手动启动还是自动启动](https://blog.51cto.com/hongman/2103127)

# 代码
网上资料很多，就不多写了

```
# encoding=utf-8
import win32serviceutil
import win32service
import win32event
import os
import logging
import inspect
import servicemanager
 
class PythonService(win32serviceutil.ServiceFramework):   
 
    _svc_name_ = "PythonService"  # 服务名
    _svc_display_name_ = "Python Service Test"  # 服务在windows系统中显示的名称
    _svc_description_ = "This is a python service test code "  # 服务的描述
 
    def __init__(self, args):
        win32serviceutil.ServiceFramework.__init__(self, args)
        self.hWaitStop = win32event.CreateEvent(None, 0, 0, None)
        self.logger = self._getLogger()
        self.run = True  
 
    def _getLogger(self):  
        """
        日志模块
        """
        logger = logging.getLogger('[PythonService]')  
 
        this_file = inspect.getfile(inspect.currentframe())
        dirpath = os.path.abspath(os.path.dirname(this_file))
        handler = logging.FileHandler(os.path.join(dirpath, "service.log"))  
 
        formatter = logging.Formatter('%(asctime)s %(name)-12s %(levelname)-8s %(message)s')
        handler.setFormatter(formatter)  
 
        logger.addHandler(handler)
        logger.setLevel(logging.INFO)  
 
        return logger  
 
    def SvcDoRun(self):
        import time
        self.logger.info("service is run....")
        while self.run:
            self.logger.info("I am runing....")
            time.sleep(2)  
 
    def SvcStop(self):
        self.logger.info("service is stop....")
        self.ReportServiceStatus(win32service.SERVICE_STOP_PENDING)
        win32event.SetEvent(self.hWaitStop)
        self.run = False  
 
if __name__=='__main__':
    if len(sys.argv) == 1:
        try:
            evtsrc_dll = os.path.abspath(servicemanager.__file__)
            servicemanager.PrepareToHostSingle(PythonService)
            servicemanager.Initialize('PythonService', evtsrc_dll)
            servicemanager.StartServiceCtrlDispatcher()
        except win32service.error, details:
            if details[0] == winerror.ERROR_FAILED_SERVICE_CONTROLLER_CONNECT:
                win32serviceutil.usage()
    else:
        win32serviceutil.HandleCommandLine(PythonService)
```