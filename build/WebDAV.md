---
title: WebDAV
slug: webdav
cover: ""
categories:
  - 网站运维
tags: []
halo:
  site: https://mengplus.top
  name: 9df8ef27-d13d-408c-8a45-859a9a03f6d5
  publish: true
---
WebDAV是一些网盘提供的协议，今天说一下如何在Ubuntu或CentOS将WebDAV挂载为本地磁盘。
## 参考资料 
1. [Linux将WebDAV为本地磁盘 - 夏日冰菓 (lincloud.pro)](https://blog.lincloud.pro/archives/36.html)
2. 


3. 安装所需程序：  
    **Ubuntu**：
    
    ```
    sudo apt-get install davfs2 -y
    ```
    
    **CentOS**：
    
    ```
    sudo yum install davfs2 -y
    ```
    
2. 创建挂载目录：
    
    ```
    sudo mkdir /mnt/WebDAV
    ```
    
3. 挂载WebDAV服务到本地目录：
    
    ```
    sudo mount -t davfs -o noexec https://example.com/webdav/ /mnt/WebDAV/
    ```
    
    之后会要求输入账户和密码登信息。挂载成功后，即可当正常磁盘一样访问WebDAV服务了。速度快慢取决于你自身和服务商的网速。
    
4. 解除挂载方法：
    
    ```
    sudo umount /mnt/WebDAV
    ```
    
5. 使用fstab挂载WebDAV：
    
    ```
    $ cat << EOF | sudo tee -a /etc/fstab# personal webdavhttps://example.com/webdav/ /mnt/WebDAV  davfs _netdev,noauto,user,uid=nobody,gid=nobody 0 0EOF
    ```
    
6. 保存账户密码：
    
    ```
    cat << EOF | sudo tee -a /etc/davfs2/secrets/mnt/dav account password EOF
    ```
    

摘录参考资料：[How to mount WebDAV share](https://shipengliang.com/go/vlybg5re6p4s)

### 实现开机自动挂载在WebDAV

普通挂载后，重启就会发现通过 WebDAV 挂载的磁盘没有了，也就意味着你每次重启 Linux 系统，都需要重新挂载，这时候需要更改几个设置来实现开机自动挂载。

**第一步、编辑davfs2.conf配置文件，将use_locks的1改为0**

```shell
vim /etc/davfs2/davfs2.conf 
```

Shell

_复制_

![image-20211012153936525](https://cdn.lincloud.pro/blog/images/202110121401.jpg "image-20211012153936525")

**第二步、修改secrets文件，添加账号信息**

```shell
vim /etc/davfs2/secrets
```

Shell

_复制_

在底部添加账号信息，如

```shell
https://pan.cloud.com/dav user password
```

Shell

_复制_

**第三步、添加开机挂载命令**

```shell
vim /etc/rc.local
```

Shell

_复制_

末尾添加挂载命令，和挂在U盘一样

```shell
mount -t davfs https://pan.cloud.com/dav /cloud
```

Shell

_复制_

重启即可自动挂载。

**最后一步、测试**

输入 `df -h` 查看是否成功

![屏幕截图 2021-10-12 155223](https://cdn.lincloud.pro/blog/images/202110121402.jpg "屏幕截图 2021-10-12 155223")

不错，1.3T空间，手到擒来。

**值得注意的是，如果开机没有自动挂载，有可能是rc.local文件没有权限，需要先执行`chmod +x /etc/rc.local`再重启系统。WebDAV服务商网络连接质量好的话，使用将非常顺滑，而且不占用本地磁盘空间。国外的VPS可以使用国外的知名的云盘运营商，他们的链路质量相对比较优秀，国内的大部分野鸡云也会提供该选项，但是有跑路的风险，无论如何，这羊毛是可以试试的**