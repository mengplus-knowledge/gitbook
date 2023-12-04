# docker笔记

## 参考资料

1. [docker仓库](hub.docker.com)
2. [docker官方手册](https://docs.docker.com/)
3. [菜鸟教程](https://www.runoob.com/docker/docker-tutorial.html)

## 章节目录

* [docker](docker.md)
* [container](container.md)
* [docker-compose](docker-compose.md)
* [docker-machine](docker-machine.md)

## 简介

## 名词解释

1. image镜像
2. container容器

## 安装与配置

1. 安装docker ubuntu环境

   ```bash
   # 安装docker环境
   sudo apt install docker.io docker-compose
   ```

2. 当前用户加入docker用户组

   ```bash
   # 检查是否已经在用户组
   $ id
   uid=1000(mengplus) gid=1000(mengplus) groups=1000(mengplus),4(adm),24(cdrom),27(sudo),30(dip),46(plugdev),110(lxd)
   # 不存在docker用户组，继续检查是否存在docker用户组
   # 装好docker一般就已经存在docker组
   $ cat /etc/group
   tss:x:116:
   landscape:x:117:
   fwupd-refresh:x:118:
   mengplus:x:1000:
   docker:x:119:
   #发现已经存在用户组docker直接添加当前用户到docker组
   $ sudo gpasswd -a $ {USER} docker
   ```
