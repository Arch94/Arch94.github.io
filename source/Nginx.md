# Nginx

## 目录

-   [nginx 基础命令](#nginx-基础命令)
-   [nginx location 路由规则](#nginx-location-路由规则)
    -   [demo 后缀名匹配](#demo-后缀名匹配)
    -   [demo2匹配规则](#demo2匹配规则)
    -   [demo3匹配顺序](#demo3匹配顺序)
    -   [demo4无符号匹配](#demo4无符号匹配)
-   [nginx编译安装](#nginx编译安装)
-   [如何使用命令查看一次http请求](#如何使用命令查看一次http请求)
-   [nginx 日志](#nginx-日志)
    -   [error.log](#errorlog)
    -   [access.log](#accesslog)
    -   [按日期生成访问日志](#按日期生成访问日志)
-   [nginx模块配置](#nginx模块配置)
    -   [查看nginx已安装模块](#查看nginx已安装模块)
    -   [查看nginx可以安装的包](#查看nginx可以安装的包)
-   [基于nginx的中间件架构](#基于nginx的中间件架构)
    -   [静态资源web服务](#静态资源web服务)
    -   [mime.type 资源的媒体类型](#mimetype-资源的媒体类型)
    -   [sendfile 配置](#sendfile-配置)
    -   [gzip压缩](#gzip压缩)
    -   [http缓存](#http缓存)
    -   [expires（http1.0）](#expireshttp10)
    -   [expires（http1.1）](#expireshttp11)
    -   [nginx 跨域访问](#nginx-跨域访问)
    -   [nginx 防盗链](#nginx-防盗链)
-   [nginx 代理](#nginx-代理)
    -   [发向代理/正向代理](#发向代理正向代理)
    -   [常用代理的配置](#常用代理的配置)
    -   [proxy\_params](#proxy_params)
    -   [nginx 轮询](#nginx-轮询)
-   [nginx 变量解析](#nginx-变量解析)
-   [nginx缓存（代理服务器端缓存，减少后端压力，提高网站并发延时）](#nginx缓存代理服务器端缓存减少后端压力提高网站并发延时)
    -   [部分页面不缓存](#部分页面不缓存)

#### nginx 基础命令

```bash
nginx -t // 检查配置文件 查看当前nginx 配置路径
nginx -s reload // 重载nginx配置文件
```

### nginx location 路由规则

```nginx
location [=|~|~*|^~|@] pattern { ... }
```

```nginx
= : 表示精确匹配后面的url
~ : 表示正则匹配，但是区分大小写
~* : 正则匹配，不区分大小写
^~ : 表示普通字符匹配，如果该选项匹配，只匹配该选项，不匹配别的选项，一般用来匹配目录
@ : "@" 定义一个命名的 location，使用在内部定向时，例如 error_page
```

#### demo 后缀名匹配

```nginx
location ~ .*\.(html|htm|gif|jpg|jpeg|bmp|png|ico|txt|js|css|woff|ttf|map|woff2|json|manifest|svg|eot)$  {   
  root /Users/sunqixiong/project/crm-cerm/ecrm-mobile/www;
}
```

就是正则匹配,其中`.*\.(html|htm|gif|jpg|jpeg|bmp|png|ico|txt|js|css|woff|ttf|map|woff2|json|manifest|svg|eot)$`就是一串匹配后缀名的正则表达式

#### demo2匹配规则

```nginx
location = /world {
  return 600;
}

location = /hello {
  return 600;
}

location ~ /hellowo {
  return 602;
}

location ^~ /hello {
  return 601;
}
```

```text
- 请求 localhost/world 返回600
- 请求 localhost/world2 localhost/test/world 返回其他
- 请求 localhost/hello  返回600
- 请求 localhost/hello/123 返回601
- 请求 localhost/hellow 返回601
- 请求 localhost/hellowo 返回601
- 请求 localhost/test/hellowo  返回602
- 请求 localhost/test/hello 返回其他
```

-   \= 是精确完整匹配, 且优秀最高
-   匹配时，如果 \~ 和 ^\~ 同时匹配规则，则 ^\~ 优先
-   ^\~ 这个不会匹配请求url中后面的路径, 如上面的 /test/hello 没有匹配上
-   ^\~ 不支持正则，和=相比，范围更广， hellowo 是可以被^\~匹配，但是 = 不会匹配
-   \~ 路径中只要包含就可以匹配，如上面的 /test/hellowo 返回了602

#### demo3匹配顺序

```nginx
location ~ /hello {
  return 602;
}

location ~ /helloworld {
  return 601;
}
```

```text
- 请求 localhost/world/helloworld 返回 602
- 请求 localhost/helloworld 返回 602
```

```nginx
location ~ /helloworld {
  return 601;
}

location ~ /hello {
  return 602;
}
```

```text
- 请求 localhost/helloworld 返回601
- 请求 localhost/world/helloworld 返回601
- 请求 localhost/helloWorld 返回602
```

所以同时正则匹配时

-   放在前面的优先匹配
-   注意如果不区分大小写时，使用\~\*
-   尽量将精确匹配的放在前面

#### demo4无符号匹配

```nginx
location ^~ /hello/ {
  return 601;
}

location /hello/world {
  return 602;
}
```

```text
- http://localhost/hello/wor 返回601
- http://localhost/hello/world 返回602
- http://localhost/hello/world23 返回602
- http://localhost/hello/world/123 返回602
```

-   没有符号时，全匹配是优先于^\~的

### nginx编译安装

首先下载依赖包

nginx，penSSL、PCRE、zlib

```bash
mv openssl-1.1.0g pcre-8.41 zlib-1.2.11 /usr/local/bin
```

执行配置命令，几个依赖包的路径对就可以，官方文档提示要写到一行

```bash
./configure --with-http_ssl_module  --with-http_stub_status_module --with-http_gzip_static_module  --with-pcre=../pcre-8.00 --with-zlib=../zlib-1.2.11 --with-openssl=../openssl-1.1.0k
```

编译

```bash
make
```

之后安装

```bash
sudo make install
```

配置nginx 环境变量

```bash
ln -s /usr/local/nginx/sbin/nginx /usr/local/bin/nginx(做软链，添加到环境变量)
```

然后就可以执行 `nginx -v` 看依赖了

### 如何使用命令查看一次http请求

```bash
curl -v http://www.baidu.com >/dev/null
```

### nginx 日志

#### error.log

错误日志的存放位置 级别，放在最外面

```nginx
error_log /usr/local/etc/nginx/log/error.log warn;
```

#### access.log

首先第一条 log\_format要放在 http之下，其次后面的main 指的是 log\_format对应的名字

```nginx
log_format  main  '$remote_addr - $remote_user [$time_local] "$request" '
                  '$status $body_bytes_sent "$http_referer" '
                  '"$http_user_agent" "$http_x_forwarded_for"';
```

```nginx
access_log  /var/log/nginx/access.log  main;
```

#### 按日期生成访问日志

```nginx
#此处是本地开发所需，如是容器，则无需owner
user root owner;
http{
    log_format  main  '$remote_addr - $remote_user [$time_local] "$request" '
                      'zhuang tai m$status $body_bytes_sent "$http_referer" '
                      '"$http_user_agent" "$http_x_forwarded_for"';
    ...
    server{
       if ($time_iso8601 ~ "^(\d{4})-(\d{2})-(\d{2})") {
            set $year $1;
            set $month $2;
            set $day $3;
        }

        access_log  /usr/local/etc/nginx/log/access_$year-$month-$day.log  main;
   }
}
```

有个缺点，访问日志和错误日志生成的代码只能放在server下面

还有其他的方法如编写定时sh脚本

### nginx模块配置

nginx编译安装以后想要再加nginx模块只能重新配置再编译安装。

#### 查看nginx已安装模块

nginx 大V看模块 小v看版本

```bash
nginx -V
```

#### 查看nginx可以安装的包

进入nginx 编译安装前的包的目录

```bash
cat auto/options | grep YES
```

模块介绍

1.  连接频率限制

`ngx_http_limit_conn_module`

在nginx配置文件中的(http, server, location) 下配置

```nginx
http {
  # ...其它代码省略...
  # 开辟一个10m的连接空间，命名为addr
  limit_conn_zone $binary_remote_addr zone=addr:10m;
  server {
    ...
    location /download/ {
      # 服务器每次只允许一个IP地址连接
      limit_conn addr 1;
    }
  }
}
```

1.  请求频率限制

`ngx_http_limit_req_module`

在nginx配置文件中的(http, server, location) 下配置

```nginx
http {
 
  # ...其它代码省略...
   
  # 开辟一个10m的请求空间，命名为one。同一个IP发送的请求，平均每秒只处理一次
  limit_req_zone $binary_remote_addr zone=one:10m rate=1r/s;
   
  server {
      ...
 
    location /search/ {
      limit_req zone=one;
      # 当客户端请求超过指定次数，最多宽限5次请求，并延迟处理，1秒1个请求
      # limit_req zone=one burst=5;
      # 当客户端请求超过指定次数，最多宽限5次请求，并立即处理。
      # limit_req zone=one burst=5 nodelay;
 
    }
  }
}
```

1.  基于IP的访问控制

`ngx_http_access_module`

```bash
Syntax:    allow address | CIDR | unix: | all;
Default:  —
Context:  http, server, location, limit_except
 
Syntax:    deny address | CIDR | unix: | all;
Default:  —
Context:  http, server, location, limit_except
```

```nginx
server {
  # ...其它代码省略...
  location ~ ^/index_1.html {
    root  /usr/share/nginx/html;
    deny 151.19.57.60; # 拒绝这个IP访问
    allow all; # 允许其他所有IP访问
  }
 
  location ~ ^/index_2.html {
    root  /usr/share/nginx/html;
    allow 151.19.57.0/24; # 允许IP 151.19.57.* 访问
    deny all; # 拒绝其他所有IP访问
  }
}
```

ngx\_http\_access\_module 的局限性

当客户端通过代理访问时，nginx的remote\_addr获取的是代理的IP

### 基于nginx的中间件架构

#### 静态资源web服务

#### mime.type 资源的媒体类型

这玩意看起来不显眼，但是如果没了它，文件资源下载了浏览器却没法识别

```nginx
include     /etc/nginx/mime.types;
```

#### sendfile 配置

```text
语法： sendfile on | off;
默认值： sendfile off;
上下文： http，server，location，if in location
```

```nginx
http ｛
    # other directives
    sendfile    on;
    # other directives
｝
```

指定是否使用sendfile系统调用来传输文件。

sendfile系统调用在两个文件描述符之间直接传递数据(完全在内核中操作)，从而避免了数据在内核缓冲区和用户缓冲区之间的拷贝，操作效率很高，被称之为零拷贝。

所以当 Nginx 是一个静态文件服务器的时候，开启 SENDFILE 配置项能大大提高 Nginx 的性能。 但是当 Nginx 是作为一个反向代理来使用的时候，SENDFILE 则没什么用了

#### gzip压缩

注：开启 gzip\_static on 指令需要 http\_gzip\_static\_module 模块

```nginx
http {
    gzip on; 
    gzip_static on; # Nginx的动态压缩是对每个请求先压缩再输出，这样造成虚拟机浪费了很多cpu，解决这个问题可以利用nginx模块Gzip Precompression，这个模块的作用是对于需要压缩的文件，直接读取已经压缩好的文件(文件名为加.gz)，而不是动态压缩，对于不支持gzip的请求则读取原文件
    gzip_buffers 4 16k;
    gzip_comp_level 5;
    gzip_types text/plain application/javascript text/css application/xml text/javascript application/x-httpd-php image/jpeg image/gif image/png;
}
```

#### http缓存

1.  先校验是否过期 Expires\[http1.0]、Cache-Control(max-age)\[http1.1]
2.  再校验协议中etag头信息校验
3.  再校验Last-Modified 头信息校验

注意：
有时候请求到的缓存文件，明明请求头没有加 `cache-control`，但请求头会有 `Cache-Control: max-age=0`经测试发现chrome会对html、png等格式自带，而对css、js文件不自带，这个机制很好，这是因为浏览器会自带这个请求头让服务器去检查这个文件的etag或者Last-Modified是否过期

#### expires（http1.0）

```nginx
location ~ .*\.(jpg|png)$ {
    expires 24h;
    root /Users/sunqixiong/Pictures;
}
```

第一次请求就会返回 response

```nginx
Expires: Thu, 15 Aug 2019 07:47:54 GMT
Cache-Control: max-age=86400
```

第二次请求返回304，返回 response 除了一样以外，它的request header 还是带上了 `Cache-Control: max-age=0`，文件来源还是other，并没有从cache  memory 来，因为它是图片chrome自带Cache-Control:max-age=0

#### expires（http1.1）

```nginx
location /zjmobile {
    if ($request_filename ~ .*\.(htm|html)$)
    {
        add_header Cache-Control no-cache;
    }
    if ($request_filename ~ .*\.(js|css|png|jpg|ttf|eot|svg|jpeg|woff)$)
    {
        add_header  Cache-Control  max-age=86400;
    }
    alias   /app/app/enrollment;
    index  index.html;
    try_files $uri $uri/ /index.html last;
}
```

#### nginx 跨域访问

```nginx
#设置允许来跨域访问的网站
add_header Access-Control-Allow-Origin *;
#add_header Access-Control-Allow-Origin http://www.jsonc.com;
add_header Access-Control-Allow-Methods GET,POST,PUT,DELETE,OPTIONS
```

#### nginx 防盗链

http\_refer 上一级页面的地址

```nginx
location / {
    valid_referers none blocked 127.0.0.1;
    if($invalid_referer){
        return 403;
    }
}
```

测试防盗链接

```bash
curl -e "http://www.baidu.com" -I http://localhost:7676/refer
curl -e 127.0.0.1 -I http://localhost:7676/refer
```

### nginx 代理

#### 发向代理/正向代理

```nginx
#发向代理
location /zjmobile/service {
    proxy_pass http://tomcat_pool/zjcm/service
}
#正向代理
location / {
    proxy_pass http://$http_host$request_uri
}
```

#### 常用代理的配置

```nginx
location / {
    proxy_pass http://127.0.0.1:8080;
    include proxy_params;
}
```

#### proxy\_params

注意:proxy buffer这一块具体数值，看服务器数值决定。虽然可以通用，但如果不熟悉，还是不建议使用。

```nginx
proxy_redirect default;
proxy_set_header HOST $http_host;
proxy_set_header X-Real-IP $remote_addr; #带上用户真实的ip

proxy_connect_timeout 30;

proxy_buffering on;#是否开启对后端response的缓冲
proxy_buffer_size 32k; #缓存response的第一部分,通常是header,默认proxy_buffer_size 被设置成 proxy_buffers 里一个buffer 的大小，当然可以设置更小些。
proxy_buffers 4 128k;#前面一个是num,后面一个是每个buffer的size.Nginx将会尽可能的读取后端服务器的数据到buffer，直到proxy_buffers设置的所有buffer们被写满或者数据被读取完(EOF)，此时Nginx开始向客户端传输数据，会同时传输这一整串buffer们。如果数据很大的话，Nginx会接收并把他们写入到temp_file里去，大小由proxy_max_temp_file_size 控制。
proxy_busy_buffers_size 256k;#一旦proxy_buffers设置的buffer被写入，直到buffer里面的数据被完整的传输完（传输到客户端），这个buffer将会一直处 在busy状态，我们不能对这个buffer进行任何别的操作。所有处在busy状态的buffer size加起来不能超过proxy_busy_buffers_size，所以proxy_busy_buffers_size是用来控制同时传输到客户端的buffer数量的。
proxy_max_temp_file_size 128m;#默认情况下proxy_max_temp_file_size值为1024MB,也就是说后端服务器的文件不大于1G都可以缓存到nginx代理硬盘中，如果超过1G，那么文件不缓存，而是直接中转发送给客户端.如果proxy_max_temp_file_size设置为0，表示不使用临时缓存。
```

#### nginx 轮询

-   RR(默认)

每个请求按时间顺序逐一分配到不同的后端服务器，如果后端服务器down掉，能自动剔除。

```nginx
upstream tomcats {
     server 10.1.1.107:88 max_fails=3 fail_timeout=3s weight=9;
     server 10.1.1.132:80 max_fails=3 fail_timeout=3s weight=9;
     server 10.1.1.137:82  backup;
     Server 10.1.1.136:86  backup;
}
```

-   ip\_hash

每个请求按访问ip的hash结果分配，这样每个访客固定访问一个后端服务器，可以解决session(cookie)的问题

```nginx
upstream tomcats {
    ip_hash;
    server 10.1.1.107:88;
    server 10.1.1.132:80;
}
```

-   url\_hash

根据请求的\$request\_uri,一致的话会请求同一台服务器，这个应用场景很少

```nginx
upstream tomcats {
    hash $request_uri;
    server 10.1.1.107:88;
    server 10.1.1.132:80;
}
```

### nginx 变量解析

```nginx
$http_host  --localhost:7676(域名加端口)
$host  --localhost（域名）
$request_uri    -- /zjmobile/css/app.956b573.css
$http_content_type  --text/css

#一些可恶能会用到的变量
$request_method #客户端请求的动作，通常为GET或POST。
$remote_addr #客户端的IP地址。
$request_uri #包含请求参数的原始URI，不包含主机名，如：”/foo/bar.php?arg=baz”。不能修改。
$server_port #请求到达服务器的端口号(这个才是url输的端口)。
$remote_port #客户端的端口。
$query_string #与args相同。
$args #这个变量等于请求行中(GET请求)的参数，如：foo=123&bar=blahblah;
$http_user_agent #客户端agent信息
$server_protocol #请求使用的协议，通常是HTTP/1.0或HTTP/1.1。
$server_addr #服务器地址，在完成一次系统调用后可以确定这个值。
$server_name #服务器名称。
```

### nginx缓存（代理服务器端缓存，减少后端压力，提高网站并发延时）

配置

```nginx
#vim /usr/local/nginx/conf/nginx.conf 
upstream node {
    server 192.9.191.31:8081;
    server 192.9.191.31:8082;
}
proxy_cache_path /cache levels=1:2 keys_zone=xcache:10m max_size=10g inactive=60m use_temp_path=off;
server {
    listen 80;
    server_name www.test.com;
    index index.html;
    location / {
        proxy_pass http://node;
        proxy_cache xcache;
        proxy_cache_valid   200 304 12h;
        proxy_cache_valid   any 10m;
        add_header  Nginx-Cache "$upstream_cache_status";
        proxy_next_upstream error timeout invalid_header http_500 http_502 http_503 http_504;
    }
}
```

```nginx
proxy_cache_path /soft/cache levels=1:2 keys_zone=cache:10m max_size=10g inactive=60m use_temp_path=off;
#proxy_cache    //存放缓存临时文件
#levels         //按照两层目录分级
#keys_zone      //开辟空间名,10m:开辟空间大小,1m可存放8000key
#max_size       //控制最大大小,超过后Nginx会启用淘汰规则
#inactive       //60分钟没有被访问缓存会被清理
#use_temp_path  //临时文件,会影响性能,建议关闭
```

```nginx
proxy_cache cache;
proxy_cache_valid   200 304 12h;
proxy_cache_valid   any 10m;
add_header  Nginx-Cache "$upstream_cache_status";
proxy_next_upstream error timeout invalid_header http_500 http_502 http_503 http_504;
#proxy_cache            //开启缓存
#proxy_cache_valid      //状态码200|304的过期为12h,其余状态码10分钟过期
#proxy_cache_key        //缓存key
#add_header             //增加头信息,观察客户端respoce是否命中
#proxy_next_upstream    //出现502-504或错误,会跳过此台服务器访问下一台服务器
```

注意：值得一提的是nginx代理服务器缓存，多是为了缓存服务器的资源文件，如果只是反向代理服务的话就不用配置

#### 部分页面不缓存

某些页面如登陆注册，不能使用缓存

```nginx
upstream node {
        server 192.9.191.31:8081;
        server 192.9.191.31:8082;
}
proxy_cache_path /cache levels=1:2 keys_zone=cache:10m max_size=10g inactive=60m use_temp_path=off;
server {
    listen 80;
    server_name www.test.com;
    index index.html;
    if ($request_uri ~ ^/(static|login|register|password)) {
            set $cookie_nocache 1;
            }
    location / {
        proxy_pass http://node;
        proxy_cache     cache;
        proxy_cache_valid       200 304 12h;
        proxy_cache_valid       any     10m;
        add_header      Nginx-Cache     "$upstream_cache_status";
        proxy_next_upstream     error timeout invalid_header http_500 http_502 http_503 http_504;
        proxy_no_cache $cookie_nocache $arg_nocache $arg_comment;
        proxy_no_cache $http_pargma $http_authorization;
    }
}
```
