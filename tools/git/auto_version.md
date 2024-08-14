---
title: git commit 自动更新版本号
slug: auto_version
categories:
  - git
tags:
  - version
halo:
  site: https://mengplus.top
  name: 0ee3f9eb-1b4e-40a2-9f87-4e5470bcdb51
  publish: true
---
# git commit 自动更新版本号
## 前言
git 有个hook可以在提交前做一些特殊动作，在`.git\hooks`可以看到各种脚本示例，本文使用`pre-commit`在提交之前做版本更新操作
## 操作过程
1. 创建文件在`.git\hooks\pre-commit`,无任何后缀

2. 打开pre-commit文件导入如下脚本信息

   ```bash
   #!/bin/sh
   #
   # An example hook script to verify what is about to be committed.
   # Called by "git commit" with no arguments.  The hook should
   # exit with non-zero status after issuing an appropriate message if
   # it wants to stop the commit.
   #
   # To enable this hook, rename this file to "pre-commit".
   # 获取最新的Git提交日期
   latest_commit_date=$(git log -1 --format=%cd --date=format:%Y%m%d)
   # 获取最新的标签
   latest_tag=$(git describe --tags --abbrev=0 2>/dev/null || echo "V1.00")
   # 指定版本文件
   VERSION_PATH="version.h"
   # 将日期字符串替换到源代码中的版本号定义
   sed -i "s/#define SOFT_VER .*/#define SOFT_VER \"$latest_tag $latest_commit_date\"/" $VERSION_PATH
   # 将变更后的文件添加到提交
   git add $VERSION_PATH
   echo "update version $latest_tag $latest_commit_date"
   ```

3. 创建版本文件"version.h"

   ```c++
   #ifndef _VERSION_H_
   #define _VERSION_H_

   #define SOFT_VER "V1.00 20240814"
   #endif
   ```

4. 执行演示

   ```bash
   $ git commit --amend --no-edit
   warning: in the working copy of 'version.h', LF will be replaced by CRLF the next time Git touches it
   updata version V1.00 20240814
   [master 6c7bc66] ✨ feat(ui_P00error): 完成错误码展示
    Date: Wed Aug 14 12:37:21 2024 +0800
    10 files changed, 549 insertions(+), 1655 deletions(-)

   ```



