### 安装 `Certbot` 的三种方式
[Certbot官网](https://certbot.eff.org/ "Certbot官网")

* 使用Certbot官方提供的对应平台的RPM包安装
* 使用Certbot官方的提供的certbot-auto安装
* 使用pip安装Certbot，因为Certbot是Python程序

### 前置条件
关闭`nginx`等服务器

### 安装 `certbot-auto`
```
    wget https://dl.eff.org/certbot-auto
```
### 赋予可执行权限
```
    sudo chmod a+x ./certbot-auto
```
### 运行certbot命令生成证书
```
    ./certbot-auto certonly --standalone --email 'vip@163.com' -d 'vip.cn'
```
### nginx 配置
```
server {
    listen 80;
    server_name vip.cn;
    rewrite ^(.*) https://vip.cn$1 permanent;
}
server {
     listen 443 ssl;
     server_name vip.cn;
     ssl_certificate  /etc/letsencrypt/live/vip.cn/fullchain.pem;
     ssl_certificate_key /etc/letsencrypt/live/vip.cn/privkey.pem;
     ssl_session_timeout 5m;
     ssl_protocols TLSv1 TLSv1.1 TLSv1.2;
     ssl_ciphers ECDHE-RSA-AES128-GCM-SHA256:HIGH:!aNULL:!MD5:!RC4:!DHE;
     ssl_prefer_server_ciphers on;
     root /data/vip/;
     index   index.php index.html index.htm;
     location ~ [^/]\.php(/|$)
     {
            fastcgi_pass unix:/dev/shm/php-cgi.sock;
            fastcgi_index index.php;
            include fastcgi.conf;

            fastcgi_split_path_info ^(.+?\.php)(/.*)$;
            set $path_info $fastcgi_path_info;
            fastcgi_param PATH_INFO       $path_info;
            try_files $fastcgi_script_name =404;
      }
      location / {
             if (!-e $request_filename){
                  rewrite ^(.*)$ /index.php/$1 last;
             }
      }
}
```
### 启动nginx访问即可
```
service nginx start
```
### 定时更新
```bash
#! /bin/bash
service nginx stop
/root/certbot/certbot-auto renew
service nginx start
```
```
sudo chmod a+x ./renew-cert.sh
```
crontab 
证书为三个月有效期,故每两个月更新一次
```
0 3 1 */2 * /root/renew-cert.sh >/root/crontab.log 2>&1
```