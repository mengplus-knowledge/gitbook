# container


## ps查看在运行容器
```bash
$ docker ps #查看容器列表

CONTAINER ID   IMAGE                  COMMAND                   CREATED          STATUS          PORTS                                                                            NAMES
ca8b765e61d8   postgres:16.1-alpine   "docker-entrypoint.s…"   10 minutes ago   Up 10 minutes   127.0.0.1:5432->5432/tcp                                                         1Panel-postgresql-YlDc
75108bdd78c6   redis:7.2.2            "docker-entrypoint.s…"   3 weeks ago      Up 3 hours      127.0.0.1:6379->6379/tcp                                                         1Panel-redis-fBQo
d68336a2b033   mysql:8.2.0            "docker-entrypoint.s…"   3 weeks ago      Up 3 hours      127.0.0.1:3306->3306/tcp, 33060/tcp                                              1Panel-mysql-hUMf
6aabd9f04777   gitea/gitea:1.20.3     "/usr/bin/entrypoint…"   2 months ago     Up 3 hours      0.0.0.0:3000->3000/tcp, :::3000->3000/tcp, 0.0.0.0:222->22/tcp, :::222->22/tcp   gitea
5786dda5dc4d   postgres:14            "docker-entrypoint.s…"   2 months ago     Up 3 hours      5432/tcp                                                                         postgres
e45dd897e830   billryan/gitbook       "gitbook serve"           2 months ago     Up 12 minutes   0.0.0.0:4000->4000/tcp, :::4000->4000/tcp                                        gitbook
```
## exec 进入正在运行容器
```bash
docker exec  -it  gitbook   bash # 示例 打开gitbook
```
```bash
docker exec --help #查看说明

Usage:  docker exec [OPTIONS] CONTAINER COMMAND [ARG...]

Execute a command in a running container

Aliases:
  docker container exec, docker exec

Options:
  -d, --detach               Detached mode: run command in the background
      --detach-keys string   Override the key sequence for detaching a container
  -e, --env list             Set environment variables
      --env-file list        Read in a file of environment variables
  -i, --interactive          Keep STDIN open even if not attached
      --privileged           Give extended privileges to the command
  -t, --tty                  Allocate a pseudo-TTY
  -u, --user string          Username or UID (format: "<name|uid>[:<group|gid>]")
  -w, --workdir string       Working directory inside the container

```
