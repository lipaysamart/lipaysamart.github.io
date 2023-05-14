---
title: Git操作指南
date: 2023-03-19 10:34:55
categories: Middle-Operator
tags: 
  - Git
  - Linux
  - Windows
thumbnail: https://cdn.jsdelivr.net/gh/lipaysamart/Blog-pic/20230319142303.png
---
### Git安装
* 下载 Git OSX 版本 https://git-scm.com/download/mac
* 下载 Git Windows 版本 https://git-scm.com/download/win
* 下载 Git Linux 版本  https://git-scm.com/download/linux
由于国内去官网下载Git比较慢，故我们选择阿里的镜像地址去下载 https://npm.taobao.org/mirrors/git-for-windows/ 根据官网最新的版本，我们在阿里镜像中找到对应的下载就行，速度飞快。安装过程直接按照默认的就行。  
---
### Git使用
{% tabs First tab %}
<!-- tab 简单使用 -->
```sh
    git init                                        #初始化仓库
    git clone                                       #从本地拉取一个仓库可以使用 git clone /path/to/repository
                                                    #从远端拉取一个仓库可以使用 git clone username@host:/path/to/repository
    git add <filename>                              #将改动添加至缓存区,也可以使用（git add *）
    git commit -m <'information'>                   #提交改动
    git push origin <master>                        #推送到仓库
    git remote add origin <server>                  #添加远程服务器
    git checkout -b <feature-X>                     #创建一个分支
    git checkout <feature>                          #切换分支
    git branch -d <feature-X>                       #删除分支
    git push origin <branch>                        #推送分支（未推送到仓库前，他人是无法看见的）
    git pull                                        #更新本地仓库至最新（也可以在后面加上你的远程仓库地址）
    git merage <branch>                             #合并分支（注意你当前分支的切换）
    git tag <lable> <ID>                            #创建标签 
    git log                                         #获取提交 ID
    git checkout --<filename>                       #替换本地改动（替换工作目录的文件，添加到缓存和新文件，不受影响）
    git fetch origin                                #指向一个服务器
    git reset --hard origin/master                  #重置你所有的本地改动与提交（通常与git fetch origin使用）
    gitk                                            #图形化 git
```
<!-- endtab -->
<!-- tab 进阶使用 -->
```sh
    git config --global --edit                                      #编辑配置文件
    git config --global user.name "Danny"                           #用户名称信息
    git config --global user.email Danny@example.com                #用户邮箱信息
    git config --global core.editor vim                             #更换文本编辑器
    git config --global core.editor "/path/to/editor/Notepad.exe"   #Windos上需要指定绝对路径
    git config --list                                               #列出Git配置信息
    git config <key>                                                #检查某一项配置（git config user.name）
    git config --unset <key>.<name>                                 #删除某一项配置（git config --unset user.name）
    git config color.ui true                                        #彩色的 git 输出
```
<!-- endtab -->
<!-- tab 高阶使用 -->
    test
<!-- endtab -->
{% endtabs %}
---




 