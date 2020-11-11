## OCSP stapling
OCSP stapling是Https优化方案之一，将原本需要客户端实时发起的 OCSP 请求转嫁给服务端；

### OCSP 简介
在线证书状态协议（Online Certificate Status Protocol），简称 OCSP，是一个用于获取 X.509 数字证书撤销状态的网际协议，在 RFC 6960 中定义。OCSP 用于检验证书合法性，查询服务一般由证书所属 CA 提供。OCSP 查询的本质，是一次完整的 HTTP 请求加响应的过程，这中间涵括的 DNS 查询、建立 TCP 连接、Web 端工作等步骤，都将耗费更多时间，使得建立 TLS 花费更多时长。

### OCSP 弊端
OCSP存在隐私和性能问题。
1、浏览器直接去请求第三方CA(Certificate Authority, 数字证书认证机构)，会暴露网站的访客(CA 机构会知道哪些用户在访问我们的网站)；
2、浏览器进行OCSP查询会降低HTTPS性能(访问我们的网站会变慢) OCSP实时查询会增加客户端的性能开销。

后来，OCSP Stapling 出现了。将原本需要客户端实时发起的 OCSP 请求转嫁给服务端，Web 端将主动获取 OCSP 查询结果，并随证书一起发送给客户端，以此让客户端跳过自己去寻求验证的过程，提高 TLS 握手效率。 可以提高HTTPS性能。

### 在线校验
此方式需要支持服务器能够主动访问证书校验服务器才能生效，并且在每次重启nginx的时候会主动请求一次，如果网络不通会导致nginx启动缓慢。


```
server {
    listen 443 ssl;
    server_name  xx.xx.com;
    index index.html index.htm index.jsp;

    ssl_certificate         server.pem;#证书的.cer文件路径
    ssl_certificate_key     server-key.pem;#证书的.key文件

    # 开启 OCSP Stapling ---当客户端访问时 NginX 将去指定的证书中查找 OCSP 服务的地址，
获得响应内容后通过证书链下发给客户端。
    ssl_stapling on;
    ssl_stapling_verify on;# 启用OCSP响应验证，OCSP信息响应适用的证书
    ssl_trusted_certificate /path/to/xxx.pem;#若 ssl_certificate 指令指定了完整的证书链，则 ssl_trusted_certificate 可省略。
    resolver 8.8.8.8 8.8.4.4 216.146.35.35 216.146.36.36 valid=60s;#添加resolver解析OSCP响应服务器的主机名，valid表示缓存。
    resolver_timeout 2s；# resolver_timeout表示网络超时时间
```     

### 人工更新
为了缓存的更新时间更可加控，你也可以人工负责更新文件内容。利用 NginX 的 ssl_stapling_file 指令直接将 OCSP 响应存成文件，NginX 从文件获取OCSP响应而无需从服务商拉取，将其随证书下发而不实时查询。

```
server {
    listen 443 ssl;
    server_name  xx.xx.com;
    index index.html index.htm index.jsp;

    ssl_certificate         server.pem;#证书的.cer文件路径
    ssl_certificate_key     server-key.pem;#证书的.key文件

    # 开启 OCSP Stapling ---当客户端访问时 NginX 将去指定的证书中查找 OCSP 服务的地址，
获得响应内容后通过证书链下发给客户端。
    ssl_stapling on;
    ssl_stapling_file /xxx/xxx/stapling_file.ocsp; 
    ssl_stapling_verify on;# 启用OCSP响应验证，OCSP信息响应适用的证书
    ssl_trusted_certificate /path/to/xxx.pem;#若 ssl_certificate 指令指定了完整的证书链，则 ssl_trusted_certificate 可省略。
```

### OCSP Stapling 将吊销信息缓存在服务端这不是和 CRL 具有相同的缺点了么？
OCSP stapling is a technology to check revocation status of X.509 digital certificates. If anyone has confusion regarding it then you can check this resource https://www.clickssl.net/blog/ocsp-stapling-check-your-certificate-revocation


[美图HTTPS优化探索与实践](https://blog.csdn.net/weixin_34050005/article/details/86718496)
[Nginx之OCSP stapling配置](https://www.jianshu.com/p/540124f370e0)