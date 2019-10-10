Harbor
====
# 参考
[HARBOR](https://goharbor.io/# getting-started)

[Installation and Configuration Guide](https://github.com/goharbor/harbor/blob/master/docs/installation_guide.md)

[Configuring Harbor with HTTPS Access](https://github.com/goharbor/harbor/blob/master/docs/configure_https.md)

[User Guide](https://github.com/goharbor/harbor/blob/master/docs/user_guide.md)

[Docker 企业级私有镜像仓库 Harbor 部署](https://jaminzhang.github.io/docker/Enterprise-class-private-Docker-Container-Registry-Harbor-deploying/)

[私有仓库高级配置](https://yeasy.gitbooks.io/docker_practice/repository/registry_auth.html)

[CentOS7搭建Harbor镜像仓库及https处理](https://blog.csdn.net/bacteriumX/article/details/88326722)
# 安装Harbor
# 在线安装
$ wget https://storage.googleapis.com/harbor-releases/release-1.7.0/harbor-online-installer-v1.7.4.tgz
$ tar xvf harbor-online-installer-v1.7.4.tgz


# 离线安装

$ wget https://storage.googleapis.com/harbor-releases/release-1.7.0/harbor-offline-installer-v1.7.4.tgz
$ tar xvf harbor-offline-installer-v1.7.4.tgz


# Openssl自行签发证书
假设需要签署证书的域名为`yourdoain.com`

正式部署请务必将其替换成自己的域名
# 获得证书授权
 openssl genrsa -out ca.key 4096
  openssl req -x509 -new -nodes -sha512 -days 3650 \
    -subj "/C=TW/ST=Taipei/L=Taipei/O=example/OU=Personal/CN=yourdomain.com" \
    -key ca.key \
    -out ca.crt

# 获得服务器证书

# 1) 创建自己的私钥

 openssl genrsa -out yourdomain.com.key 4096

# 2) 生成证书签名请求

 openssl req -sha512 -new \
    -subj "/C=TW/ST=Taipei/L=Taipei/O=example/OU=Personal/CN=yourdomain.com" \
    -key yourdomain.com.key \
    -out yourdomain.com.csr 

# 3) 生成注册表主机的证书

cat > v3.ext <<-EOF
authorityKeyIdentifier=keyid,issuer
basicConstraints=CA:FALSE
keyUsage = digitalSignature, nonRepudiation, keyEncipherment, dataEncipherment
extendedKeyUsage = serverAuth 
subjectAltName = @alt_names

[alt_names]
DNS.1=yourdomain.com
DNS.2=yourdomain
DNS.3=hostname
EOF
操作这步之后当前目录下会出现`v3.ext`文件，上面语法中的`cat > v3.ext <<-EOF`
可以参考[shell](http://www.ccclip.com/story/336)
# 配置和安装
# 1) 配置服务器证书和Harbor秘钥
将上面步骤得到的`yourdomain.com.crt`和`yourdomain.com.key`文件复制到
`/data/cert/`,用于Harbor配置文件找到证书的路径

  cp yourdomain.com.crt /data/cert/
  cp yourdomain.com.key /data/cert/ 

# 3) 为docker配置服务器证书，秘钥和CA

Docker守护程序将.crt文件解释为CA证书，将.cert文件解释为客户端证书。

将服务器转换yourdomain.com.crt为yourdomain.com.cert：


openssl x509 -inform PEM -in yourdomain.com.crt -out yourdomain.com.cert
Delpoy yourdomain.com.cert，yourdomain.com.key和ca.crtDocker：

  cp yourdomain.com.cert /etc/docker/certs.d/yourdomain.com/
  cp yourdomain.com.key /etc/docker/certs.d/yourdomain.com/
  cp ca.crt /etc/docker/certs.d/yourdomain.com/
以下说明了使用自定义证书的配置：

/etc/docker/certs.d/
    └── yourdomain.com:port   
       ├── yourdomain.com.cert  <-- Server certificate signed by CA
       ├── yourdomain.com.key   <-- Server key signed by CA
       └── ca.crt               <-- Certificate authority that signed the registry certificate

# 3) 配置Harbor

编辑文件`harbor.cfg`，更新主机名和协议，并更新`ssl_cert`和`ssl_cert_key`:
  # set hostname
  hostname = yourdomain.com:port
  # set ui_url_protocol
  ui_url_protocol = https
  ......
  # The path of cert and key files for nginx, they are applied only the protocol is set to https 
  ssl_cert = /data/cert/yourdomain.com.crt
  ssl_cert_key = /data/cert/yourdomain.com.key

为Harbor生成配置文件：

  ./prepare
如果Harbor已在运行，请停止并删除现有实例。您的图像数据保留在文件系统中

  docker-compose down -v
最后，重启港湾：

  docker-compose up -d

为Harbor设置HTTPS后，您可以通过以下步骤进行验证：

打开浏览器并输入地址：https：//yourdomain.com。它应该显示Harbor的用户界面。

请注意，即使我们通过自签名CA签署证书并将CA部署到上述位置，某些浏览器仍可能出于安全原因显示有关证书颁发机构（CA）未知的警告。这是因为自签名CA本质上不是受信任的第三方CA. 您可以自己将CA导入浏览器以解决警告。

在具有Docker守护程序的计算机上，请确保https://yourdomain.com的选项“-insecure-registry” 不存在。

如果您将nginx端口443映射到另一个端口，那么您应该创建目录/etc/docker/certs.d/yourdomain.com:port（或您的注册表主机IP：端口）。然后运行任何docker命令来验证设置，例如

  docker login yourdomain.com
如果您已将nginx 443端口映射到另一个端口，则需要将端口添加到登录，如下所示：

  docker login yourdomain.com:port

# 客户端
如果想在客户端拉取`openssl`自签署证书的仓库时可能会遇到

```
Error response from daemon: Get https://youserver.com/v2/: x509: certificate signed by unknown authority
```
即证书未受信任，解决方法有两个

# 1) 手动信任该网站
修改`/etc/docker/daemon.json`文件将域名加入

```
{
    "registry-mirror": ["https://registry.docker-cn.com"],
    "insecure-registries": ["youserver.com"]
}
```

# 2) 在客户端安装证书
```
cp yourdomain.com.crt /usr/local/share/ca-certificates/yourdomain.com.crt
update-ca-certificates
```
如果还是不行，则修改hosts文件，将域名指向IP

```
$ echo '0.0.0.0    youserver.com' >> /etc/hosts
```
