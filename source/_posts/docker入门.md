---
title: docker入门
date: 2022-11-26 00:24:22
author: 三岁浪迹天涯
categories: docker
tags:
  - docker
  - 基础
---

## docker

不支持windows10家庭版

CentOS 安装配置



yum install -y yum-utils device-mapper-persistent-data lvm2		安装依赖

 yum-config-manager --add-repo http://mirrors.aliyun.com/docker-ce/linux/centos/docker-ce.repo	添加源

yum -y install docker-ce 	安装docker	



或者curl -fsSL https://get.docker.com | bash -s docker --mirror Aliyun



此时直接使用docker ps查看进程是无效的，需要先systemctl start docker启动

设置开机自启动：	systemctl enable docker

docker pull nginx或者docker pull nginx:1.17.10

docker-compose安装

```
# curl -L https://github.com/docker/compose/releases/download/1.8.1/docker-compose-`uname -s`-`uname -m` > /usr/local/bin/docker-compose
# chmod +x /usr/local/bin/docker-compose
```

```shell
docker version	docker版本信息
docker info		docker系统信息
docker images	查看所有镜像
docker search busybox 	在网上搜索镜像
docker pull busybOx:latest	拉取下载
docker save busybox > busybox.tar	导出
docker load < busybox.tar	导入
docker rmi busybox:latest	删除镜像
docker tag busybox:latest busybox:test 	更改镜像名
docker history busybox 	查看镜像创建历史

docker run -d --name=busybox busybox:latest ping 114.114.114.114	运行容器
docker ps, docker ps -a		查看运行的容器
docker top busybox 		查看容器中运行的进程
docker stats busybox 	查看资源占用
docker start/restart/stop/kill busybox 容器
docker pause/unpause busybox 	暂停容器
docker rm -f busybox 	强制删除容器

docker exec -it busybox ls	执行命令
docker exec -it elasticsearch /bin/bash	在该容器下执行bash命令
docker cp busyboxL/etc/hosts hosts		复制文件
docker logs -f busybox		查看容器日志
docker inspect busybox		查看容器/镜像的元信息
docker inspect -f'{{.Id}}' busybox		格式化输出

docker build	基于dockerfile在构建镜像
docker commit	将容器转化为镜像
```

Nginx容器

```
docker run -d --name nginx -p 8088:80 nginx1.17.9			运行
docker run -d --name nginx1 -p 8089:80 -v ${PWD}/html:/usr/share/nginx/html/nginx:1.17.9	挂载目录
docker stop 停止
```



此外，还可以利用 `ADD` 命令复制本地文件到镜像；用 `EXPOSE` 命令来向外部开放端口；用 `CMD` 命令来描述容器启动后运行的程序等。例如



当利用 `docker run` 来创建容器时，Docker 在后台运行的标准操作包括：

- 检查本地是否存在指定的镜像，不存在就从公有仓库下载
- 利用镜像创建并启动一个容器
- 分配一个文件系统，并在只读的镜像层外面挂载一层可读写层
- 从宿主主机配置的网桥接口中桥接一个虚拟接口到容器中去
- 从地址池配置一个 ip 地址给容器
- 执行用户指定的应用程序
- 执行完毕后容器被终止

## 守护态运行：

更多的时候，需要让 Docker 容器在后台以守护态（Daemonized）形式运行。此时，可以通过添加 `-d` 参数来实现

## 容器重启

处于终止状态的容器，可以通过 `docker start` 命令来重新启动。

此外，`docker restart` 命令会将一个运行态的容器终止，然后再重新启动它。

## Dockerfile中命令

### RUN

格式为 `RUN <command>` 或 `RUN ["executable", "param1", "param2"]`。

前者将在 shell 终端中运行命令，即 `/bin/sh -c`；后者则使用 `exec` 执行。指定使用其它终端可以通过第二种方式实现，例如 `RUN ["/bin/bash", "-c", "echo hello"]`。

每条 `RUN` 指令将在当前镜像基础上执行指定命令，并提交为新的镜像。当命令较长时可以使用 `\` 来换行

### CMD

支持三种格式

- `CMD ["executable","param1","param2"]` 使用 `exec` 执行，推荐方式；
- `CMD command param1 param2` 在 `/bin/sh` 中执行，提供给需要交互的应用；
- `CMD ["param1","param2"]` 提供给 `ENTRYPOINT` 的默认参数；

指定启动容器时执行的命令，每个 Dockerfile 只能有一条 `CMD` 命令。如果指定了多条命令，只有最后一条会被执行。

如果用户启动容器时候指定了运行的命令，则会覆盖掉 `CMD` 指定的命令。

### ENV

格式为 `ENV <key> <value>`。 指定一个环境变量，会被后续 `RUN` 指令使用，并在容器运行时保持。

例如

```
ENV PG_MAJOR 9.3
ENV PG_VERSION 9.3.4
RUN curl -SL http://example.com/postgres-$PG_VERSION.tar.xz | tar -xJC /usr/src/postgress && …
ENV PATH /usr/local/postgres-$PG_MAJOR/bin:$PATH
```

### EXPOSE

格式为 `EXPOSE <port> [<port>...]`。

告诉 Docker 服务端容器暴露的端口号，供互联系统使用。在启动容器时需要通过 -P，Docker 主机会自动分配一个端口转发到指定的端口。

### ADD

格式为 `ADD <src> <dest>`。

该命令将复制指定的 `<src>` 到容器中的 `<dest>`。 其中 `<src>` 可以是Dockerfile所在目录的一个相对路径；也可以是一个 URL；还可以是一个 tar 文件（自动解压为目录）。

### COPY

格式为 `COPY <src> <dest>`。

复制本地主机的 `<src>`（为 Dockerfile 所在目录的相对路径）到容器中的 `<dest>`。

当使用本地目录为源目录时，推荐使用 `COPY`。

### WORKDIR

格式为 `WORKDIR /path/to/workdir`。

为后续的 `RUN`、`CMD`、`ENTRYPOINT` 指令配置工作目录。

可以使用多个 `WORKDIR` 指令，后续命令如果参数是相对路径，则会基于之前命令指定的路径
