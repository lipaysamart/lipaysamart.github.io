---
title: 一台VPS搭建属于你的私有ChatGPT
date: 2023-04-11 20:38:08
tags:
  - ChatGPT
categories: Other-Operator
---
{% notel primary 前言 %}
相信最近有不少的小伙伴在登录chatGPT页面的时候会出现 <font color="red">1020</font> 错误码，这是因为你的IP地址被禁止登录 https://chat.openai.com 这个页面了，今天我就教大家如何通过调用APIkey的方式在国内可以继续访问chatGPT来提高我们的工作效率。
{% endnotel %}

#### 介绍&环境
* 一台国外服务器，最好是美国或日本
* 一个OpenAI账号，最好是用服务器所在地的短信号码进行注册的
* 安装Linux系统，Debian 10 or CentOS 7
* 安装docker源

#### 安装部署
首先我们需要去创建我们的 APIkey, 进入这个网址 https://platform.openai.com, 如果首次进的话需要登录或者注册（以上就不详细说明如何注册，网上很多方法Google一下就行）
![link](https://cdn.jsdelivr.net/gh/lipaysamart/Blog-pic/20230411205953.png)
创建之后记得一定要保存我们的 "secret key" 
![link](https://cdn.jsdelivr.net/gh/lipaysamart/Blog-pic/20230411210224.png)

接下来我们就需要在服务器上进行操作了，这里推荐使用 "FinalShell" 客户端有海外加速功能
{% tabs first %}
<!-- tab CentOS7-->
```sh
# 安装 yum 包
yum install -y yum-utils

# 添加 docker 仓库
yum-config-manager \
    --add-repo \
    https://download.docker.com/linux/centos/docker-ce.repo

# 安装docker
yum install docker-ce docker-ce-cli containerd.io docker-buildx-plugin docker-compose-plugin

# 开机自启动 
systemctl enable docker --now

# 测试 docker 是否安装成功
docker run hello-world  
```
<!-- endtab -->
<!-- tab Debian -->
```sh
# 更新软件源
apt-get update

# 安装依赖软件
apt-get install \
    ca-certificates \
    curl \
    gnupg

# 添加docker GPG key
mkdir -m 0755 -p /etc/apt/keyrings
curl -fsSL https://download.docker.com/linux/debian/gpg |  gpg --dearmor -o /etc/apt/keyrings/docker.gpg

# 设置 docker仓库和安装版本
echo \
  "deb [arch="$(dpkg --print-architecture)" signed-by=/etc/apt/keyrings/docker.gpg] https://download.docker.com/linux/debian \
  "$(. /etc/os-release && echo "$VERSION_CODENAME")" stable" | \
   tee /etc/apt/sources.list.d/docker.list > /dev/null

# 安装 docker 
chmod a+r /etc/apt/keyrings/docker.gpg

apt-get update
apt-get install docker-ce docker-ce-cli containerd.io docker-buildx-plugin docker-compose-plugin

# 测试 docker是否安装成功
docker run hello-world
```
<!-- endtab -->
{% endtabs %}
看见屏幕输出显示 "Hello from Docker!" 就代表安装成功了，到这里为止我们的环境就搭建好了。
![link](https://cdn.jsdelivr.net/gh/lipaysamart/Blog-pic/20230411211614.png)

接下来我们只需要执行以下命令安装程序，即可大功告成
```sh
docker run  -d -p 0.0.0.0:3002:3002 --name chatgpt-web  --env OPENAI_API_KEY=你复制的API_KEY  chenzhaoyu94/chatgpt-web

docker ps  # 查看运行的容器是否为 UP
```
如果运行为Down可以执行 `docker rm chatgpt-web` 解决报错后在重新部署即可

#### 验证
接下来我们打开浏览器 http://服务器IP地址:3002/ 


#### 添加安全配置
想必大家登录进去的时候会发现不需要输入账号和密码，只是需要一个外网IP和端口就可以随意登录进去，而且还是http协议，这样对我们的网站非常不安全而且还容易被人盗用我们的APIkey（目前只有18美金免费），所以我们需要给我们的服务器加上点安全认证
`docker rm -f chatgpt-web`删除前面创建的容器， 然后重新执行以下命令！
```sh
docker run  -d -p 0.0.0.0:3002:3002 --name chatgpt-web --env AUTH_SECRET_KEY=你访问登录的密钥 --env OPENAI_API_KEY=你复制的API_KEY chenzhaoyu94/chatgpt-web
```
当我们再次访问的时候就会弹出一个验证的框，此时输入你的登录密钥即可。
![link](https://cdn.jsdelivr.net/gh/lipaysamart/Blog-pic/20230411214321.png)


##### 给网站添加SSL证书认证
当然你还可以为网站添加一个证书，以保证你的流量不是以明文形式暴露在外网. 这里我们使用 **Let's Encrypt**来申请免费证书
{% tabs second %}
<!-- tab CentOS7 -->
```sh
# 安装 epel扩展源
yum install epel-release -y

# 安装 Ngninx
yum install nginx -y 

# 安装 Certbot客户端
yum install certbot -y 

# 获取证书 (注意该模式下会占用443端口，如果有Nginx或apache 要先停止)
certbot certonly --standalone  -d www.example.com
```
<!-- endtab -->
<!-- tab Debian -->
```sh
# 安装nginx
apt install nginx -y

# 安装 Certbot客户端
apt install certbot -y

# 获取证书 (注意该模式下会占用443端口，如果有Nginx或apache 要先停止)
certbot certonly --standalone  -d www.example.com
```
<!-- endtab -->
{% endtabs %}

证书生成完毕后，我们可以在 `/etc/letsencrypt/live/`目录下看到对应域名的文件夹。接下来我们要配置 Nginx， 将以下配置文件粘贴到 `/etc/nginx/nginx.conf` **"http{}"** 模块下。
```sh
 server {
    listen 80;
    server_name 你的域名;
    return 301 https://$server_name$request_uri;
  }

  server {
    listen 443 ssl;
    server_name 你的域名;

    # SSL证书和密钥的路径。
    ssl_certificate /etc/letsencrypt/live/api.diamondfsd.com/fullchain.pem;
    ssl_certificate_key /etc/letsencrypt/live/api.diamondfsd.com/privkey.pem;

    location / {
      proxy_pass http://服务器IP地址:3002;
    }
  }
```
再次打开浏览器，直接输入域名即可访问到我们的chatGPT-WEB了，并且我们左上角也添加了一把小锁。
{% note warning %}
机器人的回复速度会受到网络影响，所以卡顿和加载超时属于正常现象。
{% endnote %}