---
title: git 自动清理远端不存在的分支
slug: prune_origin
cover: ""
categories:
  - git
tags: []
halo:
  site: https://mengplus.top
  name: af2eda09-1186-433f-89a0-b80674b74a23
  publish: true
---
# git 自动清理远端不存在的分支

```bash
git remote prune origin
```
用如下命令查看远程仓库信息：

```bash
git remote show origin
```
查看所有分支
```bash
git branch --all
```