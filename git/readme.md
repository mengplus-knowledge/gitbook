# git使用笔记

## 参考资料

1. 官方下载：[Git - Downloads (git-scm.com)](https://git-scm.com/downloads)

2. 官方文档：[Git - 安装 Git (git-scm.com)](https://git-scm.com/book/zh/v2/起步-安装-Git)

3. 推荐客户端列表[Git - GUI Clients (git-scm.com)](https://git-scm.com/downloads/guis)

4. 第三方的推荐资料[Git 教程 | 菜鸟教程 (runoob.com)](https://www.runoob.com/git/git-tutorial.html)

   ------

   

# 下载与安装

请参阅 参考资料 1.官方下载 安装软件

# 功能配置

为避免不同平台的对文档理解的差异，只通过命令行进行必要的 命令演示。

0. 前提环境

1. 打开命令行 输入 **git** 可以看到 命令介绍，则可证明软件安装正常，有很多可选参数，根据需要自行查阅了解

   ```shell
   # 测试命令
   $ git 
   usage: git [-v | --version] [-h | --help] [-C <path>] [-c <name>=<value>]     
              [--exec-path[=<path>]] [--html-path] [--man-path] [--info-path]    
              [-p | --paginate | -P | --no-pager] [--no-replace-objects] [--bare]
              [--git-dir=<path>] [--work-tree=<path>] [--namespace=<name>]       
              [--super-prefix=<path>] [--config-env=<name>=<envvar>]
              <command> [<args>]
   
   ```

   

## 配置用户信息

参考资料[Git - git-config Documentation (git-scm.com)](https://git-scm.com/docs/git-config/zh_HANS-CN)

1. 配置账户

   配置用户信息(必要) ,可随意填写，非账户密码 可以多次使用此命令重新配置

   ```
   $ git config --global user.name "myname"  #用户名
   $ git config --global user.email myemail@outlook.com #邮箱 用户联系方式
   ```

​	选项

​	 **--global** 

​		将本次配置列为全局配置，如果新仓库中未再次配置，则使用此默认配置,配置项将修改存放在用户目录下的 ~/.gitconfig也可直接打开此文本查看修改

2. 查看所有配置

查看当前的配置情况，根据需要可二次修改

```bash
$　git config --list
```

## 创建仓库
