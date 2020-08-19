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
brew install autojump
// linux 
wget https://github.com/downloads/joelthelion/autojump/autojump_v21.1.2.tar.gz
./install.sh
[[ -s ~/.autojump/etc/profile.d/autojump.sh ]] && . ~/.autojump/etc/profile.d/autojump.sh
```

## 安装oh-my-zsh

```bash
wget https://github.com/robbyrussell/oh-my-zsh/raw/master/tools/install.sh -O - | sh
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





# alfred 

## 快速搜索

整理如下：

### 1


## 参考

- [5分钟上手Mac效率神器Alfred以及Alfred常用操作](https://www.jianshu.com/p/e9f3352c785f)





