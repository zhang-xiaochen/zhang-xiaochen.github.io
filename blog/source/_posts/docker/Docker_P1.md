---
title: Docker_P1
excerpt: 容器、镜像、仓库概述，Docker架构描述以及Docker容器操作常用命令
index_img: /img/docker/docker.png
tags: [Docker]
categories: [容器]
date: 2021-06-12 17:30:00
---

# Docker_P1

> [Docker 入门教程 - 2021 最新版 (freecodecamp.org)](https://chinese.freecodecamp.org/news/the-docker-handbook/)

## 容器

*容器是应用程序层的抽象，可以将代码和依赖项打包在一起。容器不虚拟化整个物理机，仅虚拟化主机操作系统。*

虚拟机通常由称为虚拟机监控器的程序创建和管理，例如 [Oracle VM VirtualBox](https://www.virtualbox.org/)，[VMware Workstation](https://www.vmware.com/)，[KVM](https://www.linux-kvm.org/)，[Microsoft Hyper-V](https://docs.microsoft.com/zh-cn/virtualization/hyper-v-on-windows/about/) 等等。 该虚拟机监控程序通常位于主机操作系统和虚拟机之间，充当通信介质。

虚拟机：

![image-20210615211437030](/img/docker/image-20210615211437030.png)

在虚拟机内部运行的应用程序与 guest 操作系统进行通信，该 guest 操作系统在与虚拟机监控器进行通信，后者随后又与主机操作系统进行通信，以将必要的资源从物理基础设施分配给正在运行的应用程序。

容器：

![image-20210615211634919](/img/docker/image-20210615211634919.png)

在容器内部没有完整的 guest 操作系统，它只是通过容器运行时使用主机操作系统，同时保持隔离 – 就像传统的虚拟机一样。

容器运行时（即 Docker）位于容器和主机操作系统之间，而不是虚拟机监控器中。容器与容器运行时进行通信，容器运行时再与主机操作系统进行通信，以从物理基础设施中获取必要的资源。

## Docker镜像

镜像是分层只读文件，其中保留着应用程序所需的状态。

容器只是处于运行状态的镜像。

> 查看本地机器上的镜像 `docker images` 或 `docker image ls`

## 仓库

镜像仓库是一个集中式的位置，可以在其中上传镜像，也可以下载其他人创建的镜像。 [Docker Hub](https://hub.docker.com/) 是 Docker 的默认公共仓库。另一个非常流行的镜像仓库是 Red Hat 的 [Quay](https://quay.io/)。

## Docker架构描述

既然已经熟悉了有关容器化和 Docker 的大多数基本概念，那么现在是时候了解 Docker 作为软件的架构了。

该引擎包括三个主要组件：

1. **Docker 守护程序：** 守护程序（`dockerd`）是一个始终在后台运行并等待来自客户端的命令的进程。守护程序能够管理各种 Docker 对象。
2. **Docker 客户端：** 客户端（`docker`）是一个命令行界面程序，主要负责传输用户发出的命令。
3. **REST API：** REST API 充当守护程序和客户端之间的桥梁。使用客户端发出的任何命令都将通过 API 传递，最终到达守护程序。

根据官方[文档](https://docs.docker.com/get-started/overview/#docker-architecture),

> “ Docker 使用客户端-服务器体系结构。Docker *client* 与 Docker *daemon* 对话，daemon 繁重地构建、运行和分发 Docker 容器”。

作为用户，通常将使用客户端组件执行命令。然后，客户端使用 REST API 来访问长期运行的守护程序并完成工作。 

![image-20210615212459349](/img/docker/image-20210615212459349.png)

1. 执行 `docker run hello-world` 命令，其中 `hello-world` 是镜像的名称。
2. Docker 客户端访问守护程序，告诉它获取 `hello-world` 镜像并从中运行一个容器。
3. Docker 守护程序在本地仓库中查找镜像，并发现它不存在，所以在终端上打印 `Unable to find image 'hello-world:latest' locally`。
4. 然后，守护程序访问默认的公共仓库 Docker Hub，拉取 `hello-world` 镜像的最新副本，并在命令行中展示 `Unable to find image 'hello-world:latest' locally`。
5. Docker 守护程序根据新拉取的镜像创建一个新容器。
6. 最后，Docker 守护程序运行使用 `hello-world` 镜像创建的容器，该镜像在终端上输出文本。

## Docker容器操作常用命令

```shell
docker <object> <command> <options>
```

- `object` 表示将要操作的 Docker 对象的类型。这可以是 `container`、`image`、`network` 或者 `volume` 对象。
- `command` 表示守护程序要执行的任务，即 `run` 命令。
- `options` 可以是任何可以覆盖命令默认行为的有效参数，例如端口映射的 `--publish` 选项。

```shell
docker container run <image name>
```

image name 可以是在线仓库或者本地系统的任何镜像。

### 映射端口[--publish]

要允许从容器外部进行访问，必须将容器内的相应端口发布到本地网络上的端口。

```shell
--publish <host port>:<container port>
```

```shell
➜  chen5  sudo docker container run --publish 8080:80 fhsinchy/hello-dock
[sudo] password for chen:
Unable to find image 'fhsinchy/hello-dock:latest' locally
latest: Pulling from fhsinchy/hello-dock
0a6724ff3fcd: Pull complete
1d7c87af3754: Pull complete
9668ffa91d19: Pull complete
e81a2f5037c1: Pull complete
991b5ddb4d9e: Pull complete
9f4fab0aaa1b: Pull complete
Digest: sha256:852a90695e942a8aefe5883cb9681a3fbedfdf89f64468e22fa30e04766e5f2e
Status: Downloaded newer image for fhsinchy/hello-dock:latest
/docker-entrypoint.sh: /docker-entrypoint.d/ is not empty, will attempt to perform configuration
/docker-entrypoint.sh: Looking for shell scripts in /docker-entrypoint.d/
/docker-entrypoint.sh: Launching /docker-entrypoint.d/10-listen-on-ipv6-by-default.sh
10-listen-on-ipv6-by-default.sh: Getting the checksum of /etc/nginx/conf.d/default.conf
10-listen-on-ipv6-by-default.sh: Enabled listen on IPv6 in /etc/nginx/conf.d/default.conf
/docker-entrypoint.sh: Launching /docker-entrypoint.d/20-envsubst-on-templates.sh
/docker-entrypoint.sh: Configuration complete; ready for start up
```

![image-20210615213732401](/img/docker/image-20210615213732401.png)

### 容器后台运行[--detach]

```shell
---detach 或 -d 参数
```

```shell
➜  chen5  sudo docker container run --publish 8080:80 --detach fhsinchy/hello-dock
9918abf45820ca77a80eaa178d7ad66ee114547f3a5124a919e4b5e52b66f486
```

> --publish 和 --detach顺序没有要求

### 查看正在运行的容器[container ls]

```shell
container ls 
```



```shell
➜  chen5  sudo docker container ls
CONTAINER ID   IMAGE                 COMMAND                  CREATED         STATUS         PORTS
                       NAMES
9918abf45820   fhsinchy/hello-dock   "/docker-entrypoint.…"   2 minutes ago   Up 2 minutes   0.0.0.0:8080->80/tcp, :::8080->80/tcp   bold_dewdney
```

### 查看过去运行的容器[container ls --all]

```shell
--all 或 -a
```



```shell
➜  chen5  sudo docker container ls --all
CONTAINER ID   IMAGE                 COMMAND                  CREATED          STATUS                     PORTS                                   NAMES
9918abf45820   fhsinchy/hello-dock   "/docker-entrypoint.…"   9 minutes ago    Up 9 minutes               0.0.0.0:8080->80/tcp, :::8080->80/tcp   bold_dewdney
b5bcc6b696c9   fhsinchy/hello-dock   "/docker-entrypoint.…"   17 minutes ago   Exited (0) 9 minutes ago
                                    tender_lamarr
2c2ac99bf078   hello-world           "/hello"                 2 days ago       Exited (0) 2 days ago
```

### 指定容器名[--name]

默认情况下每个容器都有两个标识符 

* CONTAINER ID  -> 64个字符的随机字符串
* NAME -> 两个单词的随机组合，下滑线连接

```shell
--name
```

```shell
➜  chen5  sudo docker container run --publish 8888:80 --detach --name hello-docker-container fhsinchy/hello-dock
2925cec4da90141fa5f12f7fc38e22f3d01e7d5d3d9118bc63b5e145914bc1ee
➜  chen5  sudo docker container ls
CONTAINER ID   IMAGE                 COMMAND                  CREATED          STATUS          PORTS
                     NAMES
2925cec4da90   fhsinchy/hello-dock   "/docker-entrypoint.…"   13 seconds ago   Up 12 seconds   0.0.0.0:8888->80/tcp, :::8888->80/tcp   hello-docker-container
9918abf45820   fhsinchy/hello-dock   "/docker-entrypoint.…"   16 minutes ago   Up 16 minutes   0.0.0.0:8080->80/tcp, :::8080->80/tcp   bold_dewdney
```

### 重命名旧容器[container rename]

```shell
docker container rename <container identifier> <new name>
```

```shell
➜  chen5  sudo docker container rename bold_dewdney hello-docker-container2
➜  chen5  sudo docker container ls
CONTAINER ID   IMAGE                 COMMAND                  CREATED          STATUS          PORTS
                     NAMES
2925cec4da90   fhsinchy/hello-dock   "/docker-entrypoint.…"   3 minutes ago    Up 3 minutes    0.0.0.0:8888->80/tcp, :::8888->80/tcp   hello-docker-container
9918abf45820   fhsinchy/hello-dock   "/docker-entrypoint.…"   18 minutes ago   Up 18 minutes   0.0.0.0:8080->80/tcp, :::8080->80/tcp   hello-docker-container2
```

### 停止容器[container stop]

```shell
docker container stop <container identifier>
```

> container identifier 可以是 CONTAINER ID 或者NAME

```shell
➜  chen5  sudo docker container stop hello-docker-container
hello-docker-container
➜  chen5  sudo docker container ls
CONTAINER ID   IMAGE                 COMMAND                  CREATED          STATUS          PORTS
                     NAMES
9918abf45820   fhsinchy/hello-dock   "/docker-entrypoint.…"   21 minutes ago   Up 21 minutes   0.0.0.0:8080->80/tcp, :::8080->80/tcp   hello-docker-container2
```

`stop` 命令通过发送信号`SIGTERM` 来正常关闭容器。如果容器在一定时间内没有停止运行，则会发出 `SIGKILL` 信号，该信号会立即关闭容器。

如果要发送 `SIGKILL` 信号而不是 `SIGTERM` 信号，则可以改用 `container kill` 命令。`container kill` 命令遵循与 `stop` 命令相同的语法。

```shell
docker container kill <container identifier>
```

```shell
➜  chen5  sudo docker container kill hello-docker-container2
hello-docker-container2
➜  chen5  sudo docker container ls
CONTAINER ID   IMAGE     COMMAND   CREATED   STATUS    PORTS     NAMES
```

### 启动容器[container start]

```shell
docker container start <container identifier>
```

```shell
➜  chen5  sudo docker container start hello-docker-container
hello-docker-container
➜  chen5  sudo docker container ls
CONTAINER ID   IMAGE                 COMMAND                  CREATED         STATUS         PORTS
                   NAMES
2925cec4da90   fhsinchy/hello-dock   "/docker-entrypoint.…"   9 minutes ago   Up 3 seconds   0.0.0.0:8888->80/tcp, :::8888->80/tcp   hello-docker-container
```

### 重启容器[container restart]

```shell
docker container restart <container identifier>
```

```shell
➜  chen5  sudo docker container restart hello-docker-container
hello-docker-container
➜  chen5  sudo docker container ls
CONTAINER ID   IMAGE                 COMMAND                  CREATED          STATUS         PORTS
                    NAMES
2925cec4da90   fhsinchy/hello-dock   "/docker-entrypoint.…"   10 minutes ago   Up 2 seconds   0.0.0.0:8888->80/tcp, :::8888->80/tcp   hello-docker-container
```

### 创建容器[container create]

```shell
docker container create <options>
```

> `docker container run` 命令实际是 `docker container create` 和 `docker container start`两个命令的组合

```shell
➜  ~  sudo docker container create --publish 8080:80 fhsinchy/hello-dock
8b88edf8d01e531d7d96824a9add39ad79bf74c4bfde27deb58e1abc8c0de0e0
➜  ~  sudo docker container ls -all
CONTAINER ID   IMAGE                 COMMAND                  CREATED         STATUS    PORTS     NAMES
8b88edf8d01e   fhsinchy/hello-dock   "/docker-entrypoint.…"   4 seconds ago   Created             great_ganguly
```

### 移除挂起的容器[container rm]

```shell
docker container rm <container identifier>
```

> 已被停止或终止的容器仍保留在系统中。这些挂起的容器可能会占用空间或与较新的容器发生冲突

```shell
➜  ~  sudo docker container ls --all
CONTAINER ID   IMAGE                 COMMAND                  CREATED         STATUS                  PORTS     NAMES
c211f963a765   fhsinchy/hello-dock   "/docker-entrypoint.…"   4 seconds ago   Created                           amazing_turing
b5bcc6b696c9   fhsinchy/hello-dock   "/docker-entrypoint.…"   3 days ago      Exited (0) 3 days ago             tender_lamarr
2c2ac99bf078   hello-world           "/hello"                 6 days ago      Exited (0) 6 days ago             brave_golick
666d2f8672e2   hello-world           "/hello"                 6 days ago      Exited (0) 6 days ago             condescending_hellman
➜  ~  sudo docker container rm c211f963a765
c211f963a765
➜  ~  sudo docker container ls --all
CONTAINER ID   IMAGE                 COMMAND                  CREATED      STATUS                  PORTS     NAMES
b5bcc6b696c9   fhsinchy/hello-dock   "/docker-entrypoint.…"   3 days ago   Exited (0) 3 days ago             tender_lamarr
2c2ac99bf078   hello-world           "/hello"                 6 days ago   Exited (0) 6 days ago             brave_golick
666d2f8672e2   hello-world           "/hello"                 6 days ago   Exited (0) 6 days ago             condescending_hellman
➜  ~
```

可以一次删除多个容器，传入的标识符用空格隔开即可。

一次删除多个容器，也可以使用以下命令。

```shell
docker container 
```

```shell
➜  ~  sudo docker container ls --all
CONTAINER ID   IMAGE                 COMMAND                  CREATED      STATUS                  PORTS     NAMES
b5bcc6b696c9   fhsinchy/hello-dock   "/docker-entrypoint.…"   3 days ago   Exited (0) 3 days ago             tender_lamarr
2c2ac99bf078   hello-world           "/hello"                 6 days ago   Exited (0) 6 days ago             brave_golick
666d2f8672e2   hello-world           "/hello"                 6 days ago   Exited (0) 6 days ago             condescending_hellman
➜  ~  sudo docker container prune
WARNING! This will remove all stopped containers.
Are you sure you want to continue? [y/N] y
Deleted Containers:
b5bcc6b696c9eb6802bbd32dde3dc1586f2ebeee8cede3786fa03cf493454a95
2c2ac99bf078904481b0c7529b5ed5ec98dfbe7620662b5826a90daf90c8932d
666d2f8672e233e173dae327b9e5b60859a8b10fc1503070af56aebfefecb7ea

Total reclaimed space: 1.114kB
➜  ~  sudo docker container ls --all
CONTAINER ID   IMAGE     COMMAND   CREATED   STATUS    PORTS     NAMES
```

> 启动容器的时候有 `--rm` 参数，标识在容器停止运行的时候删除掉容器。

```
➜  ~  sudo docker container run --rm --publish 8888:80 --detach --name hello-docker-container fhsinchy/hello-dock
ab233d95e5cf2c5e4d020747d7a74c591947a6a049f15e11c0e1e21ada66205c
➜  ~  sudo docker container ls --all
CONTAINER ID   IMAGE                 COMMAND                  CREATED          STATUS          PORTS
    NAMES
ab233d95e5cf   fhsinchy/hello-dock   "/docker-entrypoint.…"   34 seconds ago   Up 33 seconds   0.0.0.0:8888->80/tcp, :::8888->80/tcp   hello-docker-container
➜  ~  sudo docker container stop hello-docker-container
hello-docker-container
➜  ~  sudo docker container ls --all
CONTAINER ID   IMAGE     COMMAND   CREATED   STATUS    PORTS     NAMES
```

### 以交互式模式启动容器[-it]

> 交互式镜像如 ubuntu之类的linux镜像，运行的时候进入到ubuntu的bash，启动的时候需要加  `-it`参数。
>
> - 选项 `-i` 或 `--interactive` 连接到容器的输入流，以便可以将输入发送到 bash。
> - `-t` 或 `--tty` 选项可通过分配伪 tty 来格式化展示并提供类似本机终端的体验。

```shell
➜ ~  sudo docker container run --rm -it ubuntu
Unable to find image 'ubuntu:latest' locally
latest: Pulling from library/ubuntu
c549ccf8d472: Pull complete
Digest: sha256:aba80b77e27148d99c034a987e7da3a287ed455390352663418c0f2ed40417fe
Status: Downloaded newer image for ubuntu:latest
root@269a69740972:/# cat /etc/os-release
NAME="Ubuntu"
VERSION="20.04.2 LTS (Focal Fossa)"
ID=ubuntu
ID_LIKE=debian
PRETTY_NAME="Ubuntu 20.04.2 LTS"
VERSION_ID="20.04"
HOME_URL="https://www.ubuntu.com/"
SUPPORT_URL="https://help.ubuntu.com/"
BUG_REPORT_URL="https://bugs.launchpad.net/ubuntu/"
PRIVACY_POLICY_URL="https://www.ubuntu.com/legal/terms-and-policies/privacy-policy"
VERSION_CODENAME=focal
UBUNTU_CODENAME=focal
root@269a69740972:/#
```

>需要注意的是在-it进入交互模式后，用exit退出会停止容器
>
>```shell
>➜  hello-dock (master) sudo docker container run --rm -it --name ubuntu ubuntu                                                                                                                            ✱
>root@5cf1025e8542:/# exit
>exit
>➜  hello-dock (master) sudo docker container ls                                                                                                                                                           ✱
>CONTAINER ID   IMAGE     COMMAND   CREATED   STATUS    PORTS     NAMES
>```

交互执行node，可以运行js代码

```shell
➜  ~  sudo docker run --rm -it node
Unable to find image 'node:latest' locally
latest: Pulling from library/node
d960726af2be: Pull complete
e8d62473a22d: Pull complete
8962bc0fad55: Pull complete
65d943ee54c1: Pull complete
532f6f723709: Pull complete
f8463f32765b: Pull complete
5714599515d2: Pull complete
0f0993710fe6: Pull complete
d09a14b7c3cb: Pull complete
Digest: sha256:e36cf1bb8719551220ba8c3ee1583881e79ad040803570e0849b00b8fe009153
Status: Downloaded newer image for node:latest
Welcome to Node.js v16.3.0.
Type ".help" for more information.
> var a = 1;
undefined
> var b = 2;
undefined
> a + b
3
```

### 进入容器[attach or exec -it]

启动一个ubuntu容器：

```shell
CONTAINER ID   IMAGE     COMMAND   CREATED   STATUS    PORTS     NAMES
➜  hello-dock (master) sudo docker container run --rm -dit --name ubuntu ubuntu                                                                                                                           ✱
0bbe6199d5edfc29b6dee3def6b82d33ea01e97da5c650c60a7eeebc9451693a
➜  hello-dock (master) sudo docker container ls                                                                                                                                                           ✱
CONTAINER ID   IMAGE     COMMAND   CREATED         STATUS         PORTS     NAMES
0bbe6199d5ed   ubuntu    "bash"    6 seconds ago   Up 5 seconds             ubuntu
```

`attach` 示例：

```shell
➜  hello-dock (master) sudo docker attach ubuntu                                                                                                                                                          ✱
root@7c7af33eec53:/# echo $SHELL
/bin/bash
root@7c7af33eec53:/# exit
exit
➜  hello-dock (master) sudo docker container ls                                                                                                                                                           ✱
CONTAINER ID   IMAGE     COMMAND   CREATED   STATUS    PORTS     NAMES
```

>`attach`命令在退出容器交互的时候会停止容器，不推荐使用

`exec -it`示例：

```shell
➜  hello-dock (master) sudo docker exec -it ubuntu bash                                                                                                                                                   ✱
root@5fdbf1bcfe1b:/# echo $SHELL
/bin/bash
root@5fdbf1bcfe1b:/# exit
exit
➜  hello-dock (master) sudo docker container ls                                                                                                                                                           ✱
CONTAINER ID   IMAGE     COMMAND   CREATED          STATUS          PORTS     NAMES
5fdbf1bcfe1b   ubuntu    "bash"    41 seconds ago   Up 39 seconds             ubuntu
```

### 在容器里执行命令

> 通过运行特定容器 让容器执行传入的命令。

```shell
docker container run <image name> <command>
```

```shell
➜  ~  sudo docker container run --rm busybox echo -n my-secret | base64
Unable to find image 'busybox:latest' locally
latest: Pulling from library/busybox
b71f96345d44: Pulling fs layer
b71f96345d44: Download complete
b71f96345d44: Pull complete
Digest: sha256:930490f97e5b921535c153e0e7110d251134cc4b72bbb8133c6a5065cc68580d
Status: Downloaded newer image for busybox:latest
bXktc2VjcmV0
```

> busybox -> The Swiss Army Knife of Embedded Linux

```shell
➜  ~  sudo docker container run ubuntu uname -a
Linux e090d6ac7e4b 5.4.72-microsoft-standard-WSL2 #1 SMP Wed Oct 28 23:40:43 UTC 2020 x86_64 x86_64 x86_64 GNU/Linux
```

### 处理可执行镜像

> [rmbyext](https://github.com/fhsinchy/rmbyext)一个递归删除给定扩展名文件的Python脚本。 rmbyext <file extension>
>
> [fhsinchy/rmbyext](https://hub.docker.com/r/fhsinchy/rmbyext) 镜像的行为与rmbyext类似。该镜像包含 `rmbyext` 脚本的副本，并配置为在容器内的目录 `/zone`上运行该脚本。
>
> 现在的问题是容器与本地系统隔离，因此在容器内运行的 `rmbyext` 程序无法访问本地文件系统。因此，如果可以通过某种方式将包含 pdf 文件的本地目录映射到容器内的 `/zone` 目录，则容器应该可以访问这些文件。
>
> 授予容器直接访问本地文件系统的一种方法是使用[绑定挂载](https://docs.docker.com/storage/bind-mounts/)。
>
> 绑定挂载可以在本地文件系统目录（源）与容器内另一个目录（目标）之间形成双向数据绑定。这样，在目标目录中进行的任何更改都将在源目录上生效，反之亦然。

为容器创建绑定挂载

```shell
-v 或 --volume
--volume <local file system directory absolute path>:<container file system directory absolute path>:<read write access>
```

挂载本地目录使用[fhsinchy/rmbyext](https://hub.docker.com/r/fhsinchy/rmbyext) 镜像的删除功能

```shell
➜  test  ls
t1.txt  t2.pdf  t3.jpg  t4.txt
➜  test  sudo docker container run --rm -v $(pwd):/zone fhsinchy/rmbyext pdf
Unable to find image 'fhsinchy/rmbyext:latest' locally
latest: Pulling from fhsinchy/rmbyext
801bfaa63ef2: Pull complete
8723b2b92bec: Pull complete
4e07029ccd64: Pull complete
594990504179: Pull complete
140d7fec7322: Pull complete
23038161e8da: Pull complete
0b40a42464a5: Pull complete
Digest: sha256:58969069a70a7f9b29be83abd1465cf10c568049c9d183e9d7a7d8726d074048
Status: Downloaded newer image for fhsinchy/rmbyext:latest
Removing: PDF
t2.pdf

➜  test  ls
delete_log.log  t1.txt  t3.jpg  t4.txt
➜  test  cat delete_log.log
t2.pdf
1 FILES DELETED.
➜  test
```

### 导入导出[export | import]

导出：

```shell
➜  ~  sudo docker container ls
CONTAINER ID   IMAGE     COMMAND     CREATED         STATUS         PORTS     NAMES
4a9d4431994f   alpine    "/bin/sh"   3 seconds ago   Up 2 seconds             alpine
5fdbf1bcfe1b   ubuntu    "bash"      6 minutes ago   Up 6 minutes             ubuntu
➜  ~  sudo docker export alpine > ~/imagebak/alpine.tar
➜  ~  ls ~/imagebak
alpine.tar
```

导入：

```shell
➜  ~  sudo cat ~/imagebak/alpine.tar | sudo docker import - test/alpine
sha256:08e69562ae6d5e7a656f9dc7123a226601df5e35bf1c0a768f9c2c0f666633b0
➜  ~  sudo docker images
REPOSITORY            TAG          IMAGE ID       CREATED          SIZE
test/alpine           latest       08e69562ae6d   28 seconds ago   5.59MB
uhuang/hello-dock     dev          53a1851c5630   56 minutes ago   189MB
custom-nginx          alphine      168baac68146   6 days ago       12.2MB
uhuang/custom-nginx   latest       168baac68146   6 days ago       12.2MB
custom-nginx          built2       592d52212ef2   10 days ago      84.1MB
custom-nginx          built        8ea7b598f205   10 days ago      359MB
custom-nginx          packaged     b11ac2f129e6   10 days ago      132MB
```

>*注：用户既可以使用* *`docker load`* *来导入镜像存储文件到本地镜像库，也可以使用* *`docker import`* *来导入一个容器快照到本地镜像库。这两者的区别在于容器快照文件将丢弃所有的历史记录和元数据信息（即仅保存容器当时的快照状态），而镜像存储文件将保存完整记录，体积也要大。此外，从容器快照文件导入时可以重新指定标签等元数据信息。*





