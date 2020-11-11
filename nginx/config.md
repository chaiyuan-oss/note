[Nginx](https://nginx.org/)

## nginx配置文件主要分为六个区域
* main（全局设置）
* events（nginx工作模式）
* http（http设置）
* sever（主机设置）
* location（URL匹配）
* upstream（负载均衡服务器设置

## main（全局设置）
```nginx
user nobody nobody;
worker_processes 2;
error_log /usr/local/var/log/nginx/error.log notice;
pid /usr/local/var/run/nginx/nginx.pid;
worker_rlimit_nofile 1024;
```
### user&pid
`user`和`pid`应该按默认设置 - 我们不会更改这些内容，因为更改与否没有什么不同。

### worker_processes
`worker_processes` 定义了`nginx`对外提供`web`服务时的`worker`进程数。最优值取决于许多因素，包括（但不限于）`CPU`核的数量、存储数据的硬盘数量及负载模式。不能确定的时候，将其设置为可用的`CPU`内核数将是一个好的开始（设置为`auto`将尝试自动检测它）。一般每个`Nginx`进程平均耗费`10M~12M`内存。在不使用`auto`时你可以根据经验，一般指定`1`个进程就足够了，如果是多核CPU，建议指定和CPU的数量一样的进程数即可。我这里写2，那么就会开启2个子进程，总共3个进程。

### worker_rlimit_nofile
`worker_rlimit_nofile` 更改`worker`进程的最大打开文件数限制。如果没设置的话，这个值为操作系统的限制。设置后你的操作系统和`Nginx`可以处理比`ulimit -a`更多的文件，所以把这个值设高，这样`nginx`就不会有`too many open files`问题了。

## events模块
events模块来用指定nginx的工作模式和工作模式及连接数上限，一般是这样：
```
events {
    use kqueue; #mac平台
    worker_connections 1024;
    multi_accept on;
}
```

### use
use用来指定Nginx的工作模式。Nginx支持的工作模式有select、poll、kqueue、epoll、rtsig和/dev/poll。其中select和poll都是标准的工作模式，kqueue和epoll是高效的工作模式，不同的是epoll用在Linux平台上，而kqueue用在BSD系统中，因为Mac基于BSD,所以Mac也得用这个模式，对于Linux系统，epoll工作模式是首选。

### worker_connections
worker_connections用于定义Nginx每个进程的最大连接数，即接收前端的最大请求数，默认是1024。最大客户端连接数由worker_processes和worker_connections决定，即Max_clients=worker_processesworker_connections，在作为反向代理时，Max_clients变为：Max_clients = worker_processes  worker_connections/4。

进程的最大连接数受Linux系统进程的最大打开文件数限制，在执行操作系统命令“ulimit -n 65536”后worker_connections的设置才能生效。

记住，最大客户数也由系统的可用socket连接数限制（~ 64K），所以设置不切实际的高没什么好处。

### multi_accept
multi_accept 告诉nginx收到一个新连接通知后接受尽可能多的连接。

## http模块
http模块可以说是最核心的模块了，它负责HTTP服务器相关属性的配置，它里面的server和upstream子模块，至关重要
```
http{
    include mime.types;
    default_type application/octet-stream;
    log_format main '$remote_addr - $remote_user [$time_local] "$request" '
    '$status $body_bytes_sent "$http_referer" '
    '"$http_user_agent" "$http_x_forwarded_for"';
    access_log /usr/local/var/log/nginx/access.log main;
    sendfile on;
    tcp_nopush on;
    tcp_nodelay on;
    keepalive_timeout 10;
    #gzip on;
    upstream myproject {
        .....
    }
    server {
        ....
    }
}
```

### include
include 用来设定文件的mime类型,类型在配置文件目录下的mime.type文件定义，来告诉nginx来识别文件类型

### default_type
default_type设定了默认的类型为二进制流，也就是当文件类型未定义时使用这种方式，例如在没有配置asp 的locate 环境时，Nginx是不予解析的，此时，用浏览器访问asp文件就会出现下载了

### log_format
log_format用于设置日志的格式，和记录哪些参数，这里设置为main，刚好用于access_log来记录这种类型。

main的类型日志如下：也可以增删部分参数
```
127.0.0.1 - - [21/Apr/2015:18:09:54 +0800] "GET /index.php HTTP/1.1" 200 87151 "-" "Mozilla/5.0 (Macintosh; Intel Mac OS X 10_10_2) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/41.0.2272.76 Safari/537.36"
```

### access_log
access_log 用来纪录每次的访问日志的文件地址，后面的main是日志的格式样式，对应于log_format的main。
access_log 设置nginx是否将存储访问日志。关闭这个选项可以让读取磁盘IO操作更快

### sendfile
sendfile 参数用于开启高效文件传输模式。将tcp_nopush和tcp_nodelay两个指令设置为on用于防止网络阻塞

sendfile 可以让sendfile()发挥作用。sendfile()可以在磁盘和TCP socket之间互相拷贝数据(或任意两个文件描述符)。Pre-sendfile是传送数据之前在用户空间申请数据缓冲区。之后用read()将数据从文件拷贝到这个缓冲区，write()将缓冲区数据写入网络。sendfile()是立即将数据从磁盘读到OS缓存。因为这种拷贝是在内核完成的，sendfile()要比组合read()和write()以及打开关闭丢弃缓冲更加有效(更多有关于sendfile)。

### keepalive_timeout
keepalive_timeout 设置客户端连接保持活动的超时时间。在超过这个时间之后，服务器会关闭该连接.我们将它设置低些可以让ngnix持续工作的时间更长。

### server_tokens
server_tokens  并不会让nginx执行的速度更快，但它可以关闭在错误页面中的nginx版本数字，这样对于安全性是有好处的。

### tcp_nopush
tcp_nopush 告诉nginx在一个数据包里发送所有头文件，而不一个接一个的发送。

### tcp_nodelay
tcp_nodelay 告诉nginx不要缓存数据，而是一段一段的发送--当需要及时发送数据时，就应该给应用设置这个属性，这样发送一小块数据信息时就不能立即得到返回值。

### client_header_timeout&client_body_timeout
client_header_timeout 和client_body_timeout 设置请求头和请求体(各自)的超时时间。我们也可以把这个设置低些。

### reset_timeout_connection
reset_timeout_connection 告诉nginx关闭不响应的客户端连接。这将会释放那个客户端所占有的内存空间。

### send_timeout
send_timeout 指定客户端的响应超时时间。这个设置不会用于整个转发器，而是在两次客户端读取操作之间。如果在这段时间内，客户端没有读取任何数据，nginx就会关闭连接。

### limit_conn_zone
limit_conn_zone 设置用于保存各种key（比如当前连接数）的共享内存的参数。5m就是5兆字节，这个值应该被设置的足够大以存储（32K*5）32byte状态或者（16K*5）64byte状态。

### limit_conn
limit_conn 为给定的key设置最大连接数。这里key是addr，我们设置的值是100，也就是说我们允许每一个IP地址最多同时打开有100个连接。

### gzip
gzip 是告诉nginx采用gzip压缩的形式发送数据。这将会减少我们发送的数据量。

### gzip_disable
gzip_disable 为指定的客户端禁用gzip功能。我们设置成IE6或者更低版本以使我们的方案能够广泛兼容。

### gzip_static
gzip_static 告诉nginx在压缩资源之前，先查找是否有预先gzip处理过的资源。这要求你预先压缩你的文件（在这个例子中被注释掉了），从而允许你使用最高压缩比，这样nginx就不用再压缩这些文件了（想要更详尽的gzip_static的信息，请点击这里）。

### gzip_proxied
gzip_proxied 允许或者禁止压缩基于请求和响应的响应流。我们设置为any，意味着将会压缩所有的请求。

### gzip_min_length
gzip_min_length 设置对数据启用压缩的最少字节数。如果一个请求小于1000字节，我们最好不要压缩它，因为压缩这些小的数据会降低处理此请求的所有进程的速度。

### gzip_comp_level
gzip_comp_level 设置数据的压缩等级。这个等级可以是1-9之间的任意数值，9是最慢但是压缩比最大的。我们设置为4，这是一个比较折中的设置。

### gzip_type
gzip_type 设置需要压缩的数据格式。上面例子中已经有一些了，你也可以再添加更多的格式。

### open_file_cache
open_file_cache 打开缓存的同时也指定了缓存最大数目，以及缓存的时间。我们可以设置一个相对高的最大时间，这样我们可以在它们不活动超过20秒后清除掉。

### open_file_cache_valid
open_file_cache_valid 在open_file_cache中指定检测正确信息的间隔时间。

### open_file_cache_min_uses
open_file_cache_min_uses 定义了open_file_cache中指令参数不活动时间期间里最小的文件数。

### open_file_cache_errors
open_file_cache_errors 指定了当搜索一个文件时是否缓存错误信息，也包括再次给配置中添加文件。我们也包括了服务器模块，这些是在不同文件中定义的。如果你的服务器模块不在这些位置，你就得修改这一行来指定正确的位置。

## server模块
sever 模块是http的子模块，它用来定一个虚拟主机
```
server {
    listen 80;
    server_name localhost 192.168.12.10 www.vip.com;
    # 全局定义，如果都是这一个目录，这样定义最简单。
    root /data/www;
    index index.php index.html index.htm; 
    charset utf-8;
    access_log usr/local/var/log/host.access.log main;
    aerror_log usr/local/var/log/host.error.log error;
    ....
}
```

### listen
listen用于指定虚拟主机的服务端口

### server_name
server_name用来指定IP地址或者域名，多个域名之间用空格分开。

### root
root 表示在这整个server虚拟主机内，全部的root web根目录。注意要和locate {}下面定义的区分开来。

### index
index 全局定义访问的默认首页地址。注意要和locate {}下面定义的区分开来。

### charset
charset用于设置网页的默认编码格式

## location模块
location模块是nginx中用的最多的，也是最重要的模块了，什么负载均衡啊、反向代理啊、虚拟域名啊都与它相关

location 根据它字面意思就知道是来定位的，定位URL，解析URL，所以，它也提供了强大的正则匹配功能，也支持条件判断匹配，用户可以通过location指令实现Nginx对动、静态网页进行过滤处理。

我们先来看这个，设定默认首页和虚拟机目录。
```
location / {
    root /data/www;
    index index.php index.html index.htm;
}
```

location /表示匹配访问根目录。

root指令用于指定访问根目录时，虚拟主机的web目录，这个目录可以是相对路径（相对路径是相对于nginx的安装目录）。也可以是绝对路径。

index用于设定我们只输入域名后访问的默认首页地址，有个先后顺序：index.php index.html index.htm，如果没有开启目录浏览权限，又找不到这些默认首页，就会报403错误。

location 还有一种方式就是正则匹配，开启正则匹配这样：location ~。后面加个~。

下面这个例子是运用正则匹配来链接php。我们之前搭建环境也是这样做：
```
location ~ \.php$ {
    root /data/www;
    fastcgi_pass 127.0.0.1:9000;
    fastcgi_index index.php;
    include fastcgi.conf;
}
```

.php$ 熟悉正则的我们直到，这是匹配.php结尾的URL，用来解析php文件。里面的root也是一样，用来表示虚拟主机的根目录。

fast_pass链接的是php-fpm 的地址

## upstream模块
upstream 模块负债负载均衡模块，通过一个简单的调度算法来实现客户端IP到后端服务器的负载均衡
```
upstream iyangyi.com{
    ip_hash;
    server 192.168.12.1:80;
    server 192.168.12.2:80 down;
    server 192.168.12.3:8080 max_fails=3 fail_timeout=20s;
    server 192.168.12.4:8080;
}
```

在上面的例子中，通过upstream指令指定了一个负载均衡器的名称iyangyi.com。这个名称可以任意指定，在后面需要的地方直接调用即可

里面是ip_hash这是其中的一种负载均衡调度算法，下面会着重介绍。紧接着就是各种服务器了。用server关键字表识，后面接ip。

Nginx的负载均衡模块目前支持4种调度算法:

weight 轮询（默认）。每个请求按时间顺序逐一分配到不同的后端服务器，如果后端某台服务器宕机，故障系统被自动剔除，使用户访问不受影响。weight。指定轮询权值，weight值越大，分配到的访问机率越高，主要用于后端每个服务器性能不均的情况下。

ip_hash。每个请求按访问IP的hash结果分配，这样来自同一个IP的访客固定访问一个后端服务器，有效解决了动态网页存在的session共享问题。

fair。比上面两个更加智能的负载均衡算法。此种算法可以依据页面大小和加载时间长短智能地进行负载均衡，也就是根据后端服务器的响应时间来分配请求，响应时间短的优先分配。Nginx本身是不支持fair的，如果需要使用这种调度算法，必须下载Nginx的upstream_fair模块。

url_hash。按访问url的hash结果来分配请求，使每个url定向到同一个后端服务器，可以进一步提高后端缓存服务器的效率。Nginx本身是不支持url_hash的，如果需要使用这种调度算法，必须安装Nginx 的hash软件包

在HTTP Upstream模块中，可以通过server指令指定后端服务器的IP地址和端口，同时还可以设定每个后端服务器在负载均衡调度中的状态。常用的状态有：

down，表示当前的server暂时不参与负载均衡。

backup，预留的备份机器。当其他所有的非backup机器出现故障或者忙的时候，才会请求backup机器，因此这台机器的压力最轻。

max_fails，允许请求失败的次数，默认为1。当超过最大次数时，返回proxy_next_upstream 模块定义的错误。

fail_timeout，在经历了max_fails次失败后，暂停服务的时间。max_fails可以和fail_timeout一起使用。

注意 当负载调度算法为ip_hash时，后端服务器在负载均衡调度中的状态不能是weight和backup。


[更多详细配置请点击](https://nginx.org/en/docs/)