---
description: 介绍repo在嵌入式中的应用
---
# repo
## 参考资料
1. [Google Git-Repo 多仓库项目管理](https://zhuanlan.zhihu.com/p/50564255)
2. [docker · OpenHarmony/docs - 码云 - 开源中国 (gitee.com)](https://gitee.com/openharmony/docs/tree/master/docker)
3. [Docker编译环境](https://gitee.com/openharmony/docs/blob/master/zh-cn/device-dev/get-code/gettools-acquire.md)
4. [OpenHarmony/manifest (gitee.com)](https://gitee.com/openharmony/manifest)
2. [docker · OpenHarmony/docs - 码云 - 开源中国 (gitee.com)](https://gitee.com/openharmony/docs/tree/master/docker)
```bash
3. [Docker编译环境](https://gitee.com/openharmony/docs/blob/master/zh-cn/device-dev/get-code/gettools-acquire.md)
curl -s https://gitee.com/oschina/repo/raw/fork_flow/repo-py3 > /usr/local/bin/repo  #如果没有权限，可下载至其他目录，并将其配置到环境变量中chmod a+x /usr/local/bin/repo
4. [OpenHarmony/manifest (gitee.com)](https://gitee.com/openharmony/manifest)
pip3 install -i https://repo.huaweicloud.com/repository/pypi/simple requests


```bash
#示例
curl -s https://gitee.com/oschina/repo/raw/fork_flow/repo-py3 > /usr/local/bin/repo  #如果没有权限，可下载至其他目录，并将其配置到环境变量中chmod a+x /usr/local/bin/repo
repo init -u git@gitee.com:openharmony/manifest.git -b master -g ohos:mini
pip3 install -i https://repo.huaweicloud.com/repository/pypi/simple requests
```
#示例
repo init -u git@gitee.com:openharmony/manifest.git -b master -g ohos:mini
```