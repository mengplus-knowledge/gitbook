---
title: debian10安装exfat驱动
slug: debian10an-zhuang-exfatqu-dong
cover: ""
categories:
  - linux
tags:
  - exfat
halo:
  site: https://mengplus.top
  name: b90cebfc-5614-4f38-960a-42f105efe536
  publish: true
---
Linux 内核版本（4.19.232）未内置 exfat 模块。exFAT 的内核支持是从 Linux 5.4 版本开始引入的，而你的内核版本较旧，因此无法加载 exfat 模块。当然你也可以手动更新内核 
或者像我一样按照下面的脚本一键操作 `exfatprogs`在debian11中已经默认支持
```bash
echo "deb http://mirrors.ustc.edu.cn/debian bullseye main" | tee /etc/apt/sources.list.d/bullseye.list
apt update
apt install -y  exfat-fuse 
apt-get install -t bullseye exfatprogs
rm /etc/apt/sources.list.d/bullseye.list

apt-get update
```
如果遇到exfat分区受损修复，debian 10 版本较老 需要装exfatprogs驱动 再执行修复功能，否则`fsck.exfat -y "$dev"`执行后 无法成功修复
