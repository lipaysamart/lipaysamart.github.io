---
title: Nginx 入门到实践
date: 2023-04-16 09:05:52
tags:
  - Nginx
categories: Middle-Operator
---
### Nginx 介绍
传统的 Web 服务器，每个客户端连接作为一个单独的进程或线程处理，需在切换任务时将 CPU 切换到新的任务并创建一个新的运行时上下文，消耗额外的内存和 CPU 时间，当并发请求增加时，服务器响应变慢，从而对性能产生负面影响。

Nginx 是开源、高性能、高可靠的 Web 和反向代理服务器，而且支持热部署，几乎可以做到 7 * 24 小时不间断运行，即使运行几个月也不需要重新启动，还能在不间断服务的情况下对软件版本进行热更新。性能是 Nginx 最重要的考量，其占用内存少、并发能力强、能支持高达 5w 个并发连接数，最重要的是，Nginx 是免费的并可以商业化，配置使用也比较简单。

Nginx 的最重要的几个使用场景：
* 静态资源服务，通过本地文件系统提供服务；
* 反向代理服务，延伸出包括缓存、负载均衡等；
* API 服务，OpenResty；

### Nginx配置
{% tabs first %}
<!-- tab 常用命令 -->
```sh
nginx -s reload  # 向主进程发送信号，重新加载配置文件，热重启
nginx -s reopen	 # 重启 Nginx
nginx -s stop    # 快速关闭
nginx -s quit    # 等待工作进程处理完成后关闭
nginx -T         # 查看当前 Nginx 最终的配置
nginx -t -c <配置路径>    # 检查配置是否有问题，如果已经在配置目录，则不需要-c
```
<!-- endtab -->
<!-- tab 配置结构图 -->
```sh
main        # 全局配置，对全局生效
├── events  # 配置影响 Nginx 服务器或与用户的网络连接
├── http    # 配置代理，缓存，日志定义等绝大多数功能和第三方模块的配置
│   ├── upstream # 配置后端服务器具体地址，负载均衡配置不可或缺的部分
│   ├── server   # 配置虚拟主机的相关参数，一个 http 块中可以有多个 server 块
│   ├── server
│   │   ├── location  # server 块可以包含多个 location 块，location 指令用于匹配 uri
│   │   ├── location
│   │   └── ...
│   └── ...
└── ...
```
<!-- endtab -->
<!-- tab 典型配置 -->
```sh
user  nginx;                        # 运行用户，默认即是nginx，可以不进行设置
worker_processes  1;                # Nginx 进程数，一般设置为和 CPU 核数一样
error_log  /var/log/nginx/error.log warn;   # Nginx 的错误日志存放目录
pid        /var/run/nginx.pid;      # Nginx 服务启动时的 pid 存放位置

events {
    use epoll;     # 使用epoll的I/O模型(如果你不知道Nginx该使用哪种轮询方法，会自动选择一个最适合你操作系统的)
    worker_connections 1024;   # 每个进程允许最大并发数
}

http {   # 配置使用最频繁的部分，代理、缓存、日志定义等绝大多数功能和第三方模块的配置都在这里设置
    # 设置日志模式
    log_format  main  '$remote_addr - $remote_user [$time_local] "$request" '
                      '$status $body_bytes_sent "$http_referer" '
                      '"$http_user_agent" "$http_x_forwarded_for"';

    access_log  /var/log/nginx/access.log  main;   # Nginx访问日志存放位置

    sendfile            on;   # 开启高效传输模式
    tcp_nopush          on;   # 减少网络报文段的数量
    tcp_nodelay         on;
    keepalive_timeout   65;   # 保持连接的时间，也叫超时时间，单位秒
    types_hash_max_size 2048;

    include             /etc/nginx/mime.types;      # 文件扩展名与类型映射表
    default_type        application/octet-stream;   # 默认文件类型

    include /etc/nginx/conf.d/*.conf;   # 加载子配置项
    
    server {
    	listen       80;       # 配置监听的端口
    	server_name  localhost;    # 配置的域名
    	
    	location / {
    		root   /usr/share/nginx/html;  # 网站根目录
    		index  index.html index.htm;   # 默认首页文件
    		deny 172.168.22.11;   # 禁止访问的ip地址，可以为all
    		allow 172.168.33.44； # 允许访问的ip地址，可以为all
    	}
    	
    	error_page 500 502 503 504 /50x.html;  # 默认50x对应的访问页面
    	error_page 400 404 error.html;   # 同上
    }
}
```
<!-- endtab -->
{% endtabs %}

