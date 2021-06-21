---
title: Docker_P2
excerpt: 创建镜像、标记镜像、删除镜像、镜像分层、优化镜像、创建可执行镜像、在线共享镜像
tags: [Docker]
categories: [容器]
date: 2021-06-20 16:30:00
---

# Docker_P2

## 如何创建Docker镜像

使用DockerFile文件创建镜像，Dockerfile是创建镜像的指令集。如下是一个简单的Nginx镜像的Dockerfile文件。

```tex
FROM ubuntu:latest

EXPOSE 80

RUN apt-get update && \
    apt-get install nginx -y && \
    apt-get clean && rm -rf /var/lib/apt/lists/*

CMD ["nginx", "-g", "daemon off;"]
```

* 每一个Dockerfile都以`FROM`指令开头。用以指定基础镜像。示例基础镜像为`ubuntu:latest`，可以在自定义镜像中使用Ubuntu的所有功能。

* `EXPOSE`表示自定义镜像需要发布的端口。有该指令则在运行容器的时候不需要使用`--publish`参数发布端口。

* `RUN`在自定义容器内部的Shell中执行命令。`apt-get update && apt-get install nginx -y`检查更新软件包版本并安装Nginx。

  `apt-get clean && rm -rf /var/lib/apt/lists/*`清除程序包缓存。`RUN`命令是以`shell`形式编写，此外还有[exec](https://docs.docker.com/engine/reference/builder/#run)。

* `CMD`为自定义镜像设定默认命令。该指令以 `exec` 形式编写，`CMD` 指令也可以以 `shell` 形式编写[官方参考](https://docs.docker.com/engine/reference/builder/#cmd)。

镜像相关命令格式：

```shell
docker image <command> <options>
```

可以使用如下命令从有效的DockerFile中构建镜像

```shell
➜  test  ls
Dockerfile  delete_log.log  t1.txt  t3.jpg  t4.txt
➜  test  sudo docker image build .
Sending build context to Docker daemon  4.608kB
Step 1/4 : FROM ubuntu:latest
 ---> 9873176a8ff5
Step 2/4 : EXPOSE 80
 ---> Running in 3d33468bc729
Removing intermediate container 3d33468bc729
 ---> 454bf2c6222e
Step 3/4 : RUN apt-get update &&     apt-get install nginx -y &&     apt-get clean && rm -rf /var/lib/apt/lists/*
 ---> Running in dd819d589cda
Get:1 http://archive.ubuntu.com/ubuntu focal InRelease [265 kB]
Get:2 http://security.ubuntu.com/ubuntu focal-security InRelease [114 kB]
Get:3 http://security.ubuntu.com/ubuntu focal-security/universe amd64 Packages [730 kB]
...
Setting up nginx (1.18.0-0ubuntu1.2) ...
Processing triggers for libc-bin (2.31-0ubuntu9.2) ...
Removing intermediate container dd819d589cda
 ---> f91fd5f91054
Step 4/4 : CMD ["nginx", "-g", "daemon off;"]
 ---> Running in eca213ce8ad3
Removing intermediate container eca213ce8ad3
 ---> 970ed77299da
Successfully built 970ed77299da
```

* `docker image build`用于构建镜像。守护程序会在上下文中找名为Dockerfile的文件。
* `.`设置此构建的上下文。就是构建的时候从什么路径找Dockerfile文件。

现在要运行构建的容器，需要指定容器ID`970ed77299da`

```shell
➜  test  sudo docker container run --rm --detach --name my-nginx --publish 8888:80 970ed77299da
f9800dc2c291203187db0a11ee373df38ef44ebfdb0be869bb7fb368d5e85c94
```

![image-20210621212638568](/img/docker/image-20210621212638568.png)

## 如何标记Docker镜像

在构建镜像的时候可以标记镜像，从而在使用的时候不必依赖随机生成的ID。命令如下

```shell
--tag <image repository>:<image tag>
```

repository 通常指镜像名称，而 tag 指特定的构建或版本。

以官方 [mysql](https://hub.docker.com/_/mysql) 镜像为例。如果想使用特定版本的MySQL（例如5.7）运行容器，则可以执行 `docker container run mysql:5.7`，其中 `mysql` 是镜像 repository，`5.7` 是 tag。

重新构建Nginx镜像

```shell
➜  test  sudo docker image build --tag nginx:packaged .
Sending build context to Docker daemon  4.608kB
Step 1/4 : FROM ubuntu:latest
 ---> 9873176a8ff5
Step 2/4 : EXPOSE 80
 ---> Using cache
 ---> 454bf2c6222e
Step 3/4 : RUN apt-get update &&     apt-get install nginx -y &&     apt-get clean && rm -rf /var/lib/apt/lists/*
 ---> Using cache
 ---> f91fd5f91054
Step 4/4 : CMD ["nginx", "-g", "daemon off;"]
 ---> Using cache
 ---> 970ed77299da
Successfully built 970ed77299da
Successfully tagged nginx:packaged
```

重新标记

```shell
docker image tag <image repository>:<image tag> <new image repository>:<new image tag>
```

```shell
➜  test  sudo docker image ls
REPOSITORY            TAG        IMAGE ID       CREATED          SIZE
nginx                 packaged   970ed77299da   20 minutes ago   132MB
ubuntu                latest     9873176a8ff5   3 days ago       72.7MB
busybox               latest     69593048aa3a   13 days ago      1.24MB
node                  latest     d1b3088a17b1   2 weeks ago      908MB
nginx                 latest     d1a364dc548d   3 weeks ago      133MB
hello-world           latest     d1165f221234   3 months ago     13.3kB
fhsinchy/hello-dock   latest     f540930e8157   4 months ago     21.9MB
fhsinchy/rmbyext      latest     90eafb66c390   5 months ago     50.9MB
➜  test  sudo docker image tag nginx:packaged nginx:me
➜  test  sudo docker image ls
REPOSITORY            TAG        IMAGE ID       CREATED          SIZE
nginx                 me         970ed77299da   21 minutes ago   132MB
nginx                 packaged   970ed77299da   21 minutes ago   132MB
ubuntu                latest     9873176a8ff5   3 days ago       72.7MB
busybox               latest     69593048aa3a   13 days ago      1.24MB
node                  latest     d1b3088a17b1   2 weeks ago      908MB
nginx                 latest     d1a364dc548d   3 weeks ago      133MB
hello-world           latest     d1165f221234   3 months ago     13.3kB
fhsinchy/hello-dock   latest     f540930e8157   4 months ago     21.9MB
fhsinchy/rmbyext      latest     90eafb66c390   5 months ago     50.9MB
```

根据镜像ID标记（构建时候未标记，可以抢救一下）

```shell
docker image tag <image id> <image repository>:<image tag>
```

## 如何列表展示、删除镜像

列表展示

```shell
image ls 或 images
```

```shell
➜  test  sudo docker image ls
REPOSITORY            TAG        IMAGE ID       CREATED          SIZE
nginx                 me         970ed77299da   22 minutes ago   132MB
nginx                 packaged   970ed77299da   22 minutes ago   132MB
ubuntu                latest     9873176a8ff5   3 days ago       72.7MB
busybox               latest     69593048aa3a   13 days ago      1.24MB
node                  latest     d1b3088a17b1   2 weeks ago      908MB
nginx                 latest     d1a364dc548d   3 weeks ago      133MB
hello-world           latest     d1165f221234   3 months ago     13.3kB
fhsinchy/hello-dock   latest     f540930e8157   4 months ago     21.9MB
fhsinchy/rmbyext      latest     90eafb66c390   5 months ago     50.9MB
➜  test  sudo docker images
REPOSITORY            TAG        IMAGE ID       CREATED          SIZE
nginx                 me         970ed77299da   22 minutes ago   132MB
nginx                 packaged   970ed77299da   22 minutes ago   132MB
ubuntu                latest     9873176a8ff5   3 days ago       72.7MB
busybox               latest     69593048aa3a   13 days ago      1.24MB
node                  latest     d1b3088a17b1   2 weeks ago      908MB
nginx                 latest     d1a364dc548d   3 weeks ago      133MB
hello-world           latest     d1165f221234   3 months ago     13.3kB
fhsinchy/hello-dock   latest     f540930e8157   4 months ago     21.9MB
fhsinchy/rmbyext      latest     90eafb66c390   5 months ago     50.9MB
```

删除镜像

```shell
docker image rm <image identifier>
```

标识符可以是镜像 ID 或镜像仓库。 如果使用仓库，则还必须指定标记。

```shell
➜  test  sudo docker images
REPOSITORY            TAG        IMAGE ID       CREATED          SIZE
nginx                 me         970ed77299da   22 minutes ago   132MB
nginx                 packaged   970ed77299da   22 minutes ago   132MB
ubuntu                latest     9873176a8ff5   3 days ago       72.7MB
busybox               latest     69593048aa3a   13 days ago      1.24MB
node                  latest     d1b3088a17b1   2 weeks ago      908MB
nginx                 latest     d1a364dc548d   3 weeks ago      133MB
hello-world           latest     d1165f221234   3 months ago     13.3kB
fhsinchy/hello-dock   latest     f540930e8157   4 months ago     21.9MB
fhsinchy/rmbyext      latest     90eafb66c390   5 months ago     50.9MB
➜  test  sudo docker image rm hello-world:latest
Untagged: hello-world:latest
Untagged: hello-world@sha256:9f6ad537c5132bcce57f7a0a20e317228d382c3cd61edae14650eec68b2b345c
Deleted: sha256:d1165f2212346b2bab48cb01c1e39ee8ad1be46b87873d9ca7a4e434980a7726
Deleted: sha256:f22b99068db93900abe17f7f5e09ec775c2826ecfe9db961fea68293744144bd
➜  test  sudo docker images
REPOSITORY            TAG        IMAGE ID       CREATED          SIZE
nginx                 me         970ed77299da   25 minutes ago   132MB
nginx                 packaged   970ed77299da   25 minutes ago   132MB
ubuntu                latest     9873176a8ff5   3 days ago       72.7MB
busybox               latest     69593048aa3a   13 days ago      1.24MB
node                  latest     d1b3088a17b1   2 weeks ago      908MB
nginx                 latest     d1a364dc548d   3 weeks ago      133MB
fhsinchy/hello-dock   latest     f540930e8157   4 months ago     21.9MB
fhsinchy/rmbyext      latest     90eafb66c390   5 months ago     50.9MB
```

清除所有未标记的挂起的镜像`image prune`

```shell
image prune
```

```shell
➜  test  sudo docker image prune --force
Deleted Images:
deleted: sha256:a3b3aa72358a587e2665fd4aedf1a9d8cd0c9befb7a9243574252c7c33e46fbd
deleted: sha256:346ed51eb155b2a4b5ac11fed84d8dfb7ab9f4b6e55216b4719bf6371383f7a6
deleted: sha256:fe281fc67be4751eaccb627f423960b9b4db3f6979a8953df7bedd273b14d8a0
deleted: sha256:4de16b13052380b1a47091cf5ce80200457c26af439320cff1d76697197eebb9

Total reclaimed space: 59.2MB
```

* `--force`或`-f`跳过确认问题。
* 使用`--all`或`-a`则删除本地仓库的所有缓存镜像。





