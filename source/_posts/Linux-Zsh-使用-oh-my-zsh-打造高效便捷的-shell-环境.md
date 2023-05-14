---
title: Linux Zsh + oh-my-zsh 打造高效便捷的 shell 环境
date: 2023-03-23 21:13:45
tags:
  - Linux
thumbnail: 
categories: Linux-System-Operator
---
### 安装Zsh
Zsh 完全兼容 bash，支持强大的智能命令补全功能，支持大量的界面主题和插件，功能和效率都极大的增强和提高。
* CentOS: `yum -y install zsh`
* Ubunto: `apt install zsh`  
##### 设置 默认 Shell 为 zsh
`echo $shell`                   # 查看当前Shell
`chsh -s /bin/zsh`              # 给 root 用户设置默认Shell
`chsh -s /bin/zsh <username> `  # 给特定用户设置默认Shell  
显示 **"Changing shell for root"** 则表示切换成功

---

### 安装 oh-my-zsh & 插件
这里我们采用自动安装，使用如下一键sh脚本
`sh -c "$(curl -fsSL https://raw.github.com/ohmyzsh/ohmyzsh/master/tools/install.sh)"`
##### 修改主题样式
`vim .zshrc`                     #这里是当前路径
**ZSH_THEM**字段就是主题（Passion）
![link](https://cdn.jsdelivr.net/gh/lipaysamart/Blog-pic/20230323212915.png)  
*基础样式可以查看这里: https://github.com/ohmyzsh/ohmyzsh/wiki/Themes* 
*额外主题可以查看这里: https://github.com/ohmyzsh/ohmyzsh/wiki/External-themes*
**你还可以快速修改** `sed -i '/^ZSH_THEME=.*/c ZSH_THEME="themes"' ~/.zshrc && source .zshrc`

---

##### 命令自动补全 zsh-completions
* **自动列出目录**
输入 `cd` 按 **tab** 键，目录将自动列出，在按 **tab** 可以切换

* **自动目录名简写补全**
要访问 `/usr/local/bi`n 这个长路径，只需要 `cd /u/l/b` 按 **tab** 键自动补全

* **自动大小写更正**
要访问 home 目录，只需要 `cd ho` 按 **tab** 键自动补全，或者查看 README.md，只需要 `cat rea` 自动更正补全

* **自动命令补全**
输入 `kubectl` 按 **tab** 键即可看到可用命令

* **自动补全命令参数**
输入 `kill` 按 **tab** 键会自动显示出进程的 **process id**  

```sh
git clone --depth=1 https://github.com/zsh-users/zsh-completions ${ZSH_CUSTOM:-${ZSH:-~/.oh-my-zsh}/custom}/plugins/zsh-completions
fpath+=${ZSH_CUSTOM:-${ZSH:-~/.oh-my-zsh}/custom}/plugins/zsh-completions/src 
```
![link](https://cdn.jsdelivr.net/gh/lipaysamart/Blog-pic/20230323223023.png)  
##### 依据历史命令补全 zsh-autosuggestions
根据历史输入命令的记录即时的提示（建议补全），然后按 → 键即可补全

```sh
git clone --depth=1 https://github.com/zsh-users/zsh-autosuggestions.git ${ZSH_CUSTOM:-${ZSH:-~/.oh-my-zsh}/custom}/plugins/zsh-autosuggestions
vim .zshrc              # 添加插件名字 zsh-autosuggestions
source .zshrc
```
![link](https://cdn.jsdelivr.net/gh/lipaysamart/Blog-pic/20230323222329.png)

##### 语法高亮插件 zsh-syntax-highlighting
命令错误会显示红色，直到你输入正确才会变绿色，另外路径正确会显示下划线
```sh
git clone --depth=1 https://github.com/zsh-users/zsh-syntax-highlighting.git ${ZSH_CUSTOM:-~/.oh-my-zsh/custom}/plugins/zsh-syntax-highlighting
vim .zshrc              #添加插件名字 zsh-syntax-highlighting
source .zshrc
```
![link](https://cdn.jsdelivr.net/gh/lipaysamart/Blog-pic/20230323222218.png)