```sh
# server 块可以包含多个 location 块，location 指令用于匹配 uri：
location [ = | ~ | ~* | ^~] uri {
	...
}
```
`=`  精确匹配路径，用于不含正则表达式的 uri 前，如果匹配成功，不再进行后续的查找；
`^~` 用于不含正则表达式的 uri 前，表示如果该符号后面的字符是最佳匹配，采用该规则，不再进行后续的查找；
`~`  表示用该符号后面的正则去匹配路径，区分大小写；
`~*` 表示用该符号后面的正则去匹配路径，不区分大小写。跟 ~ 优先级都比较低，如有多个location的正则能匹配的话，则使用正则表达式最长的那个；
如果 uri 包含正则表达式，则必须要有 `~` 或 `~*` 标志。

---
| Name  | Describe  |
| :---  | :---      |
|host	| 请求信息中的 Host，如果请求中没有 Host 行，则等于设置的服务器名，不包含端口
|request_method	|客户端请求类型，如 GET、POST
|remote_addr	  | 客户端的 IP 地址
|args	          | 请求中的参数
|arg_PARAMETER	|GET 请求中变量名 PARAMETER 参数的值，例如：$http_user_agent(Uaer-Agent 值), $http_referer...
|content_length	|请求头中的 Content-length 字段
|http_user_agent|	客户端agent信息
|http_cookie	  |  客户端cookie信息
|remote_addr	  |  客户端的IP地址
|remote_port	  |  客户端的端口
|http_user_agent|	客户端agent信息
|server_protocol|	请求使用的协议，如 HTTP/1.0、HTTP/1.1
|server_addr	  |  服务器地址
|server_name	  |  服务器名称
|server_port	  |  服务器的端口号
|scheme	HTTP    |  方法（如http，https）

### 配置一个反向代理
```sh
# /etc/nginx/conf.d/proxy.conf

server {
    listen 80;
    server_name proxy.theplan.space;

    location / {
      proxy_pass http://cn.bing.com
    }
}
```

---
`proxy_set_header`                在将客户端请求发送给后端服务器之前，更改来自客户端的请求头信息；
`proxy_connect_timeout:`          配置 Nginx 与后端代理服务器尝试建立连接的超时时间；
`proxy_read_timeout:`             配置 Nginx 向后端服务器组发出 read 请求后，等待相应的超时时间；
`proxy_send_timeout:`             配置 Nginx 向后端服务器组发出 write 请求后，等待相应的超时时间；
`proxy_redirect:`                 用于修改后端服务器返回的响应头中的 Location 和 Refresh。

### 配置 gzip
{% tabs three %}
<!-- tab gzip配置 -->
```sh
# /etc/nginx/conf.d/gzip.conf

gzip on; # 默认off，是否开启gzip
gzip_types text/plain text/css application/json application/x-javascript text/xml application/xml application/xml+rss text/javascript;

# 上面两个开启基本就能跑起了，下面的愿意折腾就了解一下
gzip_static on;
gzip_proxied any;
gzip_vary on;
gzip_comp_level 6;
gzip_buffers 16 8k;
# gzip_min_length 1k;
gzip_http_version 1.1;
```
<!-- endtab -->
<!-- tab gzip 说明 -->
```sh
gzip_types
# 要采用 gzip 压缩的 MIME 文件类型，其中 text/html 被系统强制启用；

gzip_static
# 默认 off，该模块启用后，Nginx 首先检查是否存在请求静态文件的 gz 结尾的文件，如果有则直接返回该 .gz 文件内容；

gzip_proxied
# 默认 off，nginx做为反向代理时启用，用于设置启用或禁用从代理服务器上收到相应内容 gzip 压缩；

gzip_vary
# 用于在响应消息头中添加 `Vary：Accept-Encoding`，使代理服务器根据请求头中的 `Accept-Encoding` 识别是否启用 gzip 压缩；

gzip_comp_level：
# gzip 压缩比，压缩级别是 1-9，1 压缩级别最低，9 最高，级别越高压缩率越大，压缩时间越长，建议 4-6；

gzip_buffers
# 获取多少内存用于缓存压缩结果，16 8k 表示以 8k*16 为单位获得；

gzip_min_length
# 允许压缩的页面最小字节数，页面字节数从header头中的 Content-Length 中进行获取。默认值是 0，不管页面多大都压缩。建议设置成大于 1k 的字节数，小于 1k 可能会越压越大；

gzip_http_version
# 默认 1.1，启用 gzip 所需的 HTTP 最低版本；
```
<!-- endtab -->
{% endtabs %}

