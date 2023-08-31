# svn

在win平台git-SVN默认是安装的，而linux平台需要手动安装

```bash
sudo apt install  git-svn
```

### 拉取已经存在的SVN仓库

```bash
git svn clone -r5791:HEAD [svn_url] [new_name]
```

r5791为最新的版本号，查看版本号可直接通过浏览器访问svn地址r5791为最新的版本号，查看版本号可直接通过浏览器访问svn地址

new\_name可以给导入的项目取个新的名称，也可以不写，默认和svn名称一致

### 本地提交到SVN

```
 git svn dcommit
```

### 同步远端到本地

```bash
git svn fetch
```

GIT-SVN 可以使用git轻松的访问SVN仓库，方便git对SVN仓库的兼容，如果你是GIT用户又不得不使用SVN管理工程那么这将是不错的选择。

### GIT对比优势

|   | GIT | SVN |
| - | --- | --- |
|   |     |     |
|   |     |     |
|   |     |     |

***

### 下方为未整理资料

如：**git svn clone -r5791:HEAD \[svn\_url] \[new\_name]** ,r5791为最新的版本号，查看版本号可直接通过浏览器访问svn地址，点击右上角的查看历史即可看到版本号信息， 如下图所示

new\_name可以给导入的项目取个新的名称，也可以不写，默认和svn名称一致，输入命令完成后如下图所示：

接下来进入到导入的prop文件夹下查看文件，可以看到文件已经从svn仓库成功导入到了本地仓库

来自 [https://www.cnblogs.com/despacito/p/10239286.html](https://www.cnblogs.com/despacito/p/10239286.html)

git svn dcommit 将本地提交到SVN

git log --graph --oneline 显示日志

**git svn clone -r5791:HEAD \[svn\_url] \[new\_name]**

r5791为最新的版本号

**git commit -am 'commit\_info'** 来把工作区的修改提交到本地仓库，然后通过**git svn dcommit** 命令将本地仓库的修改同步到远程svn仓库

**git-SVN\*\*\*\*安装**

在win平台git-SVN默认是安装的，而linux平台需要手动安装

sudo apt install git-svn

**拉取已经存在的SVN仓库**

git svn clone -r5791:HEAD \[svn\_url] \[new\_name]

r5791为最新的版本号，查看版本号可直接通过浏览器访问svn地址r5791为最新的版本号，查看版本号可直接通过浏览器访问svn地址

new\_name可以给导入的项目取个新的名称，也可以不写，默认和svn名称一致

**本地提交到\*\*\*\*SVN**

git svn dcommit

**同步远端到本地**

git svn fetch
