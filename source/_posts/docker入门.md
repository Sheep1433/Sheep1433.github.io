---
title: docker入门
date: 2022-11-26 00:24:22
author: 三岁浪迹天涯
categories: docker
tags:
  - docker
  - 基础
---

## docker概念

![image-20230117222926334](D:\Sheep1433.github.io\source\picture\image-20230117222926334.png)

![image-20230117223015800](D:\Sheep1433.github.io\source\picture\image-20230117223015800.png)

核心概念：镜像、容器、仓库

## docker安装

不支持windows10家庭版

CentOS 安装配置

yum install -y yum-utils device-mapper-persistent-data lvm2		安装依赖

 yum-config-manager --add-repo http://mirrors.aliyun.com/docker-ce/linux/centos/docker-ce.repo	添加源

yum -y install docker-ce 	安装docker	



或者curl -fsSL https://get.docker.com | bash -s docker --mirror Aliyun



此时直接使用docker ps查看进程是无效的，需要先systemctl start docker启动

设置开机自启动：	systemctl enable docker

docker-compose安装

```
# curl -L https://github.com/docker/compose/releases/download/1.8.1/docker-compose-`uname -s`-`uname -m` > /usr/local/bin/docker-compose
# chmod +x /usr/local/bin/docker-compose
```

## docker常用命令

**一张图总结 Docker 的命令**

![img](D:\Sheep1433.github.io\source\picture\cmd_logic.5970ea4d.png)

```
exec	build		commit	exec	logs	ps	 	pull
push	restart		rm		rmi		run		search	start
tag		version		info	save	history	……
```



```shell
docker version	docker版本信息
docker info		docker系统信息
# 镜像相关
docker images	查看所有镜像
docker search busybox 	在网上搜索镜像
docker pull busybOx:latest	拉取下载
docker save busybox > busybox.tar	导出
docker load < busybox.tar	导入
docker rmi busybox:latest	删除镜像
docker tag busybox:latest busybox:test 	更改镜像名
docker history busybox 	查看镜像创建历史
# 容器相关 
# docker run中-p：端口映射，--name：命名，-v：挂载目录
docker run -d -it busybox /bin/bash	# 这种运行方式，容器不会立刻被退出 -d以守护态运行
docker run -d --name=busybox busybox:latest ping 114.114.114.114	运行容器

docker ps, docker ps -a		查看运行的容器
docker top busybox 		查看容器中运行的进程
docker stats busybox 	查看资源占用
# 处于终止状态的容器，可以通过 `docker start` 命令来重新启动。
# 此外，`docker restart` 命令会将一个运行态的容器终止，然后再重新启动它。
docker start/restart/stop/kill busybox 容器
docker pause/unpause busybox 	暂停容器
docker rm -f busybox 	强制删除容器
docker exec -it busybox ls	执行命令
docker exec -it elasticsearch /bin/bash	在该容器下执行bash命令
docker cp busyboxL/etc/hosts hosts		复制文件
docker logs -f busybox		查看容器日志
docker inspect busybox		查看容器/镜像的元信息
docker inspect -f'{{.Id}}' busybox		格式化输出

docker build -t {tag} .	基于dockerfile在构建镜像
docker commit -m "提交信息" {容器id} {命名}	将容器转化为镜像
```

当利用 `docker run` 来创建容器时，Docker 在后台运行的标准操作包括：

- 检查本地是否存在指定的镜像，不存在就从公有仓库下载
- 利用镜像创建并启动一个容器
- 分配一个文件系统，并在只读的镜像层外面挂载一层可读写层
- 从宿主主机配置的网桥接口中桥接一个虚拟接口到容器中去
- 从地址池配置一个 ip 地址给容器
- 执行用户指定的应用程序
- 执行完毕后容器被终止

Nginx容器

```bash
docker run -d --name nginx -p 8088:80 nginx1.17.9			# 运行
docker run -d --name nginx1 -p 8089:80 -v ${PWD}/html:/usr/share/nginx/html/nginx:1.17.9	挂载目录
docker stop 停止
```

Jenkins容器

```bash
docker run -p 8888:8080 --name myjenkins --user root myjenkins -v /data/jenkins_home:/var/jenkins_home jenkins/jenkins:lts
docker exec -it --user root <container id> /bin/bash
```



## Dockerfile

**示例**

```dockerfile
# Dockerfile
FROM        ubuntu:14.04
RUN         apt-get update
RUN         apt-get -y install redis-server
EXPOSE      6379
ENTRYPOINT  ["/usr/bin/redis-server"]
```

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

## docker-compose常用命令

Dockerfile 可以让用户管理一个单独的应用容器；而 Compose 则允许用户在一个模板（YAML 格式）中定义一组相关联的应用容器（被称为一个 `project`，即项目），例如一个 Web 服务容器再加上后端的数据库服务容器等。

常用命令

