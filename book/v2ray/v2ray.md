#  V2ray
#  参考
[V2ray官网](https://www.v2ray.com/)
[github项目地址](https://github.com/v2ray/v2ray-core)
[栋叔整理的博客](https://github.com/Waitung/Waitung.github.io/tree/master/docs/v2ray)

#  安装
需要切换到`root`用户下操作
```
bash <(curl -L -s https://install.direct/go.sh)
```

如果不行的话就试一下

```
wget https://install.direct/go.sh
bash go.sh
```

如果在执行一键脚本时下载v2ray失败（提示连接超时），请检查网络，换个运营商网络
#  配置文件
#  服务端
```shell
{
  "log": {
    "access": "/var/log/v2ray/access.log",
    "error": "/var/log/v2ray/error.log",
    "loglevel": "debug"
  },
  "inbounds": [{
    "port": 10001,
    "protocol": "vmess",
    "settings": {
      "clients": [
        {
          "id": "12345678-1234-1234-1234-123456789012",
          "level": 1,
          "alterId": 64
        }
      ]
    },
  "StreamSettings":{
    "network": "kcp",
    "kcpSettings":{
        "mtu": 1350,
        "tti": 50,
        "uplinkCapacity": 15,
        "downlinkCapacity": 100,
        "congestion": true,
        "readBufferSize": 5,
        "writeBufferSize": 5,
        "header": {
            "type": "wechat-video"
        }
    }
  },
    "tag": "clip"
  }],
  "outbounds": [{
    "protocol": "freedom",
    "settings": {}
      },
      {
    "tag": "blocked",
    "protocol": "blackhole",
    "settings": {}
    }],
  "routing": {
    "rules": [
      {
        "type": "field",
        "domain": ["geosite.dat:category-ads",
                   "geosite.dat:category-ads-all",
                   "geosite.dat:cn",
                   "geosite.dat:google",
                   "geosite.dat:facebook",
                   "geosite.dat:geolocation-cn",
                   "geosite.dat:geolocation-!cn",
                   "geosite.dat:speedtest",
                   "geosite.dat:tld-cn"
                   ],
        "ip": ["geoip:private"],
        "outboundTag": "blocked"
      }
    ]
  }
}

```
#  客户端配置
```shell
{
  "log": {
    "access": "",
    "error": "",
    "loglevel": "warning"
  },
  "inbound": {
    "port": 1080,
    "listen": "127.0.0.1",
    "protocol": "socks",
    "domainOverride": [
      "tls",
      "http"
    ],
    "settings": {
      "auth": "noauth",
      "udp": true,
      "ip": "127.0.0.1",
      "clients": null
    },
    "streamSettings": null
  },
  "outbound": {
    "tag": "agentout",
    "protocol": "vmess",
    "settings": {
      "vnext": [
        {
          "address": "123.123.123.123",
          "port": 10001,
          "users": [
            {
              "id": "12345678-1234-1234-1234-123456789012",
              "alterId": 64,
              "email": "t@t.tt",
              "security": "aes-128-gcm"
            }
          ]
        }
      ],
      "servers": null
    },
    "streamSettings": {
      "network": "kcp",
      "security": "",
      "tlsSettings": null,
      "tcpSettings": null,
      "kcpSettings": {
        "mtu": 1350,
        "tti": 50,
        "uplinkCapacity": 12,
        "downlinkCapacity": 100,
        "congestion": false,
        "readBufferSize": 2,
        "writeBufferSize": 2,
        "header": {
          "type": "wechat-video",
          "request": null,
          "response": null
        }
      },
      "wsSettings": null,
      "httpSettings": null
    },
    "mux": {
      "enabled": true
    }
  },
  "inboundDetour": null,
  "outboundDetour": [
    {
      "protocol": "freedom",
      "settings": {
        "response": null
      },
      "tag": "direct"
    },
    {
      "protocol": "blackhole",
      "settings": {
        "response": {
          "type": "http"
        }
      },
      "tag": "blockout"
    }
  ],
  "dns": {
    "servers": [
      "8.8.8.8",
      "8.8.4.4",
      "localhost"
    ]
  },
  "routing": {
    "strategy": "rules",
    "settings": {
      "domainStrategy": "IPIfNonMatch",
      "rules": [
        {
          "type": "field",
          "port": null,
          "outboundTag": "direct",
          "ip": [
            "0.0.0.0/8",
            "10.0.0.0/8",
            "100.64.0.0/10",
            "127.0.0.0/8",
            "169.254.0.0/16",
            "172.16.0.0/12",
            "192.0.0.0/24",
            "192.0.2.0/24",
            "192.168.0.0/16",
            "198.18.0.0/15",
            "198.51.100.0/24",
            "203.0.113.0/24",
            "::1/128",
            "fc00::/7",
            "fe80::/10"
          ],
          "domain": null
        }
      ]
    }
  }
}
```

> 特别注意
> 服务器和客户端时间差不能超过1分钟

#  配置文件说明

> "log": {}.

日志文件输入配置

> "inbounds":[]

数组，入站连接配置

> "outbounds":[]
 
数组，出站连接配置

> routing:{}
 
路由配置

#  `inbounds`
> port 

端口，接的格式如下：

- 整型数值: 实际的端口号。
- 环境变量: 以"env:"开头，后面是一个环境变量的名称，如"env:PORT"。V2Ray 会以字符串形式解析这个环境变量。
- 字符串: 可以是一个数值类型的字符串，如"1234"；或者一个数值范围，如"5-10"表示端口 5 到端口 10 这 6 个端口。

当只有一个端口时，V2Ray 会在此端口监听入站连接。当指定了一个端口范围时，取决于allocate设置。

> protocol
 
连接协议的名称，可选值有：

- Blackhole
- Dokodemo-door
- Freedom
- HTTP
- MTProto
- Shadowsocks
- Socks
- VMess

>  settings

具体协议的配置内容(本文只介绍Vmess)
>> clients 

VMess 用户的主 ID，linux下可使用以下命令生成

```
cat /proc/sys/kernel/random/uuid 
```
>> alterId

为了进一步防止被探测，一个用户可以在主 ID 的基础上，再额外生成多个 ID。这里只需要指定额外的 ID 的数量，推荐值为 4。不指定的话，默认值是 0。最大值 65535。这个值不能超过服务器端所指定的值。

>> level

用户等级
   
> StreamSettings

底层传输配置

>> network

数据流所使用的网络类型，默认值为 "tcp";
可选值有：

- tcp
- kcp 
- ws
- http
- domainsocket
- quic

>> kcpSettings

当前连接的 mKCP 配置，仅当此连接使用 mKCP 时有效。

- mtu: number

最大传输单元（maximum transmission unit），请选择一个介于 576 - 1460 之间的值。默认值为 1350。

- tti: number

传输时间间隔（transmission time interval），单位毫秒（ms），mKCP 将以这个时间频率发送数据。请选译一个介于 10 - 100 之间的值。默认值为 50。

- uplinkCapacity: number

上行链路容量，即主机发出数据所用的最大带宽，单位 MB/s，默认值 5。注意是 Byte 而非 bit。可以设置为 0，表示一个非常小的带宽。

- downlinkCapacity: number

下行链路容量，即主机接收数据所用的最大带宽，单位 MB/s，默认值 20。注意是 Byte 而非 bit。可以设置为 0，表示一个非常小的带宽。
- congestion: true | false

是否启用拥塞控制，默认值为 false。开启拥塞控制之后，V2Ray 会自动监测网络质量，当丢包严重时，会自动降低吞吐量；当网络畅通时，也会适当增加吞吐量。

- readBufferSize: number

单个连接的读取缓冲区大小，单位是 MB。默认值为 2。

- writeBufferSize: number

单个连接的写入缓冲区大小，单位是 MB。默认值为 2。

- header: HeaderObject

数据包头部伪装设置，可选值有：
|伪装类型|说明|
|---|----|
|"none"| 默认值，不进行伪装，发送的数据是没有特征的数据包。|
|"srtp"| 伪装成 SRTP 数据包，会被识别为视频通话数据（如 FaceTime）。|
|"utp"| 伪装成 uTP 数据包，会被识别为 BT 下载数据。|
|"wechat-video"| 伪装成微信视频通话的数据包。|
|"dtls"| 伪装成 DTLS 1.2 数据包。|
|"wireguard"| 伪装成 WireGuard 数据包。(并不是真正的 WireGuard 协议)|

> tag

此入站连接的标识，用于在其它的配置中定位此连接。当其不为空时，其值必须在所有tag中唯一。


#  开启端口
配置好之后要将服务器的端口打开才能正常使用

```
firewall-cmd --zone=public --add-port=8963/tcp --permanent
firewall-cmd --zone=public --add-port=8963/udp --permanent
重启端口
firewall-cmd --reload
查看端口
firewall-cmd --list-ports
```

> 注意：
> 各个供应商可能会使用外部防火墙，导致系统开启端口缺仍然不通。
> 需要根据各种服务商规则进行配置



#  管理V2ray

```
启动
systemctl start v2ray
停止
systemctl stop v2ray
重启
systemctl restart v2ray
查看状态
systemctl status v2ray
```



#  shadowsocksr
使用shadowsocksr来搭建梯子（简称ss），搭建ss梯子很方便，网上教程也
多，在服务器上也能够很简单的搭建好ss

我选用的服务器是centos，和ubuntu在linux上是不同分支，所以很多地方会
有所差别

首先，安装`wget`来下载东西

```
yum install wget -y
```

安装好之后使用wget下载ss的相关内容，具体是啥给忘了，总之下载就对了

```
wget -N --nocheck-certificate https://raw.githubusercontent.com/ToyoDAdoubi/doubi/master/ssr.sh&& chmod +x ssr.sh && bash ssr.sh
# 如果这个不能下载的话就试一下
wget -N --no-check-certificate https://softs.fun/Bash/ssr.sh&& chmod +x ssr.sh && bash ssr.sh
```

下载好之后就会自己安装好，根据提示操作就可以了

输入数字 1 开始安装

要求输入需要使用的端口，这个端口不要占用系统要用的端口就行了，
这边使用8963这个端口

之后会提示输入密码，这个密码会作为连接ss的密码

之后回要求选择加密方式，我选择了`chacha20`，使用这个端口的话需要安装
`libsodum`

 之后回要求选择协议插件，这个如果是要搭建更安全的ssr的话就要设置，但是
很多ss客户端不支持使用，所以用起来会有所问题。因为并没有什么特别机密的
东西，就是用来查资料所以就不管这个来，输入默认的选项就行`origin`

因为不是原版的，而是使用了网上改良版，会提示是否要安装兼容原版的插件
，全部选`no`就好啦

之后混淆插件，一样不需要，选择`plain`

之后就会开始安装ss，耐心等待就可以啦

完成安装之后可以使用锐速加速或则谷歌bbr，我使用的是bbr
# 搭建谷歌BBR加速

```
wget  --no-check-certificate https://github.com/teddysun/across/raw/master/bbr.sh
chmod +x bbr.sh
./bbr.sh
```

有提示需要就输入 `y`

最后重启服务器就可以来，`reboot`

