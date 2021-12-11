---
layout:     post
title:      优秀的mac效率工具
subtitle:   tools
date:       2020-05-14
author:     laosuan
header-img: img/bill-books.jpg
catalog: true
tags:
    - Mac
---

# Zsh

查看系统shell语言

```shell
cat /etc/shells
```

```
//Centos 需自行安装
sudo yum install zsh
```

指定zsh

```shell
chsh -s /bin/zsh
```

## 配置：

### 安装autojump

```shell

// MACOS
/bin/bash -c "$(curl -fsSL https://raw.githubusercontent.com/Homebrew/install/master/install.sh)"

brew install autojump
// linux 
wget https://github.com/downloads/joelthelion/autojump/autojump_v21.1.2.tar.gz
./install.sh


add below code in ~/.zshrc
[[ -s ~/.autojump/etc/profile.d/autojump.sh ]] && . ~/.autojump/etc/profile.d/autojump.sh
```

## 安装oh-my-zsh

```bash
sh -c "$(curl -fsSL https://raw.github.com/robbyrussell/oh-my-zsh/master/tools/install.sh)"
```

### 修改主题：

```bash
 vim ~/.zshrc
```

将`ZSH_THEME`改成`ys`

```bash
ZSH_THEME="ys"
```

更新配置：

```bash
source ~/.zshrc   
```

### 添加插件
```
#1 zsh-z  For oh-my-zsh users
#Execute the following command:

git clone https://github.com/agkozak/zsh-z $ZSH_CUSTOM/plugins/zsh-z

#and add zsh-z to the line of your .zshrc that specifies plugins=(), e.g. plugins=( git zsh-z ).

#2 zsh-autosuggestions Clone this repository into $ZSH_CUSTOM/plugins (by default ~/.oh-my-zsh/custom/plugins)

git clone https://github.com/zsh-users/zsh-autosuggestions ${ZSH_CUSTOM:-~/.oh-my-zsh/custom}/plugins/zsh-autosuggestions
#Add the plugin to the list of plugins for Oh My Zsh to load (inside ~/.zshrc):

plugins=(zsh-autosuggestions)
# 按➡️键或Ctrl+f补全

#3 zsh-syntax-highlighting

git clone https://github.com/zsh-users/zsh-syntax-highlighting.git ${ZSH_CUSTOM:-~/.oh-my-zsh/custom}/plugins/zsh-syntax-highlighting

plugins=( [plugins...] zsh-syntax-highlighting)
````
# iterm

## 快捷密码

## 



# alfred 

## 快速搜索

整理如下：

### 1


## 参考

- [5分钟上手Mac效率神器Alfred以及Alfred常用操作](https://www.jianshu.com/p/e9f3352c785f)





