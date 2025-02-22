# ddns动态域名

docker-compose.yml
```yml
version: '3.6'
services:
  ddns:
    container_name: ddns
    image: 'newfuture/ddns'
    restart: always
    tty: true     # 给容器设置一个伪终端防止进程结束容器退出
    #command:
     #- bash
    hostname: 'ddns'
    #environment: # 添加环境变量。
     # GITLAB_OMNIBUS_CONFIG: |
     #   external_url 'https://gitlab.example.com'
     #   # Add any other gitlab.rb configuration here, each on its own line
    volumes:
      - type: bind
        source: './config.json'
        target: '/config.json'
    network_mode: host
    shm_size: '512m'
  # cap_add:
  #  - ALL # 开启全部权限
  #devices:  #挂设备
  # - "/dev/ttyUSB0:/dev/ttyUSB0"



```

当前路径下的配置文件,这里配置的是阿里云
```json
{
    "$schema": "https://ddns.newfuture.cc/schema/v2.8.json",
    "id": "XXXXXX",
    "token": "XXXXX",
    "dns": "alidns",
    "ipv4": [],
    "ipv6": [
        "mengplus.top",
        "gitea.mengplus.top",
        "gitbook.mengplus.top"
    ],
    "index4": 0,
    "index6": "public",
    "ttl": 600,
    "proxy": "127.0.0.1:1080;DIRECT",
    "debug": true
}

```
