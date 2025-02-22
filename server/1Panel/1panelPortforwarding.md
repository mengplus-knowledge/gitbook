---
title: 端口转发访问内网
categories:
  - 网站运维
tags:
  - 1Panel
  - 端口转发
  - vscode
halo:
  site: https://mengplus.top
  name: ab47a2f8-4ff2-430c-ba26-5163102aedf8
  publish: true
---
# 端口转发访问内网
## 适用场景
1. 1Panel面板端口被关，无法公网访问
2. 内网部署的服务不想暴露在公网，又想公网访问
3. 防火墙关了端口，又不想放行访问
## 准备条件
1. 服务器的ssh登录正常
2. 安装vscode软件，安装必要的插件remote-SSH
## 操作步骤
通过vscode登录到服务器,如截图所示成功登录终端，再点击旁边的端口添加转发端口

![](https://mengplus.top/upload/sshd.png)
可以看到我已经添加了个端口转发,一个是9876转发到本机的9876端口，他是我的ddns-go服务，又添加一个192.168.100:80设备的网络端口，映射到本机的80，这样我们就能通过访问localhost:9876访问ddns-go像局域网访问效果一样，而且是走的ssh做的端口转发，不受防火墙管控，总之非常好用。

![](https://mengplus.top/upload/portforward.png)
## 注意事项
1. 如果的内网设备服务有重定向，这样将不可用
2. 也可以直接使用cmd命令行使用ssh进行端口转发(自己搜索下吧)
3. 与frp有点相似，但是这是有公网IP情况下，临时搭建的转发通道，断开ssh 端口就不存在了，适合个人临时使用
