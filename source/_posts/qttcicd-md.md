---
title: 自研持续交付系统的流程
date: 2018-10-28 19:20:18
categories:
- devops
tags:
- CICD
---
    
持续交付系统可分为2个阶段：持续集成CI（生成制品） 和 持续部署CD（制品部署），辅以管理平台审核流程,记录和查看数据。
![我的应用](https://midu.studio515.cn/2019_12_cicd/0.png)
![我的应用](https://midu.studio515.cn/2019_12_cicd/1.png)

## CI的流程

服务器端程序流程
例如：Golang
GO Mod 模式下 GitLab提交代码，Gitlab或Jenkins go build 编译，生成Docker产物包push至harboar保存。 
非容器化的打包成zip发送到CDN.
以上操作都记录进产物数据库保存历史信息。

App的流程
  ios 安装黑苹果主机,Jenkins Cocospods + FastLane 的常用脚本进行编译即可。注意点： 让cocopods的依赖扁平化，不然会发生可怕的依赖冲突，或者始终保持最新版本组件的兼容。
有的组件二进制化可以大幅减少编译时间。编译完毕ipa，可传送到蒲公英和TestFlight，这里管理平台予以记录。
Android不太熟悉，预计也是使用Gradle和二进制包 进行集成，之后进行cdn分发和发布到各大市场。

## CD的流程

服务器端和h5的流程

![上线单](https://midu.studio515.cn/2019_12_cicd/2.png)
![上线单填写](https://midu.studio515.cn/2019_12_cicd/4.png)

进入上线管理平台，填写上线单（留下你的上线记录），测试或生产。上线单需要记录下对应的产物版本（tag版本，对应的需求系统里的需求例如Tapd）
流程如下：
1 从cmdb（运维的配置管理数据库，存储公司的服务器数据库项目等信息）获得对应的 项目的服务器信息
2 根据容器（阿里云k8s上docker容器的重启）和非容器化的方式进行部署（ansible或supervisor执行重启程序），若是h5程序会从ci流程里拿到编译后的React Vue webpack压缩包部署到CDN或者使用Ansbile部署解压缩到指定服务器的目录。

## 错误
![错误回滚](https://midu.studio515.cn/2019_12_cicd/3.png)
若发生程序错误，可进行回滚到上一个版本。

## 统计
![统计](https://midu.studio515.cn/2019_12_cicd/5.png)
![统计](https://midu.studio515.cn/2019_12_cicd/6.png)
![统计](https://midu.studio515.cn/2019_12_cicd/7.png)
另外需要提供详细的 cd统计数据，消耗时间，状态次数统计

## 提醒
![统计](https://midu.studio515.cn/2019_12_cicd/8.png)
完成操作之后的对应提醒，可以放入企业im工具 或微信提醒

