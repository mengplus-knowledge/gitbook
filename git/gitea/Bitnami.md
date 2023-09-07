# Bitnami部署
## 参考资料
1. docker https://hub.docker.com/r/bitnami/gitea
2. https://github.com/bitnami/containers/tree/main/bitnami/gitea

## 部署方式
1.同样是docker 环境，需要根基自身环境先安装docker docker-compose
2. 在``参考资料3``可以拿到docker-compose.yml
3. 下载到本地后，在存放文件夹执行 ``docker-compose up -d``即可，

## 配置说明
1. 默认新建的管理员账户USER:`bn_user` PASSWD:`bitnami`,初次启动时可改
2. http端口`3000` ssh端口`2222`
3. 仓库本地`gitea_data` 数据库`postgresql_data`
4. 后续修改进入`gitea_data:/var/lib/docker/volumes/gitea_gitea_data/_data`本地路径修改`app.ini`



```yaml
# Copyright VMware, Inc.
# SPDX-License-Identifier: APACHE-2.0

version: '2'
volumes:
  postgresql_data:
    driver: local

  gitea_data:
    driver: local

services:
  postgresql:
    restart: always
    image: docker.io/bitnami/postgresql:15
    volumes:
      - 'postgresql_data:/bitnami/postgresql'
      - 'postgresql_data:/docker-entrypoint-initdb.d'
      - 'postgresql_data:/docker-entrypoint-preinitdb.d'
    environment:
      - POSTGRESQL_DATABASE=bitnami_gitea
      - POSTGRESQL_USERNAME=bn_gitea
      - POSTGRESQL_PASSWORD=bitnami1
      # ALLOW_EMPTY_PASSWORD is recommended only for development.
      - ALLOW_EMPTY_PASSWORD=yes
  gitea:
    restart: always
    container_name: gitea
    image: docker.io/bitnami/gitea:1
    hostname: "docker_gitea"
    volumes:
      - 'gitea_data:/bitnami/gitea'
    environment:
      - GITEA_DATABASE_HOST=postgresql
      - GITEA_DATABASE_NAME=bitnami_gitea
      - GITEA_DATABASE_USERNAME=bn_gitea
      - GITEA_DATABASE_PASSWORD=bitnami1
      - GITEA_ADMIN_PASSWORD=password123
    ports:
      - '3000:3000'
      - '2222:2222'


```
