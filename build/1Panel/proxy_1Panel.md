---
title: 1Panel中如何快速配置TCP代理
categories:
  - 网站运维
tags:
  - 1Panel
  - OpenResty
  - proxy
halo:
  site: https://mengplus.top
  name: 54abc156-4a58-406f-8497-ed133f4a3275
  publish: true
---
# 1Panel快速配置TCP代理
## 场景需求描述
我有多个服务器，但是只有一个对外服务器，并且部署了1Panel,想通过代理实现内网服务器的访问功能。
## 前提条件
1. 公网服务器安装了1Panel
2. 1Panel安装了OpenResty
3. 内网服务器需要映射222端口到公网服务器的1222端口
## 操作步骤
 打开登录公网服务器1Panel,网站>网站>设置>配置修改
 添加如下配置保存即可
 ```bash

stream {
    upstream rtmp {
        server rh1288.mengplus.top:222;
    }
    server {
        listen 1222;
        proxy_pass rtmp;
        proxy_timeout 120s;
    }
}

```

![](https://mengplus.top/upload/proxy_tcp.png)
