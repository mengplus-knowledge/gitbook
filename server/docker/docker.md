# docker

## images
1. pull下载镜像

``` bash
#1. 直接下载安装,默认latest标签
docker pull ubuntu
#2. 搜索镜像并安装
docker  search ubuntu
#3. 存在docker-compose.yml 直接自动安装
docker-compose up

```
2. search搜索镜像


## 常用命令

```bash
$sudo docker logs -f -t --tail 行数 容器名[containerID]
-f  按日志输出
-t  显示时间戳
```
