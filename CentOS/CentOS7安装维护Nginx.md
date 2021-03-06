# Nginx 安装

Nginx 是一款面向性能设计的 HTTP 服务器，能反向代理 HTTP，HTTPS 和邮件相关(SMTP，POP3，IMAP)的协议链接。并且提供了负载均衡以及 HTTP 缓存。它的设计充分使用异步事件模型，削减上下文调度的开销，提高服务器并发能力。采用了模块化设计，提供了丰富模块的第三方模块。

所以关于 Nginx，有这些标签：「异步」「事件」「模块化」「高性能」「高并发」「反向代理」「负载均衡」

Linux系统：`Centos 7 x64`
Nginx版本：`1.11.5`

## 目录

- [安装](#安装)
  - [安装依赖](#安装依赖)
  - [下载](#下载)
  - [编译安装](#编译安装)
  - [nginx测试](#nginx测试)
- [开机自启动](#开机自启动)
- [运维](#运维)
  - [服务管理](#服务管理)
  - [重启服务防火墙报错解决](#重启服务防火墙报错解决)
- [nginx卸载](#nginx卸载)
- [参数说明](#参数说明)
- [配置](#配置)
  - [配置文件](#配置文件)
  - [反向代理](#反向代理)
  - [模块upstream](#模块upstream)
- [常见使用场景](#常见使用场景)
  - [跨域问题](#跨域问题)
  - [ssl配置](#ssl配置)

## 安装

### 安装依赖

> prce(重定向支持)和openssl(https支持，如果不需要https可以不安装。)

```bash
yum -y install pcre*
yum -y install openssl*
```

CentOS 6.5 我安装的时候是选择的“基本服务器”，默认这两个包都没安装全，所以这两个都运行安装即可。

### 下载

```bash
wget http://nginx.org/download/nginx-1.11.5.tar.gz

# 如果没有安装wget
# 下载已编译版本
$ yum install wget
```

### 编译安装

然后进入目录编译安装，[configure参数说明](#configure参数说明)

```bash
cd nginx-1.7.8
./configure
```

安装报错误的话比如：“C compiler cc is not found”，这个就是缺少编译环境，安装一下就可以了 **yum -y install gcc make gcc-c++ openssl-devel wget**

如果没有error信息，就可以执行下边的安装了：

```bash
make
make install
```

### nginx测试

运行下面命令会出现两个结果

```bash
./nginx -t

# nginx: the configuration file /usr/local/nginx/conf/nginx.conf syntax is ok
# nginx: configuration file /usr/local/nginx/conf/nginx.conf test is successful
```

## 开机自启动

编辑 **vi /lib/systemd/system/nginx.service** 文件，没有创建一个 **touch nginx.service** 然后将如下内容根据具体情况进行修改后，添加到nginx.service文件中：

```bash
[Unit]
Description=nginx1.11.5
After=network.target remote-fs.target nss-lookup.target

[Service]

Type=forking
PIDFile=/var/run/nginx.pid
ExecStartPre=/usr/local/nginx/sbin/nginx -t -c /usr/local/nginx/conf/nginx.conf
ExecStart=/usr/local/nginx/sbin/nginx -c /usr/local/nginx/conf/nginx.conf
ExecReload=/bin/kill -s HUP $MAINPID
ExecStop=/bin/kill -s QUIT $MAINPID
PrivateTmp=true

[Install]
WantedBy=multi-user.target
```

设置开机启动，使配置生效：

```
systemctl enable nginx.service
```

## 运维

### 服务管理

```bash
# 启动
/usr/local/nginx/sbin/nginx

# 重启
/usr/local/nginx/sbin/nginx -s reload

# 关闭进程
/usr/local/nginx/sbin/nginx -s stop

# 平滑关闭nginx
/usr/local/nginx/sbin/nginx -s quit

# 查看nginx的安装状态，
/usr/local/nginx/sbin/nginx -V 
```

**关闭防火墙，或者添加防火墙规则就可以测试了**

```
service iptables stop
```

或者编辑配置文件：

```
vi /etc/sysconfig/iptables
```

添加这样一条开放80端口的规则后保存：

```
-A INPUT -m state --state NEW -m tcp -p tcp --dport 80 -j ACCEPT
```

重启服务即可:

```
service iptables restart
```

### 重启服务防火墙报错解决


```bash
service iptables restart
# Redirecting to /bin/systemctl restart  iptables.service
# Failed to restart iptables.service: Unit iptables.service failed to load: No such file or directory.
```

在CentOS 7或RHEL 7或Fedora中防火墙由 **firewalld** 来管理，当然你可以还原传统的管理方式。或则使用新的命令进行管理。
假如采用传统请执行一下命令：

```bash
# 传统命令
systemctl stop firewalld
systemctl mask firewalld
```

```bash
# 安装命令
yum install iptables-services

systemctl enable iptables 
service iptables restart
```


## nginx卸载

如果通过yum安装，使用下面命令安装。

```bash
yum remove nginx
```

编译安装，删除/usr/local/nginx目录即可
如果配置了自启动脚本，也需要删除。


## 参数说明

- --prefix=`<path>` - Nginx安装路径。如果没有指定，默认为 /usr/local/nginx。
- --sbin-path=`<path>` - Nginx可执行文件安装路径。只能安装时指定，如果没有指定，默认为`<prefix>`/sbin/nginx。
- --conf-path=`<path>` - 在没有给定-c选项下默认的nginx.conf的路径。如果没有指定，默认为`<prefix>`/conf/nginx.conf。
- --pid-path=`<path>` - 在nginx.conf中没有指定pid指令的情况下，默认的nginx.pid的路径。如果没有指定，默认为 `<prefix>`/logs/nginx.pid。
- --lock-path=`<path>` - nginx.lock文件的路径。
- --error-log-path=`<path>` - 在nginx.conf中没有指定error_log指令的情况下，默认的错误日志的路径。如果没有指定，默认为 `<prefix>`/- logs/error.log。
- --http-log-path=`<path>` - 在nginx.conf中没有指定access_log指令的情况下，默认的访问日志的路径。如果没有指定，默认为 `<prefix>`/- logs/access.log。
- --user=`<user>` - 在nginx.conf中没有指定user指令的情况下，默认的nginx使用的用户。如果没有指定，默认为 nobody。
- --group=`<group>` - 在nginx.conf中没有指定user指令的情况下，默认的nginx使用的组。如果没有指定，默认为 nobody。
- --builddir=DIR - 指定编译的目录
- --with-rtsig_module - 启用 rtsig 模块
- --with-select_module --without-select_module - Whether or not to enable the select module. This module is enabled - by default if a more suitable method such as kqueue, epoll, rtsig or /dev/poll is not discovered by configure.
- //允许或不允许开启SELECT模式，如果 configure 没有找到更合适的模式，比如：kqueue(sun os),epoll (linux kenel 2.6+), rtsig(- 实时信号)或者/dev/poll(一种类似select的模式，底层实现与SELECT基本相 同，都是采用轮训方法) SELECT模式将是默认安装模式
- --with-poll_module --without-poll_module - Whether or not to enable the poll module. This module is enabled by - default if a more suitable method such as kqueue, epoll, rtsig or /dev/poll is not discovered by configure.
- --with-http_ssl_module - Enable ngx_http_ssl_module. Enables SSL support and the ability to handle HTTPS requests. - Requires OpenSSL. On Debian, this is libssl-dev.
- //开启HTTP SSL模块，使NGINX可以支持HTTPS请求。这个模块需要已经安装了OPENSSL，在DEBIAN上是libssl
- --with-http_realip_module - 启用 ngx_http_realip_module
- --with-http_addition_module - 启用 ngx_http_addition_module
- --with-http_sub_module - 启用 ngx_http_sub_module
- --with-http_dav_module - 启用 ngx_http_dav_module
- --with-http_flv_module - 启用 ngx_http_flv_module
- --with-http_stub_status_module - 启用 "server status" 页
- --without-http_charset_module - 禁用 ngx_http_charset_module
- --without-http_gzip_module - 禁用 ngx_http_gzip_module. 如果启用，需要 zlib 。
- --without-http_ssi_module - 禁用 ngx_http_ssi_module
- --without-http_userid_module - 禁用 ngx_http_userid_module
- --without-http_access_module - 禁用 ngx_http_access_module
- --without-http_auth_basic_module - 禁用 ngx_http_auth_basic_module
- --without-http_autoindex_module - 禁用 ngx_http_autoindex_module
- --without-http_geo_module - 禁用 ngx_http_geo_module
- --without-http_map_module - 禁用 ngx_http_map_module
- --without-http_referer_module - 禁用 ngx_http_referer_module
- --without-http_rewrite_module - 禁用 ngx_http_rewrite_module. 如果启用需要 PCRE 。
- --without-http_proxy_module - 禁用 ngx_http_proxy_module
- --without-http_fastcgi_module - 禁用 ngx_http_fastcgi_module
- --without-http_memcached_module - 禁用 ngx_http_memcached_module
- --without-http_limit_zone_module - 禁用 ngx_http_limit_zone_module
- --without-http_empty_gif_module - 禁用 ngx_http_empty_gif_module
- --without-http_browser_module - 禁用 ngx_http_browser_module
- --without-http_upstream_ip_hash_module - 禁用 ngx_http_upstream_ip_hash_module
- --with-http_perl_module - 启用 ngx_http_perl_module
- --with-perl_modules_path=PATH - 指定 perl 模块的路径
- --with-perl=PATH - 指定 perl 执行文件的路径
- --http-log-path=PATH - Set path to the http access log
- --http-client-body-temp-path=PATH - Set path to the http client request body temporary files
- --http-proxy-temp-path=PATH - Set path to the http proxy temporary files
- --http-fastcgi-temp-path=PATH - Set path to the http fastcgi temporary files
- --without-http - 禁用 HTTP server
- --with-mail - 启用 IMAP4/POP3/SMTP 代理模块
- --with-mail_ssl_module - 启用 ngx_mail_ssl_module
- --with-cc=PATH - 指定 C 编译器的路径
- --with-cpp=PATH - 指定 C 预处理器的路径
- --with-cc-opt=OPTIONS - Additional parameters which will be added to the variable CFLAGS. With the use of the system library PCRE in FreeBSD, it is necessary to indicate --with-cc-opt="-I /usr/local/include". If we are using select() and it is necessary to increase the number of file descriptors, then this also can be assigned here: --with-cc-opt="-D FD_SETSIZE=2048".
- --with-ld-opt=OPTIONS - Additional parameters passed to the linker. With the use of the system library PCRE in - FreeBSD, it is necessary to indicate --with-ld-opt="-L /usr/local/lib".
- --with-cpu-opt=CPU - 为特定的 CPU 编译，有效的值包括：pentium, pentiumpro, pentium3, pentium4, athlon, opteron, amd64, - sparc32, sparc64, ppc64
- --without-pcre - 禁止 PCRE 库的使用。同时也会禁止 HTTP rewrite 模块。在 "location" 配置指令中的正则表达式也需要 PCRE 。
- --with-pcre=DIR - 指定 PCRE 库的源代码的路径。
- --with-pcre-opt=OPTIONS - Set additional options for PCRE building.
- --with-md5=DIR - Set path to md5 library sources.
- --with-md5-opt=OPTIONS - Set additional options for md5 building.
- --with-md5-asm - Use md5 assembler sources.
- --with-sha1=DIR - Set path to sha1 library sources.
- --with-sha1-opt=OPTIONS - Set additional options for sha1 building.
- --with-sha1-asm - Use sha1 assembler sources.
- --with-zlib=DIR - Set path to zlib library sources.
- --with-zlib-opt=OPTIONS - Set additional options for zlib building.
- --with-zlib-asm=CPU - Use zlib assembler sources optimized for specified CPU, valid values are: pentium, pentiumpro
- --with-openssl=DIR - Set path to OpenSSL library sources
- --with-openssl-opt=OPTIONS - Set additional options for OpenSSL building
- --with-debug - 启用调试日志
- --add-module=PATH - Add in a third-party module found in directory PATH


## 配置

在Centos 默认配置文件在 **/usr/local/nginx-1.5.1/conf/nginx.conf** 我们要在这里配置一些文件。nginx.conf是主配置文件，由若干个部分组成，每个大括号`{}`表示一个部分。每一行指令都由分号结束`;`，标志着一行的结束。

### 配置文件

nginx 的配置系统由一个主配置文件和其他一些辅助的配置文件构成。这些配置文件均是纯文本文件，全部位于 nginx 安装目录下的 conf 目录下。

指令由 nginx 的各个模块提供，不同的模块会提供不同的指令来实现配置。
指令除了 Key-Value 的形式，还有作用域指令。

nginx.conf 中的配置信息，根据其逻辑上的意义，对它们进行了分类，也就是分成了多个作用域，或者称之为配置指令上下文。不同的作用域含有一个或者多个配置项。

下面的这些上下文指令是用的比较多：

| Directive |  Description | Contains Directive |
| ---- | ---- | ---- |
| main  |  nginx 在运行时与具体业务功能（比如 http 服务或者 email 服务代理）无关的一些参数，比如工作进程数，运行的身份等。 | user, worker_processes, error_log, events, http, mail |
| http  |  与提供 http 服务相关的一些配置参数。例如：是否使用 keepalive 啊，是否使用 gzip 进行压缩等。 |  server |
| server | http 服务上支持若干虚拟主机。每个虚拟主机一个对应的 server 配置项，配置项里面包含该虚拟主机相关的配置。在提供 mail 服务的代理时，也可以建立若干 server. 每个 server 通过监听的地址来区分。| listen, server_name, access_log, location, protocol, proxy, smtp_auth, xclient |
| location  |  http 服务中，某些特定的 URL 对应的一系列配置项。  | index, root |
| mail | 实现 email 相关的 SMTP/IMAP/POP3 代理时，共享的一些配置项（因为可能实现多个代理，工作在多个监听地址上）。 | server, http, imap_capabilities |
| include | 以便增强配置文件的可读性，使得部分配置文件可以重新使用。 | - |
| valid_referers | 用来校验Http请求头Referer是否有效。 | - |
| try_files | 用在server部分，不过最常见的还是用在location部分，它会按照给定的参数顺序进行尝试，第一个被匹配到的将会被使用。 | - |
| if | 当在location块中使用if指令，在某些情况下它并不按照预期运行，一般来说避免使用if指令。 | - |


例如我们再 **nginx.conf** 里面引用两个配置 vhost/example.com.conf 和 vhost/gitlab.com.conf 它们都被放在一个我自己新建的目录 vhost 下面。nginx.conf 配置如下：

```
worker_processes  1;
events {
    worker_connections  1024;
}

http {
    include       mime.types;
    default_type  application/octet-stream;

    #log_format  main  '$remote_addr - $remote_user [$time_local] "$request" '
    #                  '$status $body_bytes_sent "$http_referer" '
    #                  '"$http_user_agent" "$http_x_forwarded_for"';

    #access_log  logs/access.log  main;

    sendfile        on;
    #tcp_nopush     on;

    #keepalive_timeout  0;
    keepalive_timeout  65;

    #gzip  on;
    server {
        listen       80;
        server_name  localhost;
        location / {
            root   html;
            index  index.html index.htm;
        }
        error_page   500 502 503 504  /50x.html;
        location = /50x.html {
            root   html;
        }
    }
    include  vhost/example.com.conf;
    include  vhost/gitlab.com.conf;
}
```


简单的配置: example.com.conf

```
server {
    #侦听的80端口
    listen       80;
    server_name  baidu.com app.baidu.com; # 这里指定域名
    index        index.html index.htm;    # 这里指定默认入口页面
    root /home/www/app.baidu.com;         # 这里指定目录
}
```

### 内置预定义变量

Nginx提供了许多预定义的变量，也可以通过使用set来设置变量。你可以在if中使用预定义变量，也可以将它们传递给代理服务器。以下是一些常见的预定义变量，[更多详见](http://nginx.org/en/docs/varindex.html)

| 变量名称  |  值 |
| ----  | ---- |
| $args_name | 在请求中的name参数 |
| $args      | 所有请求参数 |
| $query_string   | $args的别名 |
| $content_length | 请求头Content-Length的值 |
| $content_type   | 请求头Content-Type的值 |
| $host |  如果当前有Host，则为请求头Host的值；如果没有这个头，那么该值等于匹配该请求的server_name的值 |
| $remote_addr  |  客户端的IP地址 |
| $request      |  完整的请求，从客户端收到，包括Http请求方法、URI、Http协议、头、请求体 |
| $request_uri  |  完整请求的URI，从客户端来的请求，包括参数 |
| $scheme | 当前请求的协议 |
| $uri    | 当前请求的标准化URI |

### 反向代理

反向代理是一个Web服务器，它接受客户端的连接请求，然后将请求转发给上游服务器，并将从服务器得到的结果返回给连接的客户端。复杂的配置: gitlab.com.conf。

```
server {
    #侦听的80端口
    listen       80;
    server_name  git.example.cn;
    location / {
        proxy_pass   http://localhost:3000;
        #以下是一些反向代理的配置可删除
        proxy_redirect             off;
        #后端的Web服务器可以通过X-Forwarded-For获取用户真实IP
        proxy_set_header           Host $host;
        client_max_body_size       10m; #允许客户端请求的最大单文件字节数
        client_body_buffer_size    128k; #缓冲区代理缓冲用户端请求的最大字节数
        proxy_connect_timeout      300; #nginx跟后端服务器连接超时时间(代理连接超时)
        proxy_send_timeout         300; #后端服务器数据回传时间(代理发送超时)
        proxy_read_timeout         300; #连接成功后，后端服务器响应时间(代理接收超时)
        proxy_buffer_size          4k; #设置代理服务器（nginx）保存用户头信息的缓冲区大小
        proxy_buffers              4 32k; #proxy_buffers缓冲区，网页平均在32k以下的话，这样设置
        proxy_busy_buffers_size    64k; #高负荷下缓冲大小（proxy_buffers*2）
    }
}
```

代理到上游服务器的配置中，最重要的是proxy_pass指令。以下是代理模块中的一些常用指令：

| 指令 | 说明 |
| ---- | ---- |
| proxy_connect_timeout  | Nginx从接受请求至连接到上游服务器的最长等待时间 |
| proxy_send_timeout  | 后端服务器数据回传时间(代理发送超时) |
| proxy_read_timeout  | 连接成功后，后端服务器响应时间(代理接收超时) |
| proxy_cookie_domain | 替代从上游服务器来的Set-Cookie头的domain属性 |
| proxy_cookie_path   | 替代从上游服务器来的Set-Cookie头的path属性 |
| proxy_buffer_size   | 设置代理服务器（nginx）保存用户头信息的缓冲区大小 |
| proxy_buffers       | proxy_buffers缓冲区，网页平均在多少k以下 |
| proxy_set_header    | 重写发送到上游服务器头的内容，也可以通过将某个头部的值设置为空字符串，而不发送某个头部的方法实现 |
| proxy_ignore_headers | 这个指令禁止处理来自代理服务器的应答。 | 
| proxy_intercept_errors | 使nginx阻止HTTP应答代码为400或者更高的应答。 | 

### 模块upstream

upstream指令启用一个新的配置区段，在该区段定义一组上游服务器。这些服务器可能被设置不同的权重，也可能出于对服务器进行维护，标记为down。

```
upstream  gitlab {
    ip_hash;
    server 192.168.122.11:8081 ;
    server 127.0.0.1:3000;
    server 127.0.0.1:3001 down;
    keepalive 32;
}
server {
    #侦听的80端口
    listen       80;
    server_name  git.example.cn;
    location / {
        proxy_pass   http://gitlab;    #在这里设置一个代理，和upstream的名字一样
        #以下是一些反向代理的配置可删除
        proxy_redirect             off;
        #后端的Web服务器可以通过X-Forwarded-For获取用户真实IP
        proxy_set_header           Host $host;
        proxy_set_header           X-Real-IP $remote_addr;
        proxy_set_header           X-Forwarded-For $proxy_add_x_forwarded_for;
        client_max_body_size       10m; #允许客户端请求的最大单文件字节数
        client_body_buffer_size    128k; #缓冲区代理缓冲用户端请求的最大字节数
        proxy_connect_timeout      300; #nginx跟后端服务器连接超时时间(代理连接超时)
        proxy_send_timeout         300; #后端服务器数据回传时间(代理发送超时)
        proxy_read_timeout         300; #连接成功后，后端服务器响应时间(代理接收超时)
        proxy_buffer_size          4k; #设置代理服务器（nginx）保存用户头信息的缓冲区大小
        proxy_buffers              4 32k; #proxy_buffers缓冲区，网页平均在32k以下的话，这样设置
        proxy_busy_buffers_size    64k; #高负荷下缓冲大小（proxy_buffers*2）
        proxy_temp_file_write_size 64k; #设定缓存文件夹大小，大于这个值，将从upstream服务器传
    }
}
```

**server指令可选参数：**

1. weight：设置一个服务器的访问权重，数值越高，收到的请求也越多；
2. fail_timeout：在这个指定的时间内服务器必须提供响应，如果在这个时间内没有收到响应，那么服务器将会被标记为down状态；
3. max_fails：设置在fail_timeout时间之内尝试对一个服务器连接的最大次数，如果超过这个次数，那么服务器将会被标记为down;
4. down：标记一个服务器不再接受任何请求；
5. backup：一旦其他服务器宕机，那么有该标记的机器将会接收请求。

**keepalive指令：**

Nginx服务器将会为每一个worker进行保持同上游服务器的连接。

**负载均衡：**

upstream模块能够使用3种负载均衡算法：轮询、IP哈希、最少连接数。

**轮询：** 默认情况下使用轮询算法，不需要配置指令来激活它，它是基于在队列中谁是下一个的原理确保访问均匀地分布到每个上游服务器；  
**IP哈希：** 通过ip_hash指令来激活，Nginx通过IPv4地址的前3个字节或者整个IPv6地址作为哈希键来实现，同一个IP地址总是能被映射到同一个上游服务器；  
**最少连接数：** 通过least_conn指令来激活，该算法通过选择一个活跃数最少的上游服务器进行连接。如果上游服务器处理能力不同，可以通过给server配置weight权重来说明，该算法将考虑到不同服务器的加权最少连接数。

## 常见使用场景

### 跨域问题

在工作中，有时候会遇到一些接口不支持跨域，这时候可以简单的添加add_headers来支持cors跨域。配置如下：

```
server {
  listen 80;
  server_name api.xxx.com;
    
  add_header 'Access-Control-Allow-Origin' '*';
  add_header 'Access-Control-Allow-Credentials' 'true';
  add_header 'Access-Control-Allow-Methods' 'GET,POST,HEAD';

  location / {
    proxy_pass http://127.0.0.1:3000;
    proxy_set_header X-Real-IP $remote_addr;
    proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
    proxy_set_header Host  $http_host;    
  } 
}
```

### ssl配置

超文本传输安全协议（缩写：HTTPS，英语：Hypertext Transfer Protocol Secure）是超文本传输协议和SSL/TLS的组合，用以提供加密通讯及对网络服务器身份的鉴定。HTTPS连接经常被用于万维网上的交易支付和企业信息系统中敏感信息的传输。HTTPS不应与在RFC 2660中定义的安全超文本传输协议（S-HTTP）相混。

HTTPS 目前已经是所有注重隐私和安全的网站的首选，随着技术的不断发展，HTTPS 网站已不再是大型网站的专利，所有普通的个人站长和博客均可以自己动手搭建一个安全的加密的网站。

查看目前nginx编译选项

```
sbin/nginx -V
```

输出下面内容

```
nginx version: nginx/1.7.8
built by gcc 4.4.7 20120313 (Red Hat 4.4.7-4) (GCC)
TLS SNI support enabled
configure arguments: --prefix=/usr/local/nginx-1.5.1 --with-http_ssl_module --with-http_spdy_module --with-http_stub_status_module --with-pcre
```

如果依赖的模块不存在，可以输入下面命令重新编译安装。

```
./configure --user=www --group=www --prefix=/mt/server/nginx --with-http_stub_status_module --with-openssl=/home/nginx-1.8.0/openssl-1.0.0d --without-http-cache --with-http_ssl_module --with-http_gzip_static_module --with-...
```


```
# HTTPS server
server {
    listen       443 ssl;
    server_name  localhost;

    ssl_certificate /usr/local/nginx/conf/vjjhd.crt;
    ssl_certificate_key /usr/local/nginx/conf/vjjhd.key;

    # 设置ssl/tls会话缓存的类型和大小。如果设置了这个参数一般是shared，buildin可能会参数内存碎片，默认是none，和off差不多，停用缓存。如shared:SSL:10m表示我所有的nginx工作进程共享ssl会话缓存，官网介绍说1M可以存放约4000个sessions。 
    ssl_session_cache    shared:SSL:1m; 

    # 客户端可以重用会话缓存中ssl参数的过期时间，内网系统默认5分钟太短了，可以设成30m即30分钟甚至4h。
    ssl_session_timeout  5m; 
    
    # 选择加密套件，不同的浏览器所支持的套件（和顺序）可能会不同。
    # 这里指定的是OpenSSL库能够识别的写法，你可以通过 openssl -v cipher 'RC4:HIGH:!aNULL:!MD5'（后面是你所指定的套件加密算法） 来看所支持算法。
    ssl_ciphers  HIGH:!aNULL:!MD5;

    # 设置协商加密算法时，优先使用我们服务端的加密套件，而不是客户端浏览器的加密套件。
    ssl_prefer_server_ciphers  on;

    location / {
        root   html;
        index  index.html index.htm;
    }
}
```

