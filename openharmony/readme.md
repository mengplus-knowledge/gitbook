---
description: 鸿蒙系统学习笔记，记录入门
title: openharmony笔记
slug: openharmony
cover: ""
categories:
  - openharmony
tags: []
halo:
  site: https://mengplus.top
  name: 7c2a2182-0669-401a-b0f3-fb8ac7016716
  publish: true
---

# openharmony笔记

## 开发环境准备
1. 源码获取
  参考链接：[docs: OpenHarmony documentation | OpenHarmony开发者文档 (gitee.com)](https://gitee.com/openharmony/docs)
  主线仓库：[OpenHarmony/manifest (gitee.com)](https://gitee.com/openharmony/manifest)
  推荐使用最新master仓库
  操作步骤：
  	1. 注册gitee账户
      	1. 步骤略
  2. git 安装与配置

     1. 操作步骤略

  3. 本地生成ssh并配置到gitee账户下

     1. 操作步骤略

  4. repo 一键安装

     ```
     curl -s https://gitee.com/oschina/repo/raw/fork_flow/repo-py3 > /usr/local/bin/repo  #如果没有权限，可下载至其他目录，并将其配置到环境变量中chmod a+x /usr/local/bin/repo
     pip3 install -i https://repo.huaweicloud.com/repository/pypi/simple requests
     ```

  5. 源码获取

     - 通过**repo+git +ssh**下载。

     从版本分支获取源码。可获取该版本分支的最新源码，包括版本发布后在该分支的合入。

     ```
     repo init -u git@gitee.com:openharmony/manifest.git -b master --no-repo-verify
     repo sync -c
     repo forall -c 'git lfs pull'
     ```

     从版本发布Tag节点获取源码。可获取与版本发布时完全一致的源码。

     ```
     repo init -u git@gitee.com:openharmony/manifest.git -b refs/tags/OpenHarmony-v4.1-Release --no-repo-verify
     repo sync -c
     repo forall -c 'git lfs pull'
     ```

     - 通过**repo +git + https** 下载。

     从版本分支获取源码。可获取该版本分支的最新源码，包括版本发布后在该分支的合入。

     ```
     repo init -u https://gitee.com/openharmony/manifest -b OpenHarmony-4.1-Release --no-repo-verify
     repo sync -c
     repo forall -c 'git lfs pull'
     ```

     从版本发布Tag节点获取源码。可获取与版本发布时完全一致的源码。

     ```
     repo init -u https://gitee.com/openharmony/manifest -b refs/tags/OpenHarmony-v4.1-Release --no-repo-verify
     repo sync -c
     repo forall -c 'git lfs pull'
     ```

     **tips:** 两者差异，首先ssh走的tcp 22端口，https走的https 443端口，ssh更加稳定可靠

1. 开发环境搭建

   1. 从仓库的build中一键安装环境
   2. 通过docker镜像启用环境
