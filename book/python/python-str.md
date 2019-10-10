Python3 字符操作
====
#  字符串转16字节
```python
a = "01020C"
# 转为字节串
b = bytes().fromhex(a)
#  b = b'\x01\x02\x0c'
```