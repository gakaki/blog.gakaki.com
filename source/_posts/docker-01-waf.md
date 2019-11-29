---
title: Docker 安装 OpenResty Daocloud国内加速 使用Airpline最小镜像 最速指南 (简单防火墙指南1)
date: 2017-04-13 09:00
categories:
- openresty
tags:
- docker
- waf
---

## Docker的安装
### 版本要求: 
Docker 17以上的版本,可以安装在 windows 10 , OSX 10.12, LINUX 4.X系统之上 ( Ubuntu 16.04, Centos 6.4 以上)

#### Windows 安装
[官网最新beta下载 windows](https://www.docker.com/community-edition)
#### MAC 安装

如果想在mac下直接使用Docker的话,其实最大用处就是用来做为开发环境了.
MySQL和Openresty这样的比较适合,如果是使用JAVA Spring boot 这样的每次编译丢入Docker这样的,方案实在是效率不高啊.

装ElasticSearch ELK方案是很适合的,甚至搭配docker compose之类的
集群方案可以很快复制出集群.
mac和windows下推荐使用官方的beta版最新的exe和dmg,经测试使用daocloud下载的exe dmg无法使用daocloud加速器,不知为何,官方最新beta无问题.
PS: MAC和windows的Docker要比vm和virtualbox方案要消耗内存要低的多,启动速度也要快的多,原理不同,并非使用虚拟机方案. 开发环境使用docker的秘诀就是使用-v 参数映射文件路径,并设置网络端口映射.
官网最新edge channel 下载 mac
记得用edge channel 下载很慢建议翻墙吧..下完拖到'应用程序' ,然后docker就多了个小海豚.
之后命令行docker 就可以玩起来了.
#### 设置daocloud 为hub.docker.com 加速mirrors
 docker菜单 -> "Preferences" -> File Sharing 里设置可以挂载的目录 
 docker菜单 -> "Preferences" -> Daemon 里设置Daocloud的加速镜像地址 
 保存save reset docker
Linux Ubuntu 安装
从中国国内各种镜像站点下载ubuntu或者centos的iso 然后搭配迅雷下载 速度很快.
这里使用最新ubuntu server(640m)系统在VMWARE或者Virtualbox中使用(为啥不是centos 因为体积大 4.3GB)
https://mirrors.aliyun.com/ubuntu-releases/16.10
ps:17.04beta应该也可以
之后vmware或者virtualbox创建虚拟机安装ubuntu系统吧 512m内存足以了 很低功耗的docker ...
装完ubuntu,增加ubuntu apt更新源为国内镜像
sudo mv /etc/apt/sources.list /etc/apt/sources.list.bak #备份系统默认的软件源
sudo vi /etc/apt/sources.list
 
#如果是 16.10 版本 阿里云源
 
deb http://mirrors.aliyun.com/ubuntu/ yakkety main restricted
deb http://mirrors.aliyun.com/ubuntu/ yakkety-updates main restricted
deb http://mirrors.aliyun.com/ubuntu/ yakkety universe
deb http://mirrors.aliyun.com/ubuntu/ yakkety-updates universe
deb http://mirrors.aliyun.com/ubuntu/ yakkety multiverse
deb http://mirrors.aliyun.com/ubuntu/ yakkety-updates multiverse
deb http://mirrors.aliyun.com/ubuntu/ yakkety-backports main restricted universe multiverse
deb http://mirrors.aliyun.com/ubuntu/ yakkety-security main restricted
deb http://mirrors.aliyun.com/ubuntu/ yakkety-security universe
deb http://mirrors.aliyun.com/ubuntu/ yakkety-security multiverse
 
# 保存完毕
 
sudo apt update
sudo apt upgrade
 
#如果是17.04 版本 阿里云源 其实就是将17.04的代号英文 替换为16.10的英文字符就可以了
 
deb http://mirrors.aliyun.com/ubuntu/ zesty main restricted
deb http://mirrors.aliyun.com/ubuntu/ zesty-updates main restricted
deb http://mirrors.aliyun.com/ubuntu/ zesty universe
deb http://mirrors.aliyun.com/ubuntu/ zesty-updates universe
deb http://mirrors.aliyun.com/ubuntu/ zesty multiverse
deb http://mirrors.aliyun.com/ubuntu/ zesty-updates multiverse
deb http://mirrors.aliyun.com/ubuntu/ zesty-backports main restricted universe multiverse
deb http://mirrors.aliyun.com/ubuntu/ zesty-security main restricted
deb http://mirrors.aliyun.com/ubuntu/ zesty-security universe
deb http://mirrors.aliyun.com/ubuntu/ zesty-security multiverse
Docker安装篇
官网的docker其实下载很慢,这里推荐Daocloud下载镜像点.先注册个Daocloud账号,否则后面daocloud加速器没法用,此加速器自带监控功能.
curl -sSL https://get.daocloud.io/docker | sh
PS: 此脚本会加入apt源 并且 apt-update 所以不用手动再apt update了..
安装完毕! docker -v 看下版本
使用Daocloud加速Docker国内速度
大家都知道的 如果用官方 hub.docker.com里搜索的docker镜像简直慢的无语,可能比你单独下载在自己安装一套还慢 ,国内有很多第三方docker镜像,这里就用DaoCloud了大部分官方镜像都有缓存,没有的就挂docker代理去官网下吧.当然最终的出路是每家公司必然会做自己的Docker Registry(可以理解为 Docker 自建仓库),否则没有好的网络真的太慢,对于频繁更新Docker镜像的其实也更适合使用内网 Docker Registry.
PS:加速器功能需要注册DaoCloud
加速器地址 有mac win linux 设置教程
linux 如下
curl -sSL https://get.daocloud.io/daotools/set_mirror.sh | sh -s http://7f5f96fe.m.daocloud.io  
尝试下载一个ubuntu docker镜像
docker pull ubuntu
看看速度起来了没
完