```bash
build		# 构建或重新构建服务。服务一旦构建后，将会带上一个标记名，例如 web_db。可以随时在项目目录下运行 `docker-compose build` 来重新构建服务。
help		
kill		# 通过发送 `SIGKILL` 信号来强制停止服务容器。支持通过参数来指定发送的信号，例如
logs
port
ps
pull
rm
run
start
stop
up			#	构建，（重新）创建，启动，链接一个服务相关的容器。链接的服务都将会启动，除非他们已经运行。默认情况， `docker-compose up` 将会整合所有容器的输出，并且退出时，所有容器将会停止。如果使用 `docker-compose up -d` ，将会在后台启动并运行所有的容器。默认情况，如果该服务的容器已经存在，`docker-compose up` 将会停止并尝试重新创建他们（保持使用`volumes-from` 挂载的卷），以保证 `docker-compose.yml` 的修改生效。如果你不想容器被停止重新创建，可以使用 `docker-compose up --no-recreate`。如果需要的话，这样将会启动已经停止的容器
```

## docker-compose.yml常用字段

默认的模板文件是 `docker-compose.yml`，其中定义的每个服务都必须通过 `image` 指令指定镜像或 `build` 指令（需要 Dockerfile）来自动构建。

其它大部分指令都跟 `docker run` 中的类似。

如果使用 `build` 指令，在 `Dockerfile` 中设置的选项(例如：`CMD`, `EXPOSE`, `VOLUME`, `ENV` 等) 将会自动被获取，无需在 `docker-compose.yml` 中再次设置。

### image

指定为镜像名称或镜像 ID。如果镜像在本地不存在，`Compose` 将会尝试拉去这个镜像。

例如：

```sh
image: ubuntu
image: orchardup/postgresql
image: a4bc65fd
```

### build

指定 `Dockerfile` 所在文件夹的路径。 `Compose` 将会利用它自动构建这个镜像，然后使用这个镜像。

```
build: /path/to/build/dir
```

### command

覆盖容器启动后默认执行的命令。

```sh
command: bundle exec thin -p 3000
```

### link

链接到其它服务中的容器。使用服务名称（同时作为别名）或服务名称：服务别名 `（SERVICE:ALIAS）` 格式都可以。

```sh
links:
 - db
 - db:database
 - redis
```

使用的别名将会自动在服务容器中的 `/etc/hosts` 里创建。例如：

```sh
172.17.2.186  db
172.17.2.186  database
172.17.2.187  redis
```

相应的环境变量也将被创建。

### external_links

链接到 docker-compose.yml 外部的容器，甚至 并非 `Compose` 管理的容器。参数格式跟 `links` 类似。

```
external_links:
 - redis_1
 - project_db_1:mysql
 - project_db_1:postgresql
```

### ports

暴露端口信息。

使用宿主：容器 `（HOST:CONTAINER）`格式或者仅仅指定容器的端口（宿主将会随机选择端口）都可以。

```
ports:
 - "3000"
 - "8000:8000"
 - "49100:22"
 - "127.0.0.1:8001:8001"
```

*注：当使用 `HOST:CONTAINER` 格式来映射端口时，如果你使用的容器端口小于 60 你可能会得到错误得结果，因为 `YAML` 将会解析 `xx:yy` 这种数字格式为 60 进制。所以建议采用字符串格式。*

### expose

暴露端口，但不映射到宿主机，只被连接的服务访问。

仅可以指定内部端口为参数

```sh
expose:
 - "3000"
 - "8000"
```

### volumes

卷挂载路径设置。可以设置宿主机路径 （`HOST:CONTAINER`） 或加上访问模式 （`HOST:CONTAINER:ro`）。

```sh
volumes:
 - /var/lib/mysql
 - cache/:/tmp/cache
 - ~/configs:/etc/configs/:ro
```

### volumes_from

从另一个服务或容器挂载它的所有卷。

```sh
volumes_from:
 - service_name
 - container_name
```

### environment

设置环境变量。你可以使用数组或字典两种格式。

只给定名称的变量会自动获取它在 Compose 主机上的值，可以用来防止泄露不必要的数据。

```
environment:
  RACK_ENV: development
  SESSION_SECRET:

environment:
  - RACK_ENV=development
  - SESSION_SECRET
```

### env_file

从文件中获取环境变量，可以为单独的文件路径或列表。

如果通过 `docker-compose -f FILE` 指定了模板文件，则 `env_file` 中路径会基于模板文件路径。

如果有变量名称与 `environment` 指令冲突，则以后者为准。

```sh
env_file: .env

env_file:
  - ./common.env
  - ./apps/web.env
  - /opt/secrets.env
```

环境变量文件中每一行必须符合格式，支持 `#` 开头的注释行。

```sh
# common.env: Set Rails/Rack environment
RACK_ENV=development
```

### extends

基于已有的服务进行扩展。例如我们已经有了一个 webapp 服务，模板文件为 `common.yml`。

```sh
# common.yml
webapp:
  build: ./webapp
  environment:
    - DEBUG=false
    - SEND_EMAILS=false
```

编写一个新的 `development.yml` 文件，使用 `common.yml` 中的 webapp 服务进行扩展。

```sh
# development.yml
web:
  extends:
    file: common.yml
    service: webapp
  ports:
    - "8000:8000"
  links:
    - db
  environment:
    - DEBUG=true
db:
  image: postgres
```

后者会自动继承 common.yml 中的 webapp 服务及相关环节变量。



