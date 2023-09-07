layout: 介绍如何维护使用这个gitbook仓库
---

# 开发与部署

# 开发环境

1. 编写使用markdown
2. 编译环境billryan/gitbook
3. 开发推荐使用vscode typora

## 安装环境

```bash
#安装docker 和compose
$ sudo apt install docker.io docker-compose
#拉取docker images文件，也就是开发环境
$ docker pull billryan/gitbook
#也可以不用手动拉取images,./gitbook路径下，启动compose将自动拉取并启动服务
$ docker-compose up -d
```

## 演示指令

```bash
# init
docker run --rm -v "$PWD:/gitbook" -p 4000:4000 billryan/gitbook gitbook init
# serve
docker run --rm -v "$PWD:/gitbook" -p 4000:4000 billryan/gitbook gitbook serve
# build
docker run --rm -v "$PWD:/gitbook" -p 4000:4000 billryan/gitbook gitbook build
```

## gitbook 工程结构介绍

SUMMARY.md 为整个工程的目录结构管理文件，通过 init命令更新整个工程结构

./README.md 工程的第一页

## 开发流程

1. 启动 serve vscode提示打开`http://localhost:4000/`地址进行本地预览
2. 在vscode正常变基md文档，文档更新则自动编译，刷新网页即可预览
    ``` bash
    Stopping server
    info: 7 plugins are installed
    info: loading plugin "livereload"... OK
    info: loading plugin "highlight"... OK
    info: loading plugin "search"... OK
    info: loading plugin "lunr"... OK
    info: loading plugin "sharing"... OK
    info: loading plugin "fontsettings"... OK
    info: loading plugin "theme-default"... OK
    info: found 7 pages
    info: found 3 asset files
    info: >> generation finished with success in 6.4s !

    Starting server ...
    Serving book on http://localhost:4000
    ```
3. 第一次使用需要根据book.json安装插件，或者直接移除此文件可跳过此步骤
启动容器后使用指令
```bash
#进入gitbook容器
docker exec -it gitbook bash
#安装gitbook插件,
gitbook install

```
如果部分失败，可以将失败的插件暂时移除,操作方式 book.json中plugins中插件前方添加`-`即是移除
```json
    "plugins": [
        "-sharing",
        "sharing-plus",
        "include-codeblock",
        "ace",
        "-lunr",
        "-search",
        "search-pro",
        "collapsible-menu",
        "-bibtex-indexed-cite",
        "-livereload",
        "-highlight",
        "url-embed",
        "page-footer-ex"
    ],
```
