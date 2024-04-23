---
title: 1Panel本地商店(超多的优秀应用)
categories:
  - 网站运维
tags:
  - 1Panel
halo:
  site: https://mengplus.top
  name: 1ca64c11-6cd0-4f77-958b-a96462af97a5
  publish: true
---

1Panel扩展使用的本地商店仓库，地址[okxlin/appstore: 1Panel 应用商店的非官方应用适配库 1Panel Store Unofficial App Adaptation Repository (github.com)](https://github.com/okxlin/appstore)
非常优秀，同时使用时请甄别

更多细节请见贮藏库介绍

## 2. 使用方式

默认`1Panel`安装在`/opt/`路径下，如果不是按需修改以下。

### 2.1 国内网络

> GitHub加速方式
>> - (本仓库已添加)自建：https://github.com/hunshcn/gh-proxy
>> - https://mirror.ghproxy.com

#### 2.1.1 使用 git 命令获取应用

`1Panel`计划任务类型`Shell 脚本`的计划任务框里，添加并执行以下命令，或者终端运行以下命令，
```shell
git clone -b localApps https://mirror.ghproxy.com/https://github.com/okxlin/appstore /opt/1panel/resource/apps/local/appstore-localApps

cp -rf /opt/1panel/resource/apps/local/appstore-localApps/apps/* /opt/1panel/resource/apps/local/

rm -rf /opt/1panel/resource/apps/local/appstore-localApps
```

然后应用商店刷新本地应用即可。

#### 2.1.2 使用压缩包方式获取应用

`1Panel`计划任务类型`Shell 脚本`的计划任务框里，添加并执行以下命令，或者终端运行以下命令，
```shell
wget -P /opt/1panel/resource/apps/local https://mirror.ghproxy.com/https://github.com/okxlin/appstore/archive/refs/heads/localApps.zip

unzip -o -d /opt/1panel/resource/apps/local/ /opt/1panel/resource/apps/local/localApps.zip

cp -rf /opt/1panel/resource/apps/local/appstore-localApps/apps/* /opt/1panel/resource/apps/local/

rm -rf /opt/1panel/resource/apps/local/appstore-localApps

rm -rf /opt/1panel/resource/apps/local/localApps.zip
```

然后应用商店刷新本地应用即可。

### 2.2 国际互联网络

#### 2.2.1 使用 git 命令获取应用

`1Panel`计划任务类型`Shell 脚本`的计划任务框里，添加并执行以下命令，或者终端运行以下命令，
```shell
git clone -b localApps https://github.com/okxlin/appstore /opt/1panel/resource/apps/local/appstore-localApps

cp -rf /opt/1panel/resource/apps/local/appstore-localApps/apps/* /opt/1panel/resource/apps/local/

rm -rf /opt/1panel/resource/apps/local/appstore-localApps
```

然后应用商店刷新本地应用即可。

#### 2.2.2 使用压缩包方式获取应用

`1Panel`计划任务类型`Shell 脚本`的计划任务框里，添加并执行以下命令，或者终端运行以下命令，
```shell
wget -P /opt/1panel/resource/apps/local https://github.com/okxlin/appstore/archive/refs/heads/localApps.zip

unzip -o -d /opt/1panel/resource/apps/local/ /opt/1panel/resource/apps/local/localApps.zip

cp -rf /opt/1panel/resource/apps/local/appstore-localApps/apps/* /opt/1panel/resource/apps/local/

rm -rf /opt/1panel/resource/apps/local/appstore-localApps

rm -rf /opt/1panel/resource/apps/local/localApps.zip
```

然后应用商店刷新本地应用即可。


## 3. 备注

**未显示在本地应用列表里的，表示未完全适配应用商店面板操作**

**但是支持直接终端运行。**

> 本仓库应用基本支持直接 `docker-compose up` 运行

以`rustdesk`为例

```shell
# 进入 rustdesk 的最新版本目录
cd /opt/1panel/resource/apps/local/rustdesk/versions/latest/

# 复制 .env.sample 为 .env
cp .env.sample .env

# 编辑 .env 文件，修改参数
nano .env

# 启动 RustDesk
docker-compose up -d

# 查看连接所需密钥
cat ./data/hbbs/id_ed25519.pub

```

## 4. 应用一览图

![](https://github.com/okxlin/appstore/raw/localApps/docs/app-list.png)