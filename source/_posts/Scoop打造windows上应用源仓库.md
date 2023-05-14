---
title: Scoop打造windows上应用源仓库
date: 2023-04-01 22:34:55
tags:
  - Windows
  - Scoop
thumbnail: 
categories: Windows-Operator
---

#### Scoop 了解 
* Scoop 是个优质的 Windows 包管理器。它不仅轻量，还将软件直接安装到我们的用户目录下，安程不需要申请管理员权限（UAC）也不会污染系统环境变量。
* Scoop 官网：https://www.scoop.sh
* Scoop 仓库：https://github.com/ScoopInstaller/Scoop
---
#### Scoop 应用场景
* 重装系统或新电脑装了window
* 批量安装程序
* 多线程下载
---
#### Scoop 安装&环境要求
* 用户名不含中文
* PowerShell 7+
* Windows10 1607+ / Windows Server 2012+
* .Net Framework 4.5+
---

#### Scoop 配置
```sh
# install
Set-ExecutionPolicy -ExecutionPolicy RemoteSigned -Scope CurrentUser
# irm -useb get.scoop.sh | iex
irm https://ghproxy.com/raw.githubusercontent.com/lzwme/scoop-proxy-cn/master/install.ps1 | iex

# config
scoop config SCOOP_REPO https://ghproxy.com/github.com/ScoopInstaller/Scoop

# 安装git ，不然后面无法拉取仓库
scoop install  7zip aria2 git        

scoop bucket rm main
# scoop bucket add main https://ghproxy.com/github.com/ScoopInstaller/Main
scoop bucket add spc https://ghproxy.com/https://github.com/lzwme/scoop-proxy-cn

# show help
scoop help

# 安装必备应用： scoop-search、aria2...
scoop install spc/scoop-search spc/aria2
```

`安装好进入 "C:\Users\{$user}\scoop"`

1.目录说明：
* apps 存放已安装的工具。
* buckets 存放添加的源仓库。其中 buckets/main 为官方源。
* cache 下载的安装包缓存。若长久使用后占用空间太大了可以清理掉。
* shims 已安装工具的入口文件。
___
2.其他说明：
* 如果安装时指定了 --global 参数，则安装的位置为：C:\ProgramData\scoop。
* 可设置环境变量 SCOOP 指定当前用户默认安装的位置。
* 可设置环境变量 SCOOP_GLOBAL 指定全局默认安装的位置。
* 当我们想要找指定APP时有没有仓库存在，我们可以使用Google搜索 [AppName + scoop]
___
3.添加其它**buckets**软件库：
* scoop bucket known 列出已安装的 bucket
* scoop bucket add <bucketname> 添加一个 bucket
* scoop bucket rm <bucketname> 删除一个 bucket
___
4.Scoop基本指令：
* 查找：scoop-search rust 从本地 buckets 中查找包(rust)
* 查看：scoop info rust 查看一个包的基本信息（rust）
* 安装：scoop install rust 安装一个包(rust)
* 卸载：scoop uninstall rust 卸载一个包(rust)
* 更新：scoop update [rust] 更新一个或全部包(rust)
___

{% tabs Scoop %}
<!-- tab 安装示例 -->
```sh
# 使用 sudo 全局安装需要系统管理员权限的应用
scoop install sudo
sudo scoop install 7zip git openssh --global

scoop install aria2 curl grep sed less touch

# 安装常见编程开发语言支持
scoop install python ruby go perl rust php

# 安装 Linux 命令行 gow
scoop install gow

# 安装 cmder
scoop install cmder
```
<!-- endtab -->
<!-- tab 简单示例 -->
```sh
scoop list                           # 查看已安装程序

scoop status                        # 查看更新

scoop checkup                        # 自身诊断

scoop hold <softname>               # 软件暂停更新

scoop reset <softname@版本号>       # 切换到指定版本

scoop reset *                        # 重置所有软件链接及图标

scoop cache rm *                    # 删除缓存软件包

scoop cleanup rm *                  # 删除旧版本

scoop home <app_name>               # 打开应用主页(homepage)
```
<!-- endtab -->
<!-- tab 可选应用 -->
```txt
7zip                压缩与解压工具

aria2               让 scoop 在批量安装多个应用时，以多进程模式并发下载和安装

NetEaseMusic        网易云音乐

WeChatWork          企业微信

wechat              微信

utools              新一代效率工具平台，插件即应用。

fscapture           轻量好用的截图工具

Tencent-Meeting     腾讯会议

Xshell	            好用的SSH/Telnet等协议连接工具
```
<!-- endtab -->
{% endtabs %}

