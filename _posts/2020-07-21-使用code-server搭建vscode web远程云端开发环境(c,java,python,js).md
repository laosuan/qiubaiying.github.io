---
layout:     post
title:      使用code-server搭建vscode web远程云端开发环境(c,java,python,js)
subtitle:   all contains
date:       2020-07-21
author:     laosuan
header-img: Java-Debugging-Tips-1280x720.jpg
catalog: true
tags:
    - java
---

本文旨在搭建一个web环境运行支持多语言的开发环境，方便在多个终端(mac,ipadpro...)同步一致的开发环境与体验，适合小型项目&demo的开发和简单调试。

## 1 编写Docker file，安装开发环境所需依赖

```yml
  1 FROM codercom/code-server:latest
  2
  3 CMD sudo cp /etc/apt/sources.list /etc/apt/sources.list.bak ;
  4 CMD sudo echo " " > /etc/apt/sources.list ;
  5 CMD sudo echo "deb http://mirrors.aliyun.com/debian jessie main" >> /etc/apt/sources.list ;
  6 CMD sudo echo "deb http://mirrors.aliyun.com/debian jessie-updates main" >> /etc/apt/sources.list ;
  7 RUN sudo apt-get clean
  8 RUN sudo apt-get update
  9 RUN sudo apt-get install -y maven
 10 CMD echo "installed maven"
 11 RUN sudo apt-get install -y npm
 12 #RUN sudo npm install -g @vue/cli@3.0.1
 13 CMD echo "intalled npm"
 14 ENV DEBIAN_FRONTEND noninteractive
 15 RUN sudo apt-get -y install gcc g++ gdb gdbserver cmake mono-mcs python3 python3-pip&& \
 16     sudo rm -rf /var/lib/apt/lists/*
 17
 18 ENV JAVA_HOME /home/coder/.cache/jdk1.8.0_171
 19 ENV JRE_HOME $JAVA_HOME/jre
 20 ENV CLASSPATH .:$JAVA_HOME/lib/dt.jar:$JAVA_HOME/lib/tools.jar
 21 RUN git config --global user.email "mike@liuxuan.work"
 22 RUN git config --global user.name "mike"
 23 RUN cd ~ && echo 'http://account:secret@github.com' > .git-credentials && git config --global credential.helper store
```

## 2 构建container, 运行容器
```shell
docker build -t="vscode:latest" .
docker run -it --name='vscode' -e 'PASSWORD=secret' -p 8090:8080 -p 5000-5100:5000-5100 -v "/usr/local/jdk1.8.0_171/:/opt/jdk/" -v "/usr/l    ocal/code-server/.local1:/home/coder/.local/share/code-server"  -v "/usr/local/code-server/projects:/home/coder/project" -v "/usr/local/code    -server/cache:/home/coder/.cache" -v "/usr/local/code-server/profile:/etc/profile" -v '/usr/local/code-server/repository:/home/coder/.m2/rep    ository' -v '/usr/local/code-server/settings.xml:/usr/share/maven/conf/settings.xml' vscode:latest
```

