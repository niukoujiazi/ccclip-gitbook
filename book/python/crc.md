循環冗餘校驗(CRC)
=====

#  关于CRC校验
参考：
 
[循環冗餘校驗](https://zh.wikipedia.org/wiki/%E5%BE%AA%E7%92%B0%E5%86%97%E9%A4%98%E6%A0%A1%E9%A9%97)
 
[二进制数的运算方法](https://www.jianshu.com/p/560aba49c9a4)
 
[模除](https://zh.wikipedia.org/wiki/%E6%A8%A1%E9%99%A4)
 
[循环冗余校验（CRC）算法入门引导](https://blog.csdn.net/liyuanbhu/article/details/7882789)
 
[python CRC16检验](https://blog.csdn.net/TGL233/article/details/72487429)

#  官方模块
 
[crccheck 0.6](https://pypi.org/project/crccheck/)
 
在线CRC工具
 
[CRC（循环冗余校验）在线计算](http://www.ip33.com/crc.html)

#  实际使用
按照官方文档上，只需要按照类名引入模块就可使用
在在线CRC校验工具上检测，确认了项目需要的是

`CRC-16/25`       `x16+x12+x5+1`

```
from crccheck.crc import CrcX25
crcx = CrcX25()
data1= bytes.fromhex('11010752533678900242700032010005')
crcx.process(data1)
s = crcx.finalhex()

```

最终得到结果
 
`>>`
 
`'1279'`
 
这个结果和在线工具上测试的相同，暂且能用，达到了目的，所以暂告一段落

#  顺便贴上官方CRC类的类名和使用说明

Project description
The crccheck.crc module implements all CRCs listed in the Catalogue of parametrised CRC algorithms:

CRC-3/ROHC, CRC-4/ITU, CRC-5/EPC, CRC-5/ITU, CRC-5/USB, CRC-6/CDMA2000-A, CRC-6/CDMA2000-B, CRC-6/DARC, CRC-6/ITU, CRC-7, CRC-7/ROHC, CRC-8, CRC-8/CDMA2000, CRC-8/DARC, CRC-8/DVB-S2, CRC-8/EBU, CRC-8/I-CODE, CRC-8/ITU, CRC-8/MAXIM, CRC-8/ROHC, CRC-8/WCDMA, CRC-10, CRC-10/CDMA2000, CRC-11, CRC-12/3GPP, CRC-12/CDMA2000, CRC-12/DECT, CRC-13/BBC, CRC-14/DARC, CRC-15, CRC-15/MPT1327, CRC-16, ARC, CRC-16/AUG-CCITT, CRC-16/BUYPASS, CRC-16/CCITT-FALSE, CRC-16/CDMA2000, CRC-16/DDS-110, CRC-16/DECT-R, CRC-16/DECT-X, CRC-16/DNP, CRC-16/EN-13757, CRC-16/GENIBUS, CRC-16/MAXIM, CRC-16/MCRF4XX, CRC-16/RIELLO, CRC-16/T10-DIF, CRC-16/TELEDISK, CRC-16/TMS37157, CRC-16/USB, CRC-A, CRC-16 CCITT, KERMIT, MODBUS, X-25, XMODEM, CRC-24, CRC-24/FLEXRAY-A, CRC-24/FLEXRAY-B, CRC-31/PHILIPS, CRC-32, CRC-32/BZIP2, CRC-32C, CRC-32D, CRC-32/MPEG-2, CRC-32/POSIX, CRC-32Q, JAMCRC, XFER, CRC-40/GSM, CRC-64, CRC-64/WE, CRC-64/XZ, CRC-82/DARC.
For the class names simply remove all dashes and slashes from the above names and apply CamelCase, e.g. “CRC-32/MPEG-2” is implemented by Crc32Mpeg2. Other CRC can be calculated by using the general classcrccheck.crc.Crc by providing all required CRC parameters.



The crccheck.checksum module implements additive and XOR checksums with 8, 16 and 32 bit: Checksum8, Checksum16, Checksum32 and ChecksumXor8, ChecksumXor16, ChecksumXor32

Usage example:

from crccheck.crc import Crc32, CrcXmodem
from crccheck.checksum import Checksum32

#  Quick calculation
data = bytearray.fromhex("DEADBEEF")
crc = Crc32.calc(data)
checksum = Checksum32.calc(data)

#  Procsss multiple data buffers
data1 = b"Binary string"  #  or use .encode(..) on normal sring - Python 3 only
data2 = bytes.fromhex("1234567890")  #  Python 3 only, use bytearray for older versions
data3 = (0x0, 255, 12, 99)  #  Iterable which returns ints in byte range (0..255)
crcinst = CrcXmodem()
crcinst.process(data1)
crcinst.process(data2)
crcinst.process(data3[1:-1])
crcbytes = crcinst.finalbytes()
crchex = crcinst.finalhex()
crcint = crcinst.final()