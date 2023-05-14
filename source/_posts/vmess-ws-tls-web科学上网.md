---
title: Vmess + TLS + WS 实现科学上网
date: 2023-03-12 12:48:39
categories: Network-Operator
thumbnail: https://cdn.jsdelivr.net/gh/lipaysamart/Blog-pic/20230317182134.png
tags:
  - 科学上网
---
### 环境准备
* vps购买：https://bandwagonhost.com/               #系统使用：Debian 10
* *优惠码：`BWHNCXNVXV`*     

* 域名购买：https://namesilo.com/     

* ssh工具：https://www.hostbuf.com/t/988.html     


### 节点搭建
{% note primary  %} 
直接全选复制粘贴即可
{% endnote %} 
`apt update -y && apt-get install vim curl wget  -y`

###### 安装x-ui面板
X-ui出于安全考虑，安装/更新完成后需要强制修改端口与账户密码，step by step 即可
安装完成后输入`x-ui`即可管理面板
`bash <(curl -Ls https://raw.githubusercontent.com/vaxilu/x-ui/master/install.sh)`
![link](/images/xui-1.png)
配置如上所示端口随便，id用默认生成的就行，点击查看按钮复制链接导入V2ray
![link](/images/xui-2.png)
对应的面板设置，重启完面板后我们是登录不进去的，因为还没配置nginx

###### 安装nginx
```sh
    apt install nginx socat -y
    #安装acme：
    curl https://get.acme.sh | sh
    #添加软链接：
    ln -s  /root/.acme.sh/acme.sh /usr/local/bin/acme.sh
    #切换CA机构： 
    acme.sh --set-default-ca --server letsencrypt
    #申请证书： 
    acme.sh  --issue -d 你的域名 -k ec-256 --webroot  /var/www/html
    #安装证书：
    acme.sh --install-cert -d 你的域名 --ecc --key-file       /etc/x-ui/server.key  --fullchain-file /etc/x-ui/server.crt --reloadcmd     "systemctl force-reload nginx"
``` 
###### 配置nginx.conf  
`vim /etc/nginx/nginx.conf` 替换原有配置文件。
```sh
    user www-data;
    worker_processes auto;
    pid /run/nginx.pid;
    include /etc/nginx/modules-enabled/*.conf;    
    events {
        worker_connections 1024;
    }

    http {
        sendfile on;
        tcp_nopush on;
        tcp_nodelay on;
        keepalive_timeout 65;
        types_hash_max_size 2048;

        include /etc/nginx/mime.types;
        default_type application/octet-stream;
        gzip on;

        server {
            listen 443 ssl;

            server_name nicename.co;  #你的域名
            ssl_certificate       /etc/x-ui/server.crt;  #证书位置
            ssl_certificate_key   /etc/x-ui/server.key; #私钥位置

            ssl_session_timeout 1d;
            ssl_session_cache shared:MozSSL:10m;
            ssl_session_tickets off;
            ssl_protocols    TLSv1.2 TLSv1.3;
            ssl_prefer_server_ciphers off;

            location / {
                proxy_pass https://bing.com; #伪装网址
                proxy_redirect off;
                proxy_ssl_server_name on;
                sub_filter_once off;
                sub_filter "bing.com" $server_name;
                proxy_set_header Host "bing.com";
                proxy_set_header Referer $http_referer;
                proxy_set_header X-Real-IP $remote_addr;
                proxy_set_header User-Agent $http_user_agent;
                proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
                proxy_set_header X-Forwarded-Proto https;
                proxy_set_header Accept-Encoding "";
                proxy_set_header Accept-Language "zh-CN";
            }


            location /ray {   #分流路径
                proxy_redirect off;
                proxy_pass http://127.0.0.1:10000; #Xray端口
                proxy_http_version 1.1;
                proxy_set_header Upgrade $http_upgrade;
                proxy_set_header Connection "upgrade";
                proxy_set_header Host $host;
                proxy_set_header X-Real-IP $remote_addr;
                proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
            }

            location /xui {   #xui路径
                proxy_redirect off;
                proxy_pass http://127.0.0.1:9999;  #xui监听端口
                proxy_http_version 1.1;
                proxy_set_header Host $host;
            }
        }

        server {
            listen 80;
            location /.well-known/ {
                   root /var/www/html;
                }
            location / {
                    rewrite ^(.*)$ https://$host$1 permanent;
                }
        }
    }
```
###### 多租户   
通过修改nginx的配置文件实现ws path路径分流  
```sh
    location /ray {   #分流路径
    proxy_redirect off;
    proxy_pass http://127.0.0.1:10000; #Xray端口
    proxy_http_version 1.1;
    proxy_set_header Upgrade $http_upgrade;
    proxy_set_header Connection "upgrade";
    proxy_set_header Host $host;
    proxy_set_header X-Real-IP $remote_addr;
    proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
    }
```
{% note danger %}
TLS开启之后记得也要去你的域名服务商处更改为端对端完全加密，不然会提示重定向次数过多
{% endnote %}