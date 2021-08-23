```
cd /usr/local/nginx/sbin/
./nginx  启动
./nginx -s stop  停止
./nginx -s quit  安全退出
./nginx -s reload  重新加载配置文件
ps aux|grep nginx  查看nginx进程
```

# nginx.conf

## 全局块

　　从配置文件开始到 events 块之间的内容，主要会设置一些影响nginx 服务器整体运行的配置指令，主要包括配置运行 Nginx 服务器的用户（组）、允许生成的 worker process 数，进程 PID 存放路径、日志存放路径和类型以及配置文件的引入等。

> worker_processes  1;

　　这是 Nginx 服务器并发处理服务的关键配置，worker_processes 值越大，可以支持的并发处理量也越多，但是会受到硬件、软件等设备的制约。

> #user nobody

Nginx用户及组：用户 组。window下不指定

> \#error_log logs/error.log;
> \#error_log logs/error.log notice;
> \#error_log logs/error.log info;

错误日志：存放路径。

> \#pid    logs/nginx.pid;

pid（进程标识符）：存放路径。

## events 块

> worker_connections  1024;

每个工作进程的最大连接数量。

## http块

> include    mime.types;

设定mime类型,类型由mime.type文件定义

> sendfile on;

sendfile指令指定 nginx 是否调用sendfile 函数（zero copy 方式）来输出文件，对于普通应用，必须设为on。如果用来进行下载等应用磁盘IO重负载应用，可设置为off，以平衡磁盘与网络IO处理速度，降低系统uptime。

>  keepalive_timeout 120;

keepalive超时时间。

> upstream bakend {
> server 127.0.0.1:8027;
> server 127.0.0.1:8028;
> server 127.0.0.1:8029;
> hash $request_uri;
> }

nginx的upstream目前支持4种方式的分配

1、轮询（默认）

每个请求按时间顺序逐一分配到不同的后端服务器，如果后端服务器down掉，能自动剔除。

2、weight
指定轮询几率，weight和访问比率成正比，用于后端服务器性能不均的情况。
例如：
upstream bakend {
server 192.168.0.14 weight=10;
server 192.168.0.15 weight=10;
}

2、ip_hash
每个请求按访问ip的hash结果分配，这样每个访客固定访问一个后端服务器，可以解决session的问题。
例如：
upstream bakend {
ip_hash;
server 192.168.0.14:88;
server 192.168.0.15:80;
}

3、fair（第三方）
按后端服务器的响应时间来分配请求，响应时间短的优先分配。
upstream backend {
server server1;
server server2;
fair;
}

4、url_hash（第三方）

按访问url的hash结果来分配请求，使每个url定向到同一个后端服务器，后端服务器为缓存时比较有效。

例：在upstream中加入hash语句，server语句中不能写入weight等其他的参数，hash_method是使用的hash算法

upstream bakend {
server squid1:3128;
server squid2:3128;
hash $request_uri;
hash_method crc32;
}

### server块

> listen    80;

配置监听端口

> server_name localhost;

配置访问域名

#### location块

> proxy_pass http://bakend