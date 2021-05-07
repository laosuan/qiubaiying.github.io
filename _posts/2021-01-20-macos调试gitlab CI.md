---
layout:     post
title:     macos调试gitlab CI
subtitle:   
date:       2021-01-21
author:     laosuan
header-img: 
catalog: true
tags:
    - gateway


---

前言

发版的时候pineline buiild maven报错:ERROR: Job failed: exit code 1,由于没有报具体出错的原因,只能根据最后执行的命令来推测,排查十分困难,故找了个本地调试ci的方法,具体如下:



## 1 安装GitLab Runner

```
brew install gitlab-runner

brew services start gitlab-runner
```





## 2 修改ci脚本

```
#1 如需调试中间步骤,可以把之前步骤的
```





## 2 运行job

```
gitlab-runner exec docker 容器名
```







### 3.3.1，百科及知识库端用户权限

|                                  | 客户（非作者） | 客户（作者） | 员工（非作者） | 员工（作者） | 员工（管理员） |
| :------------------------------- | :------------: | :----------: | :------------: | :----------: | :------------- |
| 收到待审核、审核不通过的评论通知 |                |              |                |      √       |                |
| 收到待审核、审核不通过的私信通知 |                |              |                |              |                |
| 收到审核通过的评论通知           |       √        |      √       |                |      √       |                |
| 收到审核通过的私信通知           |       √        |      √       |                |              |                |
| 发评论                           |       √        |      √       |                |      √       | √              |
| 发私信                           |                |              |                |      √       | √              |
| 删除自己的评论、私信             |       √        |      √       |                |              |                |
| 赞评论、私信                     |       √        |      √       |                |              |                |





**refference:**

[1.Install GitLab Runner on macOS](https://docs.gitlab.com/runner/install/osx.html)

https://docs.gitlab.com/runner/commands/README.html#limitations-of-gitlab-runner-exec

https://www.lullabot.com/articles/debugging-jobs-gitlab-ci