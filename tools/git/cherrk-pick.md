---
title: cherry-pick挑拣提交
slug: cherrk-pick
categories:
  - git
tags: []
halo:
  site: https://mengplus.top
  name: f4e9bcda-517e-4c89-94d0-ee81254cdb03
  publish: true
---
# cherry-pick挑拣提交

> 原文链接：https://blog.csdn.net/muzidigbig/article/details/122321393

对于多分支的代码库，将代码从一个分支转移到另一个分支是常见需求。

这时分两种情况：
        一种情况是，你需要另一个分支的所有代码变动，那么就采用合并（git merge）；
        另一种情况是，你只需要部分代码变动（某几个提交），这时可以采用 Cherry pick。

git cherry-pick可以理解为"挑拣"提交，它会获取某一个分支的单笔/多笔提交，并作为一个新的提交引入到你当前分支上。 当我们需要在本地合入其他分支的提交时，如果我们不想对整个分支进行合并，而是只想将某一次提交合入到本地当前分支上，那么就要使用git cherry-pick了。



## 一、基本用法
git cherry-pick命令的作用，就是将指定的提交（commit）应用于其他分支。

```bash
git cherry-pick <commitHash>
```

查看最近三次提交

```bash
$ git log --oneline -3
23d9422 [Description]:branch2 commit 3
2555c6e [Description]:branch2 commit 2
b82ba0f [Description]:branch2 commit 1
```

或者 git push origin 开发分支; 不进行合并

上面命令就会将指定的提交commitHash，应用于当前分支。这会在当前分支产生一个新的提交，当然它们的哈希值会不一样。

举例来说，代码仓库有master和feature两个分支。

    a - b - c - d   Master
         \
           e - f - g Feature
现在将提交f应用到master分支。

切换到 master 分支

```bash
$ git checkout master
```



### Cherry pick 操作
$ git cherry-pick f
上面的操作完成以后，代码库就变成了下面的样子。

     a - b - c - d - f   Master
         \
           e - f - g Feature
从上面可以看到，master分支的末尾增加了一个提交f。

git cherry-pick命令的参数，不一定是提交的哈希值，分支名也是可以的，表示转移该分支的最新提交。

```bash
$ git cherry-pick feature
```

上面代码表示将feature分支的最近一次提交，转移到当前分支。

## 二、转移多个提交

Cherry pick 支持一次转移多个提交。（亲测试ok）

```bash
 $ git cherry-pick <HashA> <HashB>
```

上面的命令将 A 和 B 两个提交应用到当前分支。这会在当前分支生成两个对应的新提交。

如果想要转移一系列的连续提交，可以使用下面的简便语法。

// 不包含A，包含B

```bash
$ git cherry-pick A..B
```

上面的命令可以转移从 A 到 B 的所有提交。它们必须按照正确的顺序放置：提交 A 必须早于提交 B，否则命令将失败，但不会报错。

注意，使用上面的命令，提交 A 将不会包含在 Cherry pick 中，即git cherry-pick (commitidA…commitidB]。如果要包含提交 A，可以使用下面的语法。

// 如果想搞成[]区间，使用 git cherry-pick A^..B 相当于[A B]包含A

```bash
$ git cherry-pick A^..B
```

## 三、配置项

git cherry-pick命令的常用配置项如下。

### （1）-e，--edit

打开外部编辑器，编辑提交信息。

### （2）-n，--no-commit

只更新工作区和暂存区，不产生新的提交。

### （3）-x

在提交信息的末尾追加一行(cherry picked from commit ...)，方便以后查到这个提交是如何产生的。

### （4）-s，--signoff

在提交信息的末尾追加一行操作者的签名，表示是谁进行了这个操作。

### （5）-m parent-number，--mainline parent-number

如果原始提交是一个合并节点，来自于两个分支的合并，那么 Cherry pick 默认将失败，因为它不知道应该采用哪个分支的代码变动。

-m配置项告诉 Git，应该采用哪个分支的变动。它的参数parent-number是一个从1开始的整数，代表原始提交的父分支编号。

```bash
$ git cherry-pick -m 1 <commitHash>
```

上面命令表示，Cherry pick 采用提交commitHash来自编号1的父分支的变动。

一般来说，1号父分支是接受变动的分支（the branch being merged into），2号父分支是作为变动来源的分支（the branch being merged from）。

## 四、代码冲突

如果操作过程中发生代码冲突，Cherry pick 会停下来，让用户决定如何继续操作。

### （1）--continue

用户解决代码冲突后，第一步将修改的文件重新加入暂存区（git add .），第二步使用下面的命令，让 Cherry pick 过程继续执行。

```bash
$ git cherry-pick --continue
```



### （2）--abort

```bash
$ git cherry-pick --abort
```

发生代码冲突后，放弃合并，回到操作前的样子。

### （3）--quit

发生代码冲突后，退出 Cherry pick，但是不回到操作前的样子。

## 五、转移到另一个代码库

Cherry pick 也支持转移另一个代码库的提交，方法是先将该库加为远程仓库。

```bash
$ git remote add target git://gitUrl
```

上面命令添加了一个远程仓库target。

然后，将远程代码抓取到本地。

```bash
$ git fetch target
```

上面命令将远程代码仓库抓取到本地。

接着，检查一下要从远程仓库转移的提交，获取它的哈希值。

```bash
$ git log target/master
```

最后，使用git cherry-pick命令转移提交。

```bash
$ git cherry-pick <commitHash>
```

## 六、常见问题

### 1.The previous cherry-pick is now empty, possibly due to conflict resolution.

原因:
在cherry-pick时出现冲突，解决冲突后本地分支中内容和cherry-pick之前相比没有改变，因此当在以后的步骤中继续git cherry-pick或执行其他命令时，由于此时还处于上次cherry-pick，都会提示该信息，表示可能是由于解决冲突造成上一次cherry-pick内容是空的。

解决方案:
    1.执行git cherry-pick --abort取消上次操作。
    2.执行git commit --allow-empty,表示允许空提交。

### 2.fatal: You are in the middle of a cherry-pick – cannot amend.

原因:
在cherry-pick时出现冲突，没有解决冲突就执行git commit --amend命令，从而会提示该信息。

解决方案:
    首先在git commit --amend之前解决冲突，并完成这次cherry-pick:

```bash
$ git add .
$ git cherry-pick --continue
```

