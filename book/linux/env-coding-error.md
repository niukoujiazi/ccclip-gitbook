环境编码错误
=====

使用django时出现错误

```
 Traceback (most recent call last):
   File "/usr/local/lib/python3.6/dist-packages/django/core/handlers/exception.py", line 34, in inner
     response = get_response(request)
   File "/usr/local/lib/python3.6/dist-packages/django/utils/deprecation.py", line 94, in __call__
     response = response or self.get_response(request)
   File "/usr/local/lib/python3.6/dist-packages/django/core/handlers/exception.py", line 36, in inner
     response = response_for_exception(request, exc)
   File "/usr/local/lib/python3.6/dist-packages/django/core/handlers/exception.py", line 80, in response_for_exception
     response = debug.technical_500_response(request, *sys.exc_info(), status_code=400)
   File "/usr/local/lib/python3.6/dist-packages/django/views/debug.py", line 94, in technical_500_response
     html = reporter.get_traceback_html()
   File "/usr/local/lib/python3.6/dist-packages/django/views/debug.py", line 332, in get_traceback_html
     t = DEBUG_ENGINE.from_string(fh.read())
   File "/usr/lib/python3.6/encodings/ascii.py", line 26, in decode
     return codecs.ascii_decode(input, self.errors)[0]
 UnicodeDecodeError: 'ascii' codec can't decode byte 0xe2 in position 9735: ordinal not in range(128)
```
环境编码问题，解决方法就是修改环境变量，将编码改为`utf-8`

```
$ export LC_ALL=en_US.UTF-8
$ export LANG=en_US.UTF-8
$ export LANGUAGE=en_US.UTF-8
```