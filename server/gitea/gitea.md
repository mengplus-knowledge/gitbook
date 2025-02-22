---
title: gitea个人仓库部署
categories:
  - 网站运维
tags:
  - git
  - 仓库
  - docker
halo:
  site: https://mengplus.top
  name: f43a5d7b-57db-4f85-a679-fc7d36de9d31
  publish: true
time: 2024-03-04
---
# gitea 官方指导部署方式
## 参考资料
1. 帮助文档 https://docs.gitea.cn/installation/install-with-docker
2. 容器地址 http://hub.docker.com
3. 示例仓库[gitea - Gitea: Git with a cup of tea](https://gitea.com/gitea)
4. 源码地址[go-gitea/gitea: Git with a cup of tea! Painless self-hosted all-in-one software development service, including Git hosting, code review, team collaboration, package registry and CI/CD (github.com)](https://github.com/go-gitea/gitea)
## 配置说明
- git服务器选用gitea,数据存储在gitea_data(主机路径local:/var/lib/docker/volumes/gitea_gitea_data/_data)
- 数据库选用postgresql,数据存储在gitea_data(主机路径local)
- http访问端口3000
- ssh端口222
## 使用方式
```bash
#安装docker-compose
sudo apt install docker-compose
#编写脚本
#将下文的配置信息 写入后保存
vim docker-compose.yml

#后台运行脚本
docker-compose up -d
```

docker-compose.yml配置信息
```yaml
version: "3"

networks:
  gitea:
    external: false
volumes:
  postgresql_data:
    driver: local
  gitea_data:
    driver: local
services:
  postgresql:
    image: postgres:14
    restart: always
    environment:
      - POSTGRES_USER=gitea
      - POSTGRES_PASSWORD=gitea
      - POSTGRES_DB=gitea
    networks:
      - gitea
    volumes:
      - postgresql_data:/var/lib/postgresql/data

  server:
    image: gitea/gitea:1.20.7
    container_name: gitea
    environment:
      - USER_UID=1000
      - USER_GID=1000
      - GITEA__database__DB_TYPE=postgres
      - GITEA__database__HOST=postgresql:5432
      - GITEA__database__NAME=gitea
      - GITEA__database__USER=gitea
      - GITEA__database__PASSWD=gitea
    restart: always
    networks:
      - gitea
    volumes:
      - gitea_data:/data
      - /etc/timezone:/etc/timezone:ro
      - /etc/localtime:/etc/localtime:ro
    ports:
      - "3000:3000"
      - "222:22"
    depends_on:
      - postgresql

```
