---
title: address-already-in-use
date: 2020-04-05 06:06:42
tags: [address-already-in-use,port]
categories: 爬出深坑
---

> 上周遇到个熟悉的问题`Address already in use: bind`，本来以为是个简单的端口占用问题，没想到花了很长时间才解决，避免以后再次入坑，特此记录。

### 常规端口占用

windows
> netstat -ano|findstr ${端口号}      #查找端口  
>  taskkill /f /pid ${端口号}   #杀死进程

Linux & Mac
> lsof -i:${端口号}  
> kill -9 ${端口号}

### Hyper-V保留端口

这次遇到端口占用问题，是在笔记本被强制更新之后遇到的(Windows10系统)，用了上述的方法怎么也不好使，有人说用管理员执行`netsh winsock reset`，然后机器重启就行了。可是我执行后重启了5，6次，程序依然还是报`Address already in use: bind`。这时候我就比较迷惑了，一般说程序员3大法宝：`重启程序`，`重启电脑`，`重装系统`，没有什么是解决不了的。因为是公司电脑，重装系统是不可能的了。重启程序和电脑我试了不下10遍，问题依然存在，这时候人就比较慌了。

后来上Github搜索可能是`Hyper-V`的问题，按照[issue: Unable to bind ports](https://github.com/docker/for-win/issues/3171)的方式查看发现我程序的启动端口在`Hyper-V`的保留端口的范围之内，于是乎虽然端口没被占用，但是程序依然会报`Address already in use: bind`。于是把`Hyper-V`卸了，重启电脑问题解决。

#### 查找`Hyper-V`保留端口范围

在`cmd`执行
> netsh interface ipv4 show excludedportrange protocol=tcp

返回

```
Protocol tcp Port Exclusion Ranges

Start Port    End Port
----------    --------
     49692       49791
     49792       49891
     49892       49991
     49992       50091
     50092       50191
     50214       50313
     50498       50597

* - Administered port exclusions.
```

#### issue提供的解决方法

1. 禁用`Hyper-V`（需要2次重启）

> dism.exe /Online /Disable-Feature:Microsoft-Hyper-V

2. 重启结束之后，保留你想要的端口使`Hyper-V`无法占用该端口

> netsh int ipv4 add excludedportrange protocol=tcp startport=50051 numberofports=1

3. 重启`Hyper-V`（还是需要重启）

> dism.exe /Online /Enable-Feature:Microsoft-Hyper-V /All

然后你就会发现程序可以正常的启动了

因为我用不到`Hyper-V`，所以直接卸载了`Hyper-V`。

事后感叹：Windows更新真的很坑呀！！！ 