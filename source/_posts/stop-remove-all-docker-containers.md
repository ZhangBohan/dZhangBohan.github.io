title: "Stop / remove所有Docker容器笔记"
date: 2015-11-23 17:55:48
tags:
- Linux
- Docker
- 笔记
---

一行代码停止或者删除全部的[Docker](https://www.docker.com/)容器

```
docker stop $(docker ps -a -q)
docker rm $(docker ps -a -q)
```

--------------------------

## 个人笔记

以`docker stop`为例，输入`docker stop`，显示

```
docker: "stop" requires a minimum of 1 argument.
See 'docker stop --help'.

Usage:	docker stop [OPTIONS] CONTAINER [CONTAINER...]

Stop a running container.
Sending SIGTERM and then SIGKILL after a grace period
```

帮助说明`docker stop` 可以接受一个以空格分隔的容器数组，只要把全部的容器名称以空格连接就可以了。

```
> docker ps -a -q
Usage:	docker ps [OPTIONS]

List containers

  -a, --all=false       Show all containers (default shows just running)
  --before=             Show only container created before Id or Name
  -f, --filter=[]       Filter output based on conditions provided
  --format=             Pretty-print containers using a Go template
  --help=false          Print usage
  -l, --latest=false    Show the latest created container, include non-running
  -n=-1                 Show n last created containers, include non-running
  --no-trunc=false      Don't truncate output
  -q, --quiet=false     Only display numeric IDs
  -s, --size=false      Display total file sizes
  --since=              Show created since Id or Name, include non-running
```

- `-a`表示全部的容器
- `-q`表示只显示容器ID

这样就可以取得全部的以换行符分隔的容器ID列表，再通过[$()](http://pubs.opengroup.org/onlinepubs/009695399/utilities/xcu_chap02.html#tag_02_06_03)操作将换行符替换，就可以了。
