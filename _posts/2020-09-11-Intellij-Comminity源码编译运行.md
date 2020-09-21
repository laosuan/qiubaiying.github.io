---
layout:     post
title:      docker必知必会
subtitle:   
date:       2020-09-17
author:     laosuan
header-img: 
catalog: true
tags:
    - docker


---



## Docker保存修改后的镜像

### 1.启动镜像并做出修改

docker run -it centos /bin/bash

[root@afcaf46e8305 /]#
注意afcaf46e8305是产生的容器ID，前面运行的时候不要-d后台运行了，不然无法进入容器交互执行模式：

安装vim并且退出容器：
yum install -y vim
exit

### 2.把容器打包成镜像

docker commit afcaf46e8305 centos-vim

### 3.查看镜像centos-vim

docker images | grep centos-vim
查看镜像的详细信息：
docker inspect centos-vim:afcaf46e8305

### 4.使用centos-vim这个镜像

docker run -it centos-vim /bin/bash
发现可以直接使用vim了，而不需要重新安装：
vim --version

### 5.OPTIONS说明

-a :提交的镜像作者；
-c :使用Dockerfile指令来创建镜像；
-m :提交时的说明文字；
-p :在commit时，将容器暂停。
将容器a404c6c174a2 保存为新的镜像,并添加提交人信息和说明信息。
docker commit -a "runoob.com" -m "my apache" a404c6c174a2 mymysql:v1



## docker pull 镜像

```
docker build -t=“openresty:1.11.2.5-overlay-20191031" .
docker tag openresty:1.11.2.5-overlay-20190801 docker.gf.com.cn/clickeggs/openresty:1.11.2.5-overlay-20190801
docker push docker.gf.com.cn/clickeggs/openresty:1.11.2.5-overlay-20190801
```



refference：

- [环境配置](https://www.cnblogs.com/demojie/p/11743565.html)
- [从零开始编写IntelliJ IDEA插件](https://juejin.im/post/6844904058625474573)
- [官方开发文档](https://jetbrains.org/intellij/sdk/docs/intro/welcome.html)
- [官方问题列表](https://youtrack.jetbrains.com/issues)

