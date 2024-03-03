---
title: cloudreve个人网盘部署
categories:
  - 网站运维
tags:
  - 网盘
halo:
  site: https://mengplus.top
  name: 81a5b017-d2a7-4d17-8c32-8e28407fb12c
  publish: true
aliases: 
time: 2024-03-03
---
# cloudreve Pro

## 说明

根据官方资料整理重新打包，基础使用ubuntu:22.04，容器无法直接使用，请参阅下方使用流程操作。具体打包的版本请看tag

参考链接：
1. doc[配置文件 - Cloudreve](https://docs.cloudreve.org/getting-started/config)
2. docker地址[mengplus/cloudreve - Docker Image | Docker Hub](https://hub.docker.com/r/mengplus/cloudreve)


## 启动流程

1. 准备工作文件夹

   注意：根据指引完成启动后，后续上传的文件也都将存储到此文件夹下

   ```bash
   mkdir cloudreve
   cd cloudereve
   mkdir -vp cloudreve/{uploads,avatar} \
   && touch cloudreve/conf.ini \
   && touch cloudreve/cloudreve.db \
   && mkdir -p aria2/config \
   && mkdir -p data/aria2 \
   && chmod -R 777 data/aria2
   
   touch docker-compose.yaml
   ```

   得到的目录树

   ```bash
   ./cloudreve/
   ├── aria2
   │   └── config
   ├── cloudreve
   │   ├── avatar
   │   ├── cloudreve.db #数据库
   │   ├── conf.ini  #配置文件
   │   ├── key.bin #捐赠版的key
   │   └── uploads
   ├── data
   │   └── aria2  #离线下载存储位置
   ├── docker-compose.yaml #docker 配置文件
   ├── key.bin
   ```





2. 官网拿到**key.bin**

   官网地址[授权管理 - Cloudreve](https://cloudreve.org/manage) 如果您非捐赠版，请直接看参考链接使用社区版配置

   将key.bin放入新创建的./cloudreve路径下

3. 准备**docker-compose.yaml**

   ```yaml
   version: "3.8"
   services:
   volumes:
     aria2_swap:
       driver: local
       driver_opts:
         type: none
         device: $PWD/data
         o: bind

     cloudreve:
       labels:
         createdBy: "mengplus"
       container_name: cloudreve
       image: mengplus/cloudreve:pro.3.8.4
       restart: unless-stopped
       network_mode: host
       ports:
         - "40033:5212"
       volumes:
         - aria2_swap:/downloads
         - ./data/uploads:/cloudreve/uploads
         - ./data/avatar:/cloudreve/avatar
         - ./data/conf.ini:/cloudreve/conf.ini
         - ./data/cloudreve.db:/cloudreve/cloudreve.db
         - ./data/key.bin:/cloudreve/key.bin
       depends_on:
         - aria2

     aria2:
       container_name: aria2
       image: p3terx/aria2-pro
       restart: unless-stopped
       network_mode: host
       environment:
         - PUID=65534
         - PGID=65534
         - UMASK_SET=022
         - RPC_SECRET=aria2-pro
         - RPC_PORT=6800
         - LISTEN_PORT=6888
         - DISK_CACHE=64M
         - IPV6_MODE=true
         - UPDATE_TRACKERS=true
         - CUSTOM_TRACKER_URL=
         - TZ=Asia/Shanghai
       volumes:
         - ./aria2/config:/config
         - aria2_swap:/downloads
   ```

4. 运行

   这里使用脚本快捷初始化，请docker-compose.yaml

   ```bash
   docker-compose up -d #后台执行
   #当然也有其它操作
   docker-compose down #停止，并且移除容器
   docker-compose stop #停止
   
   ```





Dockerfile

```dockerfile
# cloudreve Pro
# version 3.8.4
# docker pull mengplus/cloudreve

FROM ubuntu:22.04

 WORKDIR /cloudreve
 COPY ./prebuild/cloudreve ./cloudreve
# buildkit
RUN sed -i "s@http://.*archive.ubuntu.com@http://repo.huaweicloud.com@g" /etc/apt/sources.list \
	&& sed -i "s@http://.*security.ubuntu.com@http://repo.huaweicloud.com@g" /etc/apt/sources.list \
    # && apt-get update -y \
    # &&  apt-get install -y  tzdata \
    #&& cp /usr/share/zoneinfo/Asia/Shanghai /etc/localtime    \
    && echo "Asia/Shanghai" > /etc/timezone \
    && apt-get -y autoremove \
    && apt-get -y clean \
    && chmod +x ./cloudreve
   # buildkit

 EXPOSE 5212/tcp
 VOLUME ["/cloudreve/uploads" ,"/cloudreve/avatar"]
 ENTRYPOINT ["./cloudreve"]

ENV LANG=en_US.UTF-8 LANGUAGE=en_US.UTF-8 LC_ALL=en_US.UTF-8

```





