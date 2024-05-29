---
title: pinus 学习（一）项目结构与项目启动介绍
description: 项目结构介绍，项目入口，服务器启动流程介绍
slug: pinusLearn0
date: 2024-05-28 00:00:00+0000
categories:
    - pinus
    - TypeScript
tags:
    - 笔记
weight: 1       # You can add weight to some posts to override the default sorting (date descending)
params:
    author: 王新晓
---

# pinus 学习（一）项目结构与项目启动介绍
项目结构介绍，项目入口，服务器启动流程介绍

## 目录结构

创建一个helloworld示例工程，会创建两个服务器，一个game-server一个web-server，game-server是游戏服务器，目录结构如下

![game-server目录](image-20240528165211562.png)

* 目录app中存放项目代码
* 目录config中存放服务器配置
* 目录dist是编译后生成的目录
* 目录node_modules是第三方库包
* 文件app.ts是服务器入口
* 文件copy.js 功能是将配置文件拷贝到dist目录
* 文件preload.ts实现捕获错误
* 文件package.js与package-lock.json 是npm包管理配置文件 （依赖的第三方库和自定义脚本）
* 文件tsconfig.json 是 typescript项目的配置文件（编译的配置与需要编译的文件）

## 项目结构

创建的helloworld的示例工程的game-server中启动了两个服务器，master服务器和connector服务器

服务器中通过添加组件（component)来增加功能

#### master服务器

##### MasterComponent

监控功能，启动了其他服务器

##### MonitorComponent

监控功能

#### connector服务器

##### ProxyComponent

rpc 中 的client，调用其他服务器的函数

##### RemoteComponent

rpc 中 的server，服务与其他服务器，接收其他服务器的调用

##### ConnectionComponent

前台服务器才有，管理连接上的用户信息与用户数量

##### ConnectorComponent

前台服务器才有，管理用户连接，发送与接受消息，（根据配置会决定是否启动**DictionaryComponent**和**ProtobufComponent**）

##### SessionComponent

前台服务器才有，管理sessionID与socket的对应关系，sessionID在connectorComponent中设置

##### PushSchedulerComponent

前台服务器才有，推送消息

##### BackendSessionComponent

绑定后台服务器sessionID与（用户和前台服务器）的关系，连接绑定前台服务器与后台服务器的用户

##### ChannelComponent

将几个用户串成一组，可以群发消息

##### ServerComponent

服务器功能，其中有Filter（过滤器），Handler（处理某条客户端消息），Crons（一些定时任务）

##### MonitorComponent

监控功能

## 项目启动

在game-server目录中执行`npm i`安装第三方依赖，然后运行`tsc`，编译TS代码，编译结果将输出到dist文件夹中，然后运行`node cooy` 拷贝配置文件到dist目录中，再在dist目录中执行`node app` ,服务器就启动了。



### app.ts

这个文件是入口，创建项目后默认的代码如下

```ts
import { pinus } from 'pinus';
import { preload } from './preload';

/**
 *  替换全局Promise
 *  自动解析sourcemap
 *  捕获全局错误
 */
preload();

/**
 * Init app for client.
 */
let app = pinus.createApp();
app.set('name', 'helloworld');

// app configuration
app.configure('production|development', 'connector', function () {
    app.set('connectorConfig',
        {
            connector: pinus.connectors.hybridconnector,
            heartbeat: 3,
            useDict: true,
            useProtobuf: true
        });
});

// start app
app.start();
```

这里会启动Master服务器，app.configure的第二个参数说明，这个配置是connector服务器的配置，master在这里不会执行， app.start（）中会加载Master服务器的两个默认Component，并初始化，然后再根据配置文件servers.json去启动connector服务器

先来看下app.ts里面的内容

`let app = pinus.createApp();` 这里创建了Application类，在createApp中调用了init，然后使用`app.configure()` 来给每个服务器设置配置，添加自定义组件，最后`app.start()` 启动服务器



```ts
start(cb ?: (err ?: Error, result ?: void) => void) {
        this.startTime = Date.now();
        if (this.state > STATE_INITED) {
            utils.invokeCallback(cb, new Error('application has already start.'));
            return;
        }

        let self = this;
        appUtil.startByType(self, function () {
            appUtil.loadDefaultComponents(self);
            let startUp = function () {
                self.state = STATE_BEFORE_START;
                logger.info('%j enter before start...', self.getServerId());

                appUtil.optComponents(self.loaded, Constants.RESERVED.BEFORE_START, function (err) {
                    if (err) {
                        utils.invokeCallback(cb, err);
                    } else {
                        logger.info('%j enter start...', self.getServerId());

                        appUtil.optComponents(self.loaded, Constants.RESERVED.START, function (err) {
                            self.state = STATE_START;
                            if (err) {
                                utils.invokeCallback(cb, err);
                            } else {
                                logger.info('%j enter after start...', self.getServerId());
                                self.afterStart(cb);
                            }
                        });
                    }
                });

            };

            appUtil.optLifecycles(self.usedPlugins, Constants.LIFECYCLE.BEFORE_STARTUP, self, function (err) {
                if (err) {
                    utils.invokeCallback(cb, err);
                } else {
                    startUp();
                }
            });
        });
    }
```



## Component实现

下面来看一下各个组件的具体实现，先看Master的两个默认组件

### MasterComponent



### MonitorComponent