#### 定制私有仓库

* 参考以下项目仓库克隆至本地
`git clone https://github.com/duzyn/scoop-cn.git`

* 进入 my-bucket `cd my-bucket`
**创建 App 的安装配置文件 7zip.json**(参数详解[App-Manifests](https://github.com/ScoopInstaller/Scoop/wiki/App-Manifests))  
```json
{
    "version": "22.01",
    "description": "A multi-format file archiver with high compression ratios",
    "homepage": "https://www.7-zip.org/",
    "license": "LGPL-2.1-or-later",
    "notes": "Add 7-Zip as a context menu option by running: \"$dir\\install-context.reg\"",
    "architecture": {
        "64bit": {
            "url": "https://experiments-alicdn.sparanoid.net/7z/7z2201-x64.msi",
            "hash": "f4afba646166999d6090b5beddde546450262dc595dddeb62132da70f70d14ca",
            "extract_dir": "Files\\7-Zip"
        },
        "32bit": {
            "url": "https://experiments-alicdn.sparanoid.net/7z/7z2201.msi",
            "hash": "a4913f98821e0da0c58cd3a7f2a59f1834b85b6ca6b3fdefa5454d6c3bbef54c",
            "extract_dir": "Files\\7-Zip"
        },
        "arm64": {
            "url": "https://experiments-alicdn.sparanoid.net/7z/7z2201-arm64.exe",
            "hash": "700dea3e4012319a09ccadfce91cf090334cfe658d0bdc42204e77acbea1ef99",
            "pre_install": [
                "$7zr = Join-Path $env:TMP '7zr.exe'",
                "Invoke-WebRequest https://experiments-alicdn.sparanoid.net/7z/7zr.exe -OutFile $7zr",
                "Invoke-ExternalCommand $7zr @('x', \"$dir\\$fname\", \"-o$dir\", '-y') | Out-Null",
                "Remove-Item \"$dir\\Uninstall.exe\", \"$dir\\*-arm64.exe\", $7zr"
            ]
        }
    },
    "post_install": [
        "$7zip_root = \"$dir\".Replace('\\', '\\\\')",
        "'install-context.reg', 'uninstall-context.reg' | ForEach-Object {",
        "    $content = Get-Content \"$bucketsdir\\main\\scripts\\7-zip\\$_\"",
        "    $content = $content.Replace('$7zip_root', $7zip_root)",
        "    if ($global) {",
        "       $content = $content.Replace('HKEY_CURRENT_USER', 'HKEY_LOCAL_MACHINE')",
        "    }",
        "    Set-Content \"$dir\\$_\" $content -Encoding Ascii",
        "}"
    ],
    "bin": [
        "7z.exe",
        "7zFM.exe",
        "7zG.exe"
    ],
    "shortcuts": [
        [
            "7zFM.exe",
            "7-Zip"
        ]
    ],
    "persist": [
        "Codecs",
        "Formats"
    ],
    "checkver": {
        "url": "https://www.7-zip.org/download.html",
        "regex": "Download 7-Zip ([\\d.]+)"
    },
    "autoupdate": {
        "architecture": {
            "64bit": {
                "url": "https://experiments-alicdn.sparanoid.net/7z/7z$cleanVersion-x64.msi"
            },
            "32bit": {
                "url": "https://experiments-alicdn.sparanoid.net/7z/7z$cleanVersion.msi"
            },
            "arm64": {
                "url": "https://experiments-alicdn.sparanoid.net/7z/7z$cleanVersion-arm64.exe"
            }
        }
    }
}
```
* 将本地更改同步至 GitHub
```sh
git add .
git commit -m "add 7zip app"
git push
```
* 添加你的bucket库
`scoop bucket add my-bucket https://github.com/<你的 GitHub 用户名>/my-bucket`

* 测试是否成功
`scoop install <仓库名>/7zip`


{% notel blue 精选的第三方仓库参考(适用于国内) %}
https://github.com/scoopcn/scoopcn
https://github.com/kkzzhizhou/scoop-apps
https://github.com/Paxxs/Cluttered-bucket
https://github.com/duzyn/scoop-cn
{% endnotel %}