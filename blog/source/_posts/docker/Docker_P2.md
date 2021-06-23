---
title: Docker_P2
excerpt: 创建镜像、标记镜像、删除镜像、镜像分层、优化镜像、创建可执行镜像、在线共享镜像
index_img: /img/docker/docker.png
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

* `EXPOSE`表示自定义镜像需要发布的端口。仍需要使用`--publish`参数发布端口与宿主机端口映射。

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

## 理解Docker镜像的分层

使用`image history`可以可视化镜像的多个层，以前面的Dockerfile构建一个名为`custom-nginx:packaged`的镜像为例。

```shell
➜  test  sudo docker image history custom-nginx:packaged
IMAGE          CREATED          CREATED BY                                      SIZE      COMMENT
b11ac2f129e6   59 seconds ago   /bin/sh -c #(nop)  CMD ["nginx" "-g" "daemon…   0B
7c8f3aa0cbfe   59 seconds ago   /bin/sh -c apt-get update &&     apt-get ins…   59.2MB
ebff7392ea65   2 minutes ago    /bin/sh -c #(nop)  EXPOSE 80                    0B
9873176a8ff5   5 days ago       /bin/sh -c #(nop)  CMD ["bash"]                 0B
<missing>      5 days ago       /bin/sh -c #(nop) ADD file:920cf788d1ba88f76…   72.7MB
```

`custom-nginx:packaged`镜像一共有5层，最上面的一层是最新的层，最顶层通常是用于运行容器的那一层。

忽略`<missing>`层，探讨其他层作用：

* `9873176a8ff5`层由 `/bin/sh -c #(nop)  CMD ["bash"] `创建，表示Ubuntu中的默认shell已经成功加载。
* `ebff7392ea65`层由`/bin/sh -c #(nop)  EXPOSE 80 `创建，表示暴露了端口80。
* `7c8f3aa0cbfe`层由`apt-get update && apt-get install nginx -y && apt-get clean && rm -rf /var/lib/apt/lists/*`创建，表示安装了必要的程序，大小为59.2MB。
* `b11ac2f129e6`层由`CMD ["nginx", "-g", "daemon off;"]`创建表示为改镜像创建了默认命令。

镜像由许多只读层组成，每个层都`记录`了由某些指令触发的一组新的`状态更改`。

当使用镜像启动容器时，会在其他层之上获得一个新的可写层。

>分层现象，是通过一个称为 union file system 的技术概念而得以实现的。
>
>它允许透明地覆盖独立文件系统（称为分支）的文件和目录，从而形成单个一致的文件系统。`合并分支内具有相同路径的目录的内容将在新的虚拟文件系统内的单个合并目录中一起看到`。
>
>通过利用这一概念，Docker 可以避免数据重复，并且可以将先前创建的层用作以后构建的缓存。这样便产生了可在任何地方使用的紧凑，有效的镜像。

## 如何从源码构建NGINX

构建步骤：

