---
title: pinus 学习（二）默认组件介绍
description: 各默认组件的功能介绍
slug: pinuslearn2
date: 2024-05-29 00:00:00+0000
categories:
    - pinus
    - TypeScript
tags:
    - 笔记
weight: 1       # You can add weight to some posts to override the default sorting (date descending)
params:
    author: 王新晓
---
# pinus 学习（二）默认组件介绍

上节介绍了pinus框架创建的helloworld示例工程的创建,编译,运行,以及服务器启动的源码分析
下面来看一下各个组件的具体实现与功能

### ProxyComponent

rpc的客户端，在服务器启动后由Monitor组件监视到rpc服务器后添加rpc服务器，使用惰性连接，在第一次使用的时候才会和rpc服务器建立连接，可以自定义使用的协议，默认使用的是mqtt协议

### RemoteComponent

rpc服务器，默认使用mqtt协议，会在构造函数里读取服务器组件中的remote和插件中的remote，然后再start里设置

### MonitorComponent



### BackendSessionComponent

### ServerComponent



### MasterComponent

### 



