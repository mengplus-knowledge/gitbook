---
title: 限制用户sudo权限
slug: xian-zhi-yong-hu-quan-xian
cover: ""
categories:
  - linux
tags: []
halo:
  site: https://mengplus.top
  name: 9586a252-e5f6-413e-94b7-8722a5ac55b7
  publish: true
---

在我的应用场景中有个`cat`用户需要作为产品的默认账户使用，但是同时不想让操作者对设备进行非法操作，那么可以进行如下约束配置

> [!NOTE]
>
> 关权限容易 ，也要注意把root进行激活



```bash
#cat用户权限限制

echo "正在限制cat用户权限..."

ROOT_PASSWORD="cat_passwd"                                # 或者动态生成密码：ROOT_PASSWORD=$(openssl rand -base64 12)

echo -e "$ROOT_PASSWORD\n$ROOT_PASSWORD" | passwd root # 启用 root 并设置密码
deluser cat sudo
deluser cat adm
touch /usr/local/bin/restricted_sudo

echo "#!/bin/bash
ALLOWED_PATH=\"/mnt/b/script/*.sh\"
if [[ \"\$1\" == \$ALLOWED_PATH ]]; then
    sudo \"\$1\"
else
    echo \"Permission denied.\"
    exit 1
fi" >>/usr/local/bin/restricted_sudo

chown root:root /usr/local/bin/restricted_sudo

chmod 755 /usr/local/bin/restricted_sudo

  

touch /etc/sudoers.d/cat

chmod 711 /etc/sudoers.d/cat

echo "cat ALL=(root) NOPASSWD: /usr/local/bin/restricted_sudo" >>/etc/sudoers.d/cat

echo "cat ALL=(root) NOPASSWD: /mnt/b/script/*" >>/etc/sudoers.d/cat

echo "cat ALL=(root) NOPASSWD: /usr/bin/apt" >>/etc/sudoers.d/cat

echo "cat ALL=(root) NOPASSWD: /usr/bin/apt-get" >>/etc/sudoers.d/cat

echo 'alias sudo="/usr/local/bin/restricted_sudo"' >>/home/cat/.bashrc

source /home/cat/.bashrc
```
此处使用了两种限制方式
1. **restricted_sudo** 替换sudo限制只允许`restricted_sudo`文件中约定的程序使用
2. **cat ALL=(root) NOPASSWD: /usr/bin/apt** 限制管理员权限只允许进行软件安装
另外使用时注意把核心配置文件做好权限控制别给用户编辑权限，甚至是阅读权限使用`chmod`

