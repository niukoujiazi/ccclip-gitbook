
#  Ubuntu系统下安装使用MySQL
```
sudo apt-get update
sudo apt-get install mysql-server
```

之后可以运行安全脚本

```
mysql_secure_installation
```
                

大体提示为：

保護MySQL服務器部署。

使用空白密碼連接到MySQL。

VALIDATE PASSWORD PLUGIN可用於測試密碼
並提高安全性。它檢查密碼的強度
並允許用戶只設置那些密碼
足夠安全。你想設置VALIDATE PASSWORD插件嗎？

按y | Y表示是，否則為No的任何其他鍵：
LOW長度> = 8
MEDIUM長度> = 8，數字，大小寫混合和特殊字符
STRONG長度> = 8，數字，大小寫混寫，特殊字符和字典文件

請輸入0 = LOW，1 = MEDIUM，2 = STRONG：@ Jun819628
密碼的估計強度：100
您是否希望繼續提供密碼？（按y | Y表示是，其他任何鍵表示否）：
默認情況下，MySQL安裝有一個匿名用戶，
允許任何人登錄MySQL而不必擁有
為他們創建的用戶帳戶。這僅適用於
測試，並使安裝更順暢。
您應該在進入生產之前將其刪除
環境。

刪除匿名用戶？ （按y | Y表示是，任何其他鍵表示否）：
通常，只允許root連接
'localhost'的。這可以確保有人無法猜測
來自網絡的root密碼。

禁止遠程登錄？ （按y | Y表示是，任何其他鍵表示否）：
 ......跳過
默認情況下，MySQL附帶一個名為'test'的數據庫
任何人都可以訪問這也僅用於測試，
並且應該在進入生產之前將其移除
環境。


刪除測試數據庫並訪問它？ （按y | Y表示是，任何其他鍵表示否）：
  - 刪除測試數據庫......
成功。

  - 刪除測試數據庫的權限...
成功。

重新加載權限表將確保所有更改
到目前為止，將立即生效。

現在重新加載權限表？ （按y | Y表示是，任何其他鍵表示否）：


根据具体意愿设置就行了