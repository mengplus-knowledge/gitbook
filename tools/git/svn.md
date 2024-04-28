---
title: 用git管理svn仓库
categories:
  - git
tags:
  - git
halo:
  site: https://mengplus.top
  name: f65bd8e5-e503-4b00-bc03-e77d9310233f
  publish: true
---

# 用git管理svn仓库
GIT-SVN 可以使用git轻松的访问SVN仓库，方便git对SVN仓库的兼容，如果你是GIT用户又不得不使用SVN管理工程那么这将是不错的选择。
## 环境安装

在win平台git-SVN默认是安装的，而linux平台需要手动安装

```bash
sudo apt install  git-svn
```

## 1. 拉取已经存在的SVN仓库

```bash
git svn clone -r5791:HEAD [svn_url] [new_name]
```
![](https://img2018.cnblogs.com/blog/1499025/201901/1499025-20190108104314032-340329377.png)
r5791为最新的版本号，查看版本号可直接通过浏览器访问svn地址r5791为最新的版本号，查看版本号可直接通过浏览器访问svn地址

new\_name可以给导入的项目取个新的名称，也可以不写，默认和svn名称一致
![](https://img2018.cnblogs.com/blog/1499025/201901/1499025-20190108104759300-1484051819.png)
## 2. 本地提交到SVN

```
 git  add .                   #添加所有改动到暂存区
 git commit -am 'commit_info' #提交信息到本地仓库
 git svn dcommit              #将本地仓库的修改同步到远程svn仓库
```

## 3. 同步远端到本地

```bash
git svn fetch  #拉取不改变本地工作区与分支
git svn rebase #拉取并变基到最新分支
```



## GIT与SVN对比
1. git是分布式，拉取仓库即获得仓库所有提交记录，离线不影响使用；SVN集中式，服务器离线本地无法做任何关于仓库的操作
2. git是文本管理，最小精细到一行；SVN是文件为单位；因此SVN的占用量更大
3. git可以同时推送到多个仓库；SVN只能切换一个仓库的路径
4. git支持多分支与合并，SVN不擅长分支，分支成本高，合并不好用
5. git主要应用于文本类型工作者使用，方便差异比对；SVN适合二进制文件工程管理
6. git一般有友好的服务器端，而SVN服务器端较为单一
7. git支持svn仓库管理；SVN不支持git仓库，但是有工具可以将git当SVN仓库风格使用


