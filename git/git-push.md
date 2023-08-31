# git push

## 参考资料

1. 官方文档[Git - git-push Documentation (git-scm.com)](https://git-scm.com/docs/git-push/zh_HANS-CN)

2. [git push 命令 | 菜鸟教程 (runoob.com)](https://www.runoob.com/git/git-push.html)

## 操作演示

### 命令格式

部分命令参数可以省略

```bash
$git <远端名称> <本地分支>:<远程分支>
```

```bash
# 完整的命令格式
git push [--all | --branches | --mirror | --tags] [--follow-tags] [--atomic] [-n | --dry-run] [--receive-pack=<git-receive-pack>]
	   [--repo=<仓库>] [-f | --force] [-d | --delete] [--prune] [-v | --verbose]
	   [-u | --set-upstream] [-o <字符串> | --push-option=<字符串>]
	   [--[no-]signed|--signed=(true|false|if-asked)]
	   [--force-with-lease[=<引用名>[:<expect>]] [--force-if-includes]]
	   [--no-verify] [<仓库> [<引用规范>…​]]
```



### 推送本分支到默认仓库

```bash
$git push origin main:main  #推送main分支到远端origin 的main分支
$git push 					#可简写为
$git push -f 				#-f 可选参数强制推送，会覆盖远端仓库的分支
```

### 其他示例

```bash
$ git push origin develop:master -f #把本地的develop分支强制 (-f)推送到远端master。

```