* 准备Nginx源代码，[nignx源码下载](http://nginx.org/download/)

  ```shell
  ➜  nginxTest  wget http://nginx.org/download/nginx-1.19.9.tar.gz
  --2021-06-23 21:09:50--  http://nginx.org/download/nginx-1.19.9.tar.gz
  Connecting to 172.25.144.1:8889... connected.
  Proxy request sent, awaiting response... ^[[B200 OK
  Length: 1060580 (1.0M) [application/octet-stream]
  Saving to: ‘nginx-1.19.9.tar.gz’
  
  nginx-1.19.9.tar.gz         100%[=========================================>]   1.01M   398KB/s    in 2.6s
  
  2021-06-23 21:09:54 (398 KB/s) - ‘nginx-1.19.9.tar.gz’ saved [1060580/1060580]
  
  ➜  nginxTest  ls
  nginx-1.19.9.tar.gz
  ```

* 获得用于构建应用程序的基础镜像，例如 [ubuntu](https://hub.docker.com/_/ubuntu)。

* 在基础镜像上安装必要的构建依赖项。

* 复制 `nginx-1.19.9.tar.gz` 文件到镜像里。

* 解压缩压缩包的内容并删除压缩包。

* 使用 `make` 工具配置构建，编译和安装程序。

* 删除解压缩的源代码。

* 运行`nginx`可执行文件。

根据以上步骤创建Dockerfile如下：

```shell
FROM ubuntu:latest
RUN apt-get update && \
    apt-get install build-essential\ 
                    libpcre3 \
                    libpcre3-dev \
                    zlib1g \
                    zlib1g-dev \
                    libssl-dev \
                    -y && \
    apt-get clean && rm -rf /var/lib/apt/lists/*
COPY nginx-1.19.9.tar.gz .\
RUN tar -xvf nginx-1.19.9.tar.gz && rm nginx-1.19.9.tar.gz
RUN cd nginx-1.19.9 && \
    ./configure \
        --sbin-path=/usr/bin/nginx \
        --conf-path=/etc/nginx/nginx.conf \
        --error-log-path=/var/log/nginx/error.log \
        --http-log-path=/var/log/nginx/access.log \
        --with-pcre \
        --pid-path=/var/run/nginx.pid \
        --with-http_ssl_module && \
    make && make install
RUN rm -rf /nginx-1.19.9
CMD ["nginx", "-g", "daemon off;"]
```

 `COPY` 指令的通用语法是 `COPY <source> <destination>`，其中 source 在本地文件系统中，而 destination 在镜像内部。作为目标的 `.` 表示镜像内的工作目录，除非另有设置，否则默认为 `/`。

根据Dockerfile构建镜像：

```shell
➜  nginxTest  sudo docker image build --tag custom-nginx:built .
Sending build context to Docker daemon  1.064MB
Step 1/7 : FROM ubuntu:latest
 ---> 9873176a8ff5
Step 2/7 : RUN apt-get update &&     apt-get install build-essential                    libpcre3
       libpcre3-dev                     zlib1g                     zlib1g-dev                     libssl-dev                     -y &&     apt-get clean && rm -rf /var/lib/apt/lists/*
 ---> Using cache
 ---> 0badacf5e519
Step 3/7 : COPY nginx-1.19.9.tar.gz .
 ---> Using cache
 ---> 2d4f99ec42f5
Step 4/7 : RUN tar -xvf nginx-1.19.9.tar.gz && rm nginx-1.19.9.tar.gz
 ---> Using cache
 ---> 7a64789d9d5f
Step 5/7 : RUN cd nginx-1.19.9 &&     ./configure         --sbin-path=/usr/bin/nginx         --conf-path=/etc/nginx/nginx.conf         --error-log-path=/var/log/nginx/error.log         --http-log-path=/var/log/nginx/access.log         --with-pcre         --pid-path=/var/run/nginx.pid         --with-http_ssl_module &&     make && make install
 ---> Using cache
 ---> 8c5be265b082
Step 6/7 : RUN rm -rf /nginx-1.19.9
 ---> Using cache
 ---> 6e4d5674c0d1
Step 7/7 : CMD ["nginx", "-g", "daemon off;"]
 ---> Using cache
 ---> a8d8468e7f8a
Successfully built a8d8468e7f8a
Successfully tagged custom-nginx:built
```

启动镜像访问：

```shell
➜  nginxTest  sudo docker container run --rm --detach --publish 8888:80 --name nginx-custom custom-nginx:built

8928f42659485aba27737f95ec237c786d002bd88efadae300512ac8ce7fa8e5
```

![image-20210623212919846](/img/docker/image-20210623212919846.png)

`ARG`命令可以声明变量，此后可以用`${param}`引用。

`ADD`命令允许在构建的时候从互联网添加文件。

优化Dockerfile如下：

```shell
FROM ubuntu:latest
RUN apt-get update && \
    apt-get install build-essential\ 
                    libpcre3 \
                    libpcre3-dev \
                    zlib1g \
                    zlib1g-dev \
                    libssl-dev \
                    -y && \
    apt-get clean && rm -rf /var/lib/apt/lists/*
ARG FILENAME="nginx-1.19.9"
ARG EXTENSION="tar.gz"
ADD http://nginx.org/download/${FILENAME}.${EXTENSION} .
RUN tar -xvf ${FILENAME}.${EXTENSION} && rm ${FILENAME}.${EXTENSION}
RUN cd ${FILENAME} && \
    ./configure \
        --sbin-path=/usr/bin/nginx \
        --conf-path=/etc/nginx/nginx.conf \
        --error-log-path=/var/log/nginx/error.log \
        --http-log-path=/var/log/nginx/access.log \
        --with-pcre \
        --pid-path=/var/run/nginx.pid \
        --with-http_ssl_module && \
    make && make install
RUN rm -rf /${FILENAME}
CMD ["nginx", "-g", "daemon off;"]
```

>变量可以在构建镜像的时候用`--build-arg`传递
>
>```shell
> docker build --build-arg user=what_user .
>```

构建镜像：

```shell
➜  nginxTest  sudo docker image build --tag custom-nginx:built .
Sending build context to Docker daemon  1.064MB
Step 1/9 : FROM ubuntu:latest
 ---> 9873176a8ff5
Step 2/9 : RUN apt-get update &&     apt-get install build-essential                    libpcre3
       libpcre3-dev                     zlib1g                     zlib1g-dev                     libssl-dev                     -y &&     apt-get clean && rm -rf /var/lib/apt/lists/*
 ---> Using cache
 ---> 0badacf5e519
Step 3/9 : ARG FILENAME="nginx-1.19.9"
 ---> Using cache
 ---> 640cd1ce143a
Step 4/9 : ARG EXTENSION="tar.gz"
 ---> Using cache
 ---> d015cb4c0906
Step 5/9 : ADD http://nginx.org/download/${FILENAME}.${EXTENSION} .
Downloading  1.061MB/1.061MB
 ---> Using cache
 ---> 3d7e07a99d26
Step 6/9 : RUN tar -xvf ${FILENAME}.${EXTENSION} && rm ${FILENAME}.${EXTENSION}
 ---> Using cache
 ---> d28d4be633f5
Step 7/9 : RUN cd ${FILENAME} &&     ./configure         --sbin-path=/usr/bin/nginx         --conf-path=/etc/nginx/nginx.conf         --error-log-path=/var/log/nginx/error.log         --http-log-path=/var/log/nginx/access.log         --with-pcre         --pid-path=/var/run/nginx.pid         --with-http_ssl_module &&     make && make install
 ---> Using cache
 ---> 1f71f3ddda51
Step 8/9 : RUN rm -rf /${FILENAME}
 ---> Using cache
 ---> 07dacd933a51
Step 9/9 : CMD ["nginx", "-g", "daemon off;"]
 ---> Using cache
 ---> 8ea7b598f205
Successfully built 8ea7b598f205
Successfully tagged custom-nginx:built
```

## 如何优化Docker镜像

查看镜像会发现我们构建的镜像，大小为359M远大于官方镜像133M。

但是之前直接通过apt下载的方式构建的镜像`custom-nginx:packaged`大小只有132M，小于官方镜像，区别就是没有构建前下载的各种类库。

```shell
➜  nginxTest  sudo docker images
REPOSITORY            TAG        IMAGE ID       CREATED             SIZE
custom-nginx          built      8ea7b598f205   13 minutes ago      359MB
custom-nginx          packaged   b11ac2f129e6   About an hour ago   132MB
ubuntu                latest     9873176a8ff5   5 days ago          72.7MB
busybox               latest     69593048aa3a   2 weeks ago         1.24MB
node                  latest     d1b3088a17b1   2 weeks ago         908MB
nginx                 latest     d1a364dc548d   4 weeks ago         133MB
fhsinchy/hello-dock   latest     f540930e8157   4 months ago        21.9MB
fhsinchy/rmbyext      latest     90eafb66c390   5 months ago        50.9MB
```

查看构建镜像的Dockerfile

```shell
FROM ubuntu:latest
RUN apt-get update && \
    apt-get install build-essential\ 
                    libpcre3 \
                    libpcre3-dev \
                    zlib1g \
                    zlib1g-dev \
                    libssl-dev \
                    -y && \
    apt-get clean && rm -rf /var/lib/apt/lists/*
ARG FILENAME="nginx-1.19.9"
ARG EXTENSION="tar.gz"
ADD http://nginx.org/download/${FILENAME}.${EXTENSION} .
RUN tar -xvf ${FILENAME}.${EXTENSION} && rm ${FILENAME}.${EXTENSION}
RUN cd ${FILENAME} && \
    ./configure \
        --sbin-path=/usr/bin/nginx \
        --conf-path=/etc/nginx/nginx.conf \
        --error-log-path=/var/log/nginx/error.log \
        --http-log-path=/var/log/nginx/access.log \
        --with-pcre \
        --pid-path=/var/run/nginx.pid \
        --with-http_ssl_module && \
    make && make install
RUN rm -rf /${FILENAME}
CMD ["nginx", "-g", "daemon off;"]
```

RUN apt-get update && \
    apt-get install build-essential\ 
                    libpcre3 \
                    libpcre3-dev \
                    zlib1g \
                    zlib1g-dev \
                    libssl-dev \
                    -y && \
    apt-get clean && rm -rf /var/lib/apt/lists/* 下载编译镜像需要的类库。后续构建完镜像运行的时候只需要 `libpcre3` 和 `zlib1g`其他可以删除。

优化Dockerfile如下：

```shell
FROM ubuntu:latest

EXPOSE 80

ARG FILENAME="nginx-1.19.9"
ARG EXTENSION="tar.gz"

ADD https://nginx.org/download/${FILENAME}.${EXTENSION} .

RUN apt-get update && \
    apt-get install build-essential \ 
                    libpcre3 \
                    libpcre3-dev \
                    zlib1g \
                    zlib1g-dev \
                    libssl-dev \
                    -y && \
    tar -xvf ${FILENAME}.${EXTENSION} && rm ${FILENAME}.${EXTENSION} && \
    cd ${FILENAME} && \
    ./configure \
        --sbin-path=/usr/bin/nginx \
        --conf-path=/etc/nginx/nginx.conf \
        --error-log-path=/var/log/nginx/error.log \
        --http-log-path=/var/log/nginx/access.log \
        --with-pcre \
        --pid-path=/var/run/nginx.pid \
        --with-http_ssl_module && \
    make && make install && \
    cd / && rm -rfv /${FILENAME} && \
    apt-get remove build-essential \ 
                    libpcre3-dev \
                    zlib1g-dev \
                    libssl-dev \
                    -y && \
    apt-get autoremove -y && \
    apt-get clean && rm -rf /var/lib/apt/lists/*

CMD ["nginx", "-g", "daemon off;"]
```

重新构建：

```shell
➜  nginxTest  sudo docker image build --tag custom-nginx:built2 .
Sending build context to Docker daemon  1.064MB
Step 1/8 : FROM ubuntu:latest
 ---> 9873176a8ff5
Step 2/8 : RUN apt-get update &&     apt-get install build-essential                    libpcre3
       libpcre3-dev                     zlib1g                     zlib1g-dev                     libssl-dev                     -y &&     apt-get clean && rm -rf /var/lib/apt/lists/*
 ---> Using cache
 ---> 0badacf5e519
Step 3/8 : ARG FILENAME="nginx-1.19.9"
 ---> Using cache
 ---> 640cd1ce143a
Step 4/8 : ARG EXTENSION="tar.gz"
 ---> Using cache
 ---> d015cb4c0906
Step 5/8 : ADD http://nginx.org/download/${FILENAME}.${EXTENSION} .
Downloading  1.061MB/1.061MB
 ---> Using cache
 ---> 3d7e07a99d26
Step 6/8 : RUN tar -xvf ${FILENAME}.${EXTENSION} && rm ${FILENAME}.${EXTENSION}
 ---> Using cache
 ---> d28d4be633f5
Step 7/8 : RUN cd ${FILENAME} &&     ./configure         --sbin-path=/usr/bin/nginx         --conf-path=/etc/nginx/nginx.conf         --error-log-path=/var/log/nginx/error.log         --http-log-path=/var/log/nginx/access.log         --with-pcre         --pid-path=/var/run/nginx.pid         --with-http_ssl_module &&     make && make install &&     cd / && rm -rfv /${FILENAME} &&     apt-get remove build-essential                     libpcre3-dev                     zlib1g-dev                     libssl-dev                     -y &&     apt-get autoremove -y &&     apt-get clean && rm -rf /var/lib/apt/lists/*
 ---> Running in 648d256ec084
checking for OS
 + Linux 5.4.72-microsoft-standard-WSL2 x86_64
 ...
 Removing libroken18-heimdal:amd64 (7.7.0+dfsg-1ubuntu1) ...
Processing triggers for libc-bin (2.31-0ubuntu9.2) ...
Removing intermediate container 648d256ec084
 ---> 89a3115c6c9c
Step 8/8 : CMD ["nginx", "-g", "daemon off;"]
 ---> Running in b237b32f764e
Removing intermediate container b237b32f764e
 ---> f2a8f466d8dd
Successfully built f2a8f466d8dd
Successfully tagged custom-nginx:built2
```

查看大小：

```shell
➜  nginxTest  sudo docker images
REPOSITORY            TAG        IMAGE ID       CREATED          SIZE
custom-nginx          built2     592d52212ef2   8 seconds ago    84.1MB
custom-nginx          built      8ea7b598f205   43 minutes ago   359MB
custom-nginx          packaged   b11ac2f129e6   2 hours ago      132MB
ubuntu                latest     9873176a8ff5   5 days ago       72.7MB
busybox               latest     69593048aa3a   2 weeks ago      1.24MB
node                  latest     d1b3088a17b1   2 weeks ago      908MB
nginx                 latest     d1a364dc548d   4 weeks ago      133MB
fhsinchy/hello-dock   latest     f540930e8157   4 months ago     21.9MB
fhsinchy/rmbyext      latest     90eafb66c390   5 months ago     50.9MB
```

> custom-nginx:built 359MB -> custom-nginx:built2 84.1MB