### 负载均衡配置
```sh
http {
  upstream myserver {
  	# ip_hash;  # ip_hash 方式
    # fair;   # fair 方式
    server 127.0.0.1:8081;  # 负载均衡目的服务地址
    server 127.0.0.1:8080;
    server 127.0.0.1:8082 weight=10;  # weight 方式，不写默认为 1
  }
 
  server {
    location / {
    	proxy_pass http://myserver;
      proxy_connect_timeout 10;
    }
  }
}
```

---
**Nginx 提供了好几种分配方式，默认为轮询，就是轮流来。有以下几种分配方式**:
**轮询**:     默认方式，每个请求按时间顺序逐一分配到不同的后端服务器，如果后端服务挂了，能自动剔除；
**weight**:   权重分配，指定轮询几率，权重越高，在被访问的概率越大，用于后端服务器性能不均的情况；
**ip_hash**:  每个请求按访问 IP 的 hash 结果分配，这样每个访客固定访问一个后端服务器，可以解决动态网页 session 共享问题。负载均衡每次请求都会重新定位到服务器集群中的某一个，那么已经登录某一个服务器的用户再重新定位到另一个服务器，其登录信息将会丢失，这样显然是不妥的；
**fair**（第三方）:   按后端服务器的响应时间分配，响应时间短的优先分配，依赖第三方插件 **nginx-upstream-fair**，需要先安装

### 配置动静分离
```sh
server {
  location /www/ {
  	root /data/;
    index index.html index.htm;
  }
  
  location /image/ {
  	root /data/;
    autoindex on;
  }
}
```

---
**方式主要有两种**：
第一种方法是纯粹把静态文件独立成单独的域名，放在独立的服务器上，也是目前主流推崇的方案。
第二种方法就是动态跟静态文件混合在一起发布， 通过 Nginx 配置来分开。

通过 `location` 指定不同的后缀名实现不同的请求转发。通过 `expires` 参数设置，可以使浏览器缓存过期时间，减少与服务器之前的请求和流量。具体 expires 定义：是给一个资源设定一个过期时间，也就是说无需去服务端验证，直接通过浏览器自身确认是否过期即可，所以不会产生额外的流量。此种方法非常适合不经常变动的资源。（如果经常更新的文件，不建议使用 expires 来缓存），我这里设置 3d，表示在这 3 天之内访问这个URL，发送一个请求，比对服务器该文件最后更新时间没有变化。则不会从服务器抓取，返回状态码 304，如果有修改，则直接从服务器重新下载，返回状态码 200。

### 配置高可用
首先安装 keepalived，`yum install keepalived -y`
然后编辑 `/etc/keepalived/keepalived.conf` 配置文件，并在配置文件中增加 `vrrp_script` 定义一个外围检测机制，并在 `vrrp_instance` 中通过定义 `track_script` 来追踪脚本执行过程，实现节点转移
{% tabs second %}
<!-- tab keeplived配置 -->
```sh
global_defs{
   notification_email {
        acassen@firewall.loc
   }
   notification_email_from Alexandre@firewall.loc
   smtp_server 127.0.0.1
   smtp_connect_timeout 30 // 上面都是邮件配置，没卵用
   router_id LVS_DEVEL     // 当前服务器名字，用hostname命令来查看
}
vrrp_script chk_maintainace { // 检测机制的脚本名称为chk_maintainace
    script "[[ -e/etc/keepalived/down ]] && exit 1 || exit 0" // 可以是脚本路径或脚本命令
    // script "/etc/keepalived/nginx_check.sh"    // 比如这样的脚本路径
    interval 2  // 每隔2秒检测一次
    weight -20  // 当脚本执行成立，那么把当前服务器优先级改为-20
}
vrrp_instanceVI_1 {   // 每一个vrrp_instance就是定义一个虚拟路由器
    state MASTER      // 主机为MASTER，备用机为BACKUP
    interface eth0    // 网卡名字，可以从ifconfig中查找
    virtual_router_id 51 // 虚拟路由的id号，一般小于255，主备机id需要一样
    priority 100      // 优先级，master的优先级比backup的大
    advert_int 1      // 默认心跳间隔
    authentication {  // 认证机制
        auth_type PASS
        auth_pass 1111   // 密码
    }
    virtual_ipaddress {  // 虚拟地址vip
       172.16.2.8
    }
}
```
<!-- endtab -->
<!-- tab check-health -->
```sh
#!/bin/bash
A=`ps -C nginx --no-header | wc -l`
if [ $A -eq 0 ];then
    /usr/sbin/nginx # 尝试重新启动nginx
    sleep 2         # 睡眠2秒
    if [ `ps -C nginx --no-header | wc -l` -eq 0 ];then
        killall keepalived # 启动失败，将keepalived服务杀死。将vip漂移到其它备份节点
    fi
fi
```
<!-- endtab -->
{% endtabs %}

