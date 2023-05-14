---
title: Gcore CDN 加速你的站点
date: 2023-03-22 20:58:17
tags:
  - Gore
  - 科学上网
categories: Network-Operator
thumbnail:
--- 
### 介绍&环境准备
Gcore 是一家提供全球托管、CDN、边缘和云服务的公司。它拥有超过 140 个 CDN POPs 和 15 个云位置的全球网络，可以保护网站、应用和服务器免受复杂的 DDoS 攻击。Gcore 还专注于视频点播和直播流媒体的优化3，并且最近推出了一种支持 150 多个国家连接的零信任 5G eSIM 云平台。
* 准备一个域名  https://namesilo.com/
* 注册一个**Gcore**账号（用QQ邮箱也可以） https://gcore.com/
### 配置Gcore
首先进入https://gcore.com/网站后，选择 **"Try for free"**
![link](https://cdn.jsdelivr.net/gh/lipaysamart/Blog-pic/20230322210930.png)
输入你注册的账号登录之后，进入主页面在左侧栏找到 **CDN** 点开选中 **"CDN resources"** 选择创建 **"Create CDN resource"**
![link](https://cdn.jsdelivr.net/gh/lipaysamart/Blog-pic/20230322211445.png)
选择第一个加速类型（无需修改代码加速）
![link](https://cdn.jsdelivr.net/gh/lipaysamart/Blog-pic/20230322211945.png)
"exmaple" 里面填写你的域名，**"alias"** 随便填写一个别名（稍后需要用到做解析）
![link](https://cdn.jsdelivr.net/gh/lipaysamart/Blog-pic/20230322212503.png)
这里需要指定你域名绑定的IP地址(解析类型必须是 "A" 类型),然后点击ADD 
![link](https://cdn.jsdelivr.net/gh/lipaysamart/Blog-pic/20230322212830.png)
在这个页面我们可以看到Gcore想让你把域名托管到它家我们这里使用 **"CNAME"** 就不进行NS转移了，直接下一步即可
![link](https://cdn.jsdelivr.net/gh/lipaysamart/Blog-pic/20230322213231.png)
直接下一步即可，然后会弹出个页面，我们点击小窗口中的 **"open resource setting"**
![link](https://cdn.jsdelivr.net/gh/lipaysamart/Blog-pic/20230322213435.png)
在 "Resource settings" 页面 我们需要配置以下几个参数 开启 **"SSL"** ，**"CDN cache"** 设置不缓存
![link](https://cdn.jsdelivr.net/gh/lipaysamart/Blog-pic/20230322213856.png)
这边我们的 **"Resource settings"** 配置就完善好了，然后去到DNS查看我们的 **"CDN"** , **"Step by Step"** 即可
![link](https://cdn.jsdelivr.net/gh/lipaysamart/Blog-pic/20230322220609.png)
此时我们回到域名注册服务商处，生成一条 **"Type"** 类型为 **"Cname"**，**"Name"** 里面填你的别名前缀 **"Target"** 指定我们刚刚复制的 **"CDN"** （如果你是托管在CF，要把 **"Proxy status"** 关掉）
![link]("https://cdn.jsdelivr.net/gh/lipaysamart/Blog-pic/20230322222858.png")
### 检验配置
通常在你 **"Get SSl certificate"** 会需要15-20分钟生效，这里我们可以打开无痕模式输入你之前注册的 **"Cname"** 别名,在域名前面出现了一把小锁，然后返回的信息是 **"bad request"** 代表已经生效了
![link](https://cdn.jsdelivr.net/gh/lipaysamart/Blog-pic/20230322214528.png)
我们也可以通过 **"Ping"** 命令去检验一下 ，有返回信息则代表解析成功。
![link](https://cdn.jsdelivr.net/gh/lipaysamart/Blog-pic/20230322215441.png)
### Tips
如果你是用来做科学上网，这里需要配置你的源端口,默认是80
![link](https://cdn.jsdelivr.net/gh/lipaysamart/Blog-pic/20230322214109.png)

### 关于域名申请证书问题申请证书的方式有两种:
* dns-01:需要证书申请工具向域名解析网站发起请求(所以需要把域名解析网站的 **"key"** 和 **"secret"** 传给证书申请工具);
* http-01:不需要依赖于域名解析网站，只需要把域名正确解析到当前服务器，你就可以在当前服务器申请，缺点是无法申请通配符证书(但可以一个证书多个域名);
前面我们做cdn的域名 **"alias.example.com"** 肯定是要申请证书的，如果你之前申请过 **"example.com"** 通配符证书，那么其实就不用管证书的问题了，因为新加的 **"alias.example.com"** 也会用这个通配符证书。











