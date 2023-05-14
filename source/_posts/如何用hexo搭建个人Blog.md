---
title: 如何用Hexo搭建个人Blog
date: 2023-03-11 21:43:29
categories: Other-Operator
tags: 
  - Hexo
thumbnail: "https://cdn.jsdelivr.net/gh/lipaysamart/Blog-pic/20230313160808.png"
---
### Hexo环境准备

##### 安装Node.js
直接到官网上下载安装即可https://nodejs.org/en/download/

* Node.js (Node.js 版本需不低于 10.13，建议使用 Node.js 12.0 及以上版本)
* Node自带npm
* `npm install -g cnpm --registry=https://registry.npm.taobao.org`      #更换npm源

##### 安装Git
* Windows：下载并安装 git.
* Mac：使用 Homebrew, MacPorts 或者下载 安装程序。
* Linux (Ubuntu, Debian)：sudo apt-get install git-core
* Linux (Fedora, Red Hat, CentOS)：sudo yum install git-core
* 安装完之后到你要安装的路径下右键打开"git bash here"

### 开始安装Hexo
```
    npm install -g hexo-cli
    or
    cnpm install -g hexo-cli
```
* 输入hexo -v 查看版本

##### 新建Blog文件夹/初始化Hexo/安装npm
```
    hexo init Blog          #初始化Hexo
    cd blog
    npm install             #安装npm
```
##### 启动服务Site
```
    hexo g                  #生成 hexo generate 
    hexo s                  #启动服务预览 hexo server                           
```
* 本地访问http://localhost:4000/ 至此Hexo就搭建好了。

### 将网站托管至GitHub
##### 1. 新建仓库和Token
仓库名称格式：用户名+GitHub.io
Token: 登录github设置setting->Developer Settings->Prosonal access tokens 注意勾选权限！
##### 2. 安装upload插件
`npm install hexo-deployer-git --save`
##### 3. 修改Hexo配置文件指定仓库路径
在你blog文件夹下找到"_config.yml,ctrl+f 定位到deploy配置,注意格式deploy下的配置要缩进两行。
```yaml
    deploy:
      type: git
      repo: https://<Token>@github.com/用户名/仓库名.git
      branch: main
```
##### 4. 推送Site到Github
`hexo d`                      #部署hexo deploy

##### 5. 访问地址 https://仓库名.github.io/

### 更换主题
想要更换掉固定模板风格，你可以在GitHub搜索Hexo主题或者去到https://hexo.io/themes/下挑选自定义模板，之后按照该模板下的 How To Use? step by step 即可。
一般步骤为：
1. 下载
2. 把包丢到themes文件夹下面
3. 配置Hero的_config.yml，修改"themes: 模板名"
4. 本地调试完之后，我们就可以推送到GitHub上了。
5. cd到你的blog文件夹下Use commond:`hexo clean` -> `hexo g` -> `hexo d`
6. 访问地址 https://仓库名.github.io/

### 新建文章
Hexo使用Markdown语法，它可以使普通文本具有一定的格式。
cd至blog文件下 `hexo new "my frist blog"`
查看source资源下有没有我们刚刚创建的文章
之后就是发布的步骤//清理hexo clean//构建hexo g//上传hexo d （如果上传报错，大多是网络原因，多上传几次即可）


### 新建页面
有时我们不满足主题自由的一些页面，希望自己添加一些页面。
我们可以新建页面,新建页面则会在hexo的source中新建该页面文件并生成md文件，这就是你要编辑的博客页了。
`hexo new page "my home page"`
然后打开主题的配置文件_config.yml，在菜单属性menu中的添加如下（注意不是Hexo的配置文件）
将页面路径联接到页面上去（是一个key:value）左侧定义菜单,右侧定义页面

#### Hexo命令清单
```
    npm install hexo -g                      #安装Hexo
    npm update hexo -g                       #升级
    hexo init                                #初始化博客
    hexo server                              #Hexo会监视文件变动并自动更新，无须重启服务器
    hexo server -s                           #静态模式
    hexo server -p 5000                      #更改端口
    hexo server -i 192.168.1.1               #自定义 IP
    hexo clean                               #清除缓存，若是网页正常情况下可以忽略这条命令
    hexo new draft "文件名"                  #新建草稿文件
    hexo server --draft                      #预览草稿文件
    hexo publish "文件名"                    #发布草稿文件     
```
#### 命令简写
```
    hexo n "我的博客" == hexo new "我的博客"          #新建文章
    hexo g == hexo generate                          #生成      
    hexo s == hexo server                            #启动服务预览
    hexo d == hexo deploy                            #部署
```