---
复制一份到备份服务器，备份 Nginx 的配置要将 `state` 后改为 `BACKUP`，`priority` 改为比主机小。
设置完毕后各自 `service keepalived start` 启动，经过访问成功之后，可以把 Master 机的 `keepalived` 停掉，此时 Master 机就不再是主机了 `service keepalived stop`，看访问虚拟 IP 时是否能够自动切换到备机 `ip addr`。
再次启动 Master 的 keepalived，此时 `vip` 又变到了主机上。

### HTTPS配置
```sh
server {
  listen 443 ssl http2 default_server;   # SSL 访问端口号为 443
  server_name proxy.theplan.space;         # 填写绑定证书的域名

  ssl_certificate /etc/nginx/https/your_crt.crt;   # 证书文件地址
  ssl_certificate_key /etc/nginx/https/your_key.key;      # 私钥文件地址
  ssl_session_timeout 10m;

  ssl_protocols TLSv1 TLSv1.1 TLSv1.2;      #请按照以下协议配置
  ssl_ciphers ECDHE-RSA-AES128-GCM-SHA256:HIGH:!aNULL:!MD5:!RC4:!DHE; 
  ssl_prefer_server_ciphers on;
  
  location / {
    root         /usr/share/nginx/html;
    index        index.html index.htm;
  }
}

```

---
**提高安全性**
* add_header X-Frame-Options DENY;           # 减少点击劫持
* add_header X-Content-Type-Options nosniff; # 禁止服务器自动解析资源类型
* add_header X-Xss-Protection 1;             # 防XSS攻击

### Nginx指标可用性
| NAME  | Describe  |   
|  :---            | :---     |
| `accepts`    |（接受） / accepted（已接受）	 |
| `handled`    |（已处理）	 |
| `dropped`    |（已丢弃）	 |
| `active `    |（活跃）	 |
| `request`    |（请求数）/ total（全部请求数）	| 
| `4xx 代`	 |
| `5xx 代`	 |
| `request time`|（请求处理时间）               |

