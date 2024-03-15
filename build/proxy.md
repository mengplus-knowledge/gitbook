---
title: proxy代理宿主机与实例
categories:
  - 网站运维
tags: []
halo:
  site: https://mengplus.top
  name: e2d42985-11bd-433e-8abd-6248bac7c6ce
  publish: true
---
# proxy代理宿主机与实例


## 说明

本笔记记录的使用clash配置代理的方式，由于使用的是服务器，因此只有命令行操作，除了宿主机配置还有gitea容器想使用代理方便镜像github的配置方案，请对应学习使用。如果有更合适的方案欢迎交流。

## 参考链接

1. https://www.clash.la/releases/
2. 配置链接https://clash.razord.top/#/proxies

## 软件获取与安装

 在参考链接1中获取最新版安装包，当前为clash-linux-amd64-v1.18.0 由于作者跑路，估计这就是最总版本了。拿到安装包为方便运行将其放入bin路径，并添加到系统自启。

### 软件获取与安装

```bash
#演示链接若失效自行寻找
mkdir ~/clash && cd ~/clash
wget https://down.clash.la/Clash/Core/Releases/clash-linux-amd64-v1.18.0.gz
gunzip clash-linux-amd64-v1.18.0.gz
sudo cp clash-linux-amd64-v1.18.0  /usr/local/bin/clash
sudo chmod +x /usr/local/bin/clash
```

### 获取clash配置文件

这里用的文件为各自平台获得，请根据指导获得配置文件

```bash
sudo  mkdir /etc/clash #创建配置clash的文件夹
#将得的config.yaml文件放入路径下
```

### 配置clash自启

```bash
sudo vim /etc/systemd/system/clash.service
# 放下面的配置文件下去 保存退出
[Unit]
Description=Clash Daemon

[Service]
ExecStart=/usr/local/bin/clash -d /etc/clash/
Restart=on-failure

[Install]
WantedBy=multi-user.target
```

启动clash和自启

```bash
sudo systemctl daemon-reload #重新加载 systemd 模块
sudo systemctl start clash.service #启动clash
sudo systemctl enable clash.service #设置Clash开机自启动

```

以下为Clash相关的管理命令

```bash
## 启动Clash ##
sudo systemctl start clash.service

## 重启Clash ##
sudo systemctl restart clash.service

## 查看Clash运行状态 ##
sudo systemctl status clash.service

## 实时滚动状态 ##
sudo journalctl -u clash.service -f
```

## 配置宿主机代理

自此完成了 clash的配置，接下来只剩配置宿主机代理和容器代理,在clash的配置文件<u>config.yaml</u>中规定了代理端口，比如我的是**127.0.0.1:7890**

新建proxy.sh文件，填入下面的内容，放入/etc/profile.d/proxy.sh路径便会被默认加载成为全局代理，如果不想使用就从此处移除

```bash
# set proxy config via profie.d - should apply for all users
#
export http_proxy="http://127.0.0.1:7890/"
export https_proxy=$http_proxy
export ftp_proxy=$http_proxy
export no_proxy="127.0.0.1,localhost"

# For curl
export HTTP_PROXY=$http_proxy
export HTTPS_PROXY=$http_proxy
export FTP_PROXY=$http_proxy
export NO_PROXY="127.0.0.1,localhost"
```
添加完毕之后想要立刻生效做如下操作
```bash
source /etc/profile
```
有效性验证
```bash
echo $http_proxy
#回应 http://127.0.0.1:7890 与配置一致。则配置生效
curl www.google.com
##回应下文 则代理运行正常
<HTML><HEAD><meta http-equiv="content-type" content="text/html;charset=utf-8">
<TITLE>302 Moved</TITLE></HEAD><BODY>
<H1>302 Moved</H1>
The document has moved
<A HREF="http://www.google.com.hk/url?sa=p&amp;hl=zh-CN&amp;pref=hkredirect&amp;pval=yes&amp;q=http://www.google.com.hk/&amp;ust=1706175322886575&amp;usg=AOvVaw1iCqrexpYT503DrB4t0o93">here</A>.
</BODY></HTML>
```

如果仅仅想对当前用户有效，就放入用户文件下的 ~/.bashrc文件里，如果只想在当前终端有效，直接执行此文件即可
## 为docker配置代理
让容器采用宿主机代理，首先看下容器与宿主机网络关系，bridge还是host,其中host应该无需再配置，我未测试，下面说bridge模式下如何配置。
我的gitea挂在了1panel-network 网络，网关也就是宿主机地址是172.25.0.0.1,gitea的地址是172.25.0.3/16,gitea容器访问网关既是宿主机。
另外记得放行宿主机7890端口，否则容器访问不到
在gitea容器中，准备配置文件proxy.sh,填入下文内容

```bash
# set proxy config via profie.d - should apply for all users
#
export http_proxy="http://172.25.0.1:7890"
export https_proxy=$http_proxy
export ftp_proxy=$http_proxy
export no_proxy="127.0.0.1,localhost"

# For curl
export HTTP_PROXY=$http_proxy
export HTTPS_PROXY=$http_proxy
export FTP_PROXY=$http_proxy
export NO_PROXY="127.0.0.1,localhost"
```
将文件加入开机启动即可，我使用的gitea.cn/gitea/gitea:1.21.4  容器启动入口程序是"/usr/bin/entrypoint" ,在里面加入source /etc/profile.d/proxy.sh即可，注意路径。

为什么我不像宿主机一样的配置呢？ 理论上是可以的，但是这个容器使用的系统是"Alpine Linux v3.18",我未找到其他的能够自动调用的地方配置完毕后，重启容器 再次进入容器测试。

### 为docker-compose配置代理
请看 environment 配置
```yaml
networks:
    1panel-network:
        external: true
services:
    gitea:
        container_name: ${CONTAINER_NAME}
        deploy:
            resources:
                limits:
                    cpus: ${CPUS}
                    memory: ${MEMORY_LIMIT}
        environment:
            - USER_UID=1000
            - USER_GID=1000
            - GITEA__database__DB_TYPE=${PANEL_DB_TYPE}
            - GITEA__database__HOST=${PANEL_DB_HOST}:${PANEL_DB_PORT}
            - GITEA__database__NAME=${PANEL_DB_NAME}
            - GITEA__database__USER=${PANEL_DB_USER}
            - GITEA__database__PASSWD=${PANEL_DB_USER_PASSWORD}
            - http_proxy=http://172.25.0.1:7890
            - https_proxy=http://172.25.0.1:7890
            - ftp_proxy=http://172.25.0.1:7890
            - no_proxy="127.0.0.1,localhost"
            - HTTPS_PROXY=http://172.25.0.1:7890
            - HTTP_PROXY=http://172.25.0.1:7890
            - NO_PROXY="127.0.0.1,localhost"

        image: gitea.cn/gitea/gitea:1.21.7
        labels:
            createdBy: Apps
        networks:
            - 1panel-network
        ports:
            - ${HOST_IP}:${PANEL_APP_PORT_HTTP}:3000
            - ${HOST_IP}:${PANEL_APP_PORT_SSH}:22
        restart: always
        volumes:
            - /mnt/b/1panel/apps/gitea/gitea/data:/data
            - /etc/timezone:/etc/timezone:ro
            - /etc/localtime:/etc/localtime:ro
version: "3"
```