### 配置 stub status模块
```sh
# 检查模块是否启用
nginx -V 2>&1 | grep -o with-http_stub_status_module

# 开启配置
server {
    location /nginx_status {
        stub_status on;             # 开启模块
        allow 127.0.0.1;            # 添加白名单
        deny all;                   # 拒绝 all访问
    }
}

# 验证文件
nginx -t

# 重启nginx
systemctl restart nginx 
```
##### 查看前端结果
![link](https://cdn.jsdelivr.net/gh/lipaysamart/Blog-pic/20230416095800.png)

### 常用技巧
##### 静态服务
```sh
server {
  listen       80;
  server_name  static.theplan.space;
  charset utf-8;    # 防止中文文件名乱码

  location /download {
    alias	          /usr/share/nginx/html/static;  # 静态资源目录
    
    autoindex               on;    # 开启静态资源列目录
    autoindex_exact_size    off;   # on(默认)显示文件的确切大小，单位是byte；off显示文件大概大小，单位KB、MB、GB
    autoindex_localtime     off;   # off(默认)时显示的文件时间为GMT时间；on显示的文件时间为服务器时间
  }
}
```
##### 图片防盗链
```sh
server {
  listen       80;        
  server_name  *.theplan.space theplan.space;
  
  # 图片防盗链
  location ~* \.(gif|jpg|jpeg|png|bmp|swf)$ {
    valid_referers none blocked server_names ~\.google\. ~\.baidu\. *.qq.com;  # 只允许本机 IP 外链引用，感谢 @木法传 的提醒，将百度和谷歌也加入白名单
    if ($invalid_referer){
      return 403;
    }
  }
}
```
##### 请求过滤
```sh
# 非指定请求全返回 403
if ( $request_method !~ ^(GET|POST|HEAD)$ ) {
  return 403;
}

location / {
  # IP访问限制（只允许IP是 192.168.0.2 机器访问）
  allow 192.168.0.2;
  deny all;
  
  root   html;
  index  index.html index.htm;
}
```
##### 配置图片、字体等静态文件缓存
```sh
# 由于图片、字体、音频、视频等静态文件在打包的时候通常会增加了 hash，所以缓存可以设置的长一点，先设置强制缓存，再设置协商缓存；如果存在没有 hash 值的静态文件，建议不设置强制缓存，仅通过协商缓存判断是否需要使用缓存。

# 图片缓存时间设置
location ~ .*\.(css|js|jpg|png|gif|swf|woff|woff2|eot|svg|ttf|otf|mp3|m4a|aac|txt)$ {
	expires 10d;
}

# 如果不希望缓存
expires -1;
15.5 单页面项目 history 路由配置
server {
  listen       80;
  server_name  fe.theplan.space;
  
  location / {
    root       /usr/share/nginx/html/dist;  # vue 打包后的文件夹
    index      index.html index.htm;
    try_files  $uri $uri/ /index.html @rewrites;  
    
    expires -1;                          # 首页一般没有强制缓存
    add_header Cache-Control no-cache;
  }
  
  # 接口转发，如果需要的话
  #location ~ ^/api {
  #  proxy_pass http://be.theplan.space;
  #}
  
  location @rewrites {
    rewrite ^(.+)$ /index.html break;
  }
}
```
##### HTTP 请求转发到 HTTPS
```sh
# 配置完 HTTPS 后，浏览器还是可以访问 HTTP 的地址 http://theplan.space/ 的，可以做一个 301 跳转，把对应域名的 HTTP 请求重定向到 HTTPS 上

server {
    listen      80;
    server_name www.theplan.space;

    # 单域名重定向
    if ($host = 'www.theplan.space'){
        return 301 https://www.theplan.space$request_uri;
    }
    # 全局非 https 协议时重定向
    if ($scheme != 'https') {
        return 301 https://$server_name$request_uri;
    }

    # 或者全部重定向
    return 301 https://$server_name$request_uri;

    # 以上配置选择自己需要的即可，不用全部加
}
```
##### 泛域名路径分离
```sh
# 这是一个非常实用的技能，经常有时候我们可能需要配置一些二级或者三级域名，希望通过 Nginx 自动指向对应目录，比如：

test1.doc.theplan.space 自动指向 /usr/share/nginx/html/doc/test1 服务器地址；
test2.doc.theplan.space 自动指向 /usr/share/nginx/html/doc/test2 服务器地址；
server {
    listen       80;
    server_name  ~^([\w-]+)\.doc\.sherlocked93\.club$;

    root /usr/share/nginx/html/doc/$1;
}
```
##### 泛域名转发
```sh
# 和之前的功能类似，有时候我们希望把二级或者三级域名链接重写到我们希望的路径，让后端就可以根据路由解析不同的规则：

test1.serv.theplan.space/api?name=a 自动转发到 127.0.0.1:8080/test1/api?name=a；
test2.serv.theplan.space/api?name=a 自动转发到 127.0.0.1:8080/test2/api?name=a ；
server {
    listen       80;
    server_name ~^([\w-]+)\.serv\.sherlocked93\.club$;

    location / {
        proxy_set_header        X-Real-IP $remote_addr;
        proxy_set_header        X-Forwarded-For $proxy_add_x_forwarded_for;
        proxy_set_header        Host $http_host;
        proxy_set_header        X-NginX-Proxy true;
        proxy_pass              http://127.0.0.1:8080/$1$request_uri;
    }
}
```