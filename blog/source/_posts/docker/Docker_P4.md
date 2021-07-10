---
title: Docker_P4
excerpt: Docker网路基础、Docker中创用户自定义的桥联网络、Docker中将容器连接到网络、Docker中从网络分离容器、删除 Docker 中的网络
index_img: /img/docker/docker.png
tags: [Docker]
categories: [容器]
date: 2021-07-04 10:34:00
---

# Docker_P4

>实际项目中，应用程序、缓存Redis、数据库等可能是分别部署在不同的容器，这时候各容器该如何连接？
>
>[Netwoek_Doc]([Networking overview | Docker Documentation](https://docs.docker.com/network/))

## Docker网路基础

> 和镜像、容器一样，网络可以视为Docker的一个逻辑对象，默认有如下三个网络。

```shell
➜  ~  sudo docker network ls
NETWORK ID     NAME      DRIVER    SCOPE
9e0b2c105163   bridge    bridge    local
8b273f0ea115   host      host      local
800d433d1305   none      null      local
```

Driver列表示网络类型，默认情况下，Docker有五种网络类型：

- `bridge` - Docker 中的默认网络驱动程序。当多个容器以标准模式运行并且需要相互通信时，可以使用此方法。
- `host` - 完全消除网络隔离。在 `host` 网络下运行的任何容器基本上都连接到主机系统的网络。
- `none` - 此驱动程序完全禁用容器的联网。 我还没有找到其应用场景。
- `overlay` - 这用于跨计算机连接多个 Docker 守护程序，这超出了本书的范围。
- `macvlan` - 允许将 MAC 地址分配给容器，使它们的功能类似于网络中的物理设备。

> 也有第三方插件，可让你将 Docker 与专用网络堆栈集成。
>
> 以下使用bridge类型的网络演示

## Docker中创用户自定义的桥联网络

默认情况下，容器运行将自动连接到名为`bridge`的桥联网络。

```shell
➜  ~  sudo docker container run --rm --detach --publish 8888:80 --name hello-dock uhuang/hello-dock:prd
fec72e0016abdfa3ed4b57cd6f359531d2b6d5781d1a9f97a58a267c833ea61e
➜  ~  sudo docker network inspect --format='{{range .Containers}}{{.Name}}{{end}}' bridge
hello-dock
```

>默认连接到bridge桥联网络的容器，相互通信可以使用IP地址，获取方式如下：
>
>```shell
>➜  ~  sudo docker container inspect --format='{{range .NetworkSettings.Networks}} {{.IPAddress}} {{end}}' hell
>o-dock
> 172.17.0.2
>```
>
>不建议使用 IP 地址来引用容器。另外，如果容器被破坏并重新创建，则 IP 地址可能会更改。跟踪这些不断变化的 IP 地址可能非常麻烦。正确的做法应该是，**将它们放置在用户定义的桥接网络下即可将它们连接起来。**

用户定义的桥接网络比默认桥接网络多一些额外的功能。根据有关此主题的官方 [docs](https://docs.docker.com/network/bridge/#differences-between-user-defined-bridges-and-the-default-bridge)，一些值得注意的额外功能如下：

- **用户定义的网桥可在容器之间提供自动 DNS 解析：** 这意味着连接到同一网络的容器可以使用容器名称相互通信。 因此，如果你有两个名为 `notes-api` 和 `notes-db` 的容器，则 API 容器将能够使用 `notes-db` 名称连接到数据库容器。
- **用户定义的网桥提供更好的隔离性：** 默认情况下，所有容器都连接到默认桥接网络，这可能会导致它们之间的冲突。将容器连接到用户定义的桥可以确保更好的隔离。
- **容器可以即时与用户定义的网络连接和分离：** 在容器的生命周期内，可以即时将其与用户定义的网络连接或断开连接。要从默认网桥网络中删除容器，需要停止容器并使用其他网络选项重新创建它。

创建网络命令：

```shell
docker network create <network name>
```

```shell
➜  ~  sudo docker network create mynet
96c0880ef194676015bb3fd91d70e1160d8f89e52b6f2f8139cbb24211d56be8
➜  ~  sudo docker network ls
NETWORK ID     NAME      DRIVER    SCOPE
9e0b2c105163   bridge    bridge    local
8b273f0ea115   host      host      local
96c0880ef194   mynet     bridge    local
800d433d1305   none      null      local
```

## Docker中将容器连接到网络

方式一：

```shell
ocker network connect <network identifier> <container identifier>
```

```shell
➜  ~  sudo docker network connect mynet hello-dock
➜  ~  sudo docker network inspect --format='{{range .Containers}}{{.Name}}{{end}}' mynet
hello-dock
➜  ~  sudo docker network inspect --format='{{range .Containers}}{{.Name}}{{end}}' bridge
hello-dock
```

>hello-dock容器现在连接到了`mybet`和`bridge`两个网络上。

方式二：

将容器连接到网络的第二种方法是对 `container run` 或 `container create` 命令使用 `--network` 选项。 该选项的通用语法如下：

```shell
--network <network identifier>
```

```shell
➜  ~  sudo docker container run --rm --detach  --network  mynet --name apline-box -it alpine
ecae679ff69c70933507e72f7f4c1724451140d9e3fd389720cb6d177263384d
➜  ~  sudo docker network inspect --format='{{range .Containers}}{{.Name}}{{end}}' mynet
apline-box hello-dock
➜  ~  sudo docker network inspect --format='{{range .Containers}}{{.Name}}{{end}}' bridge
hello-dock
➜  ~  sudo docker exec -it apline-box sh
/ # ping hello-dock
PING hello-dock (172.18.0.2): 56 data bytes
64 bytes from 172.18.0.2: seq=0 ttl=64 time=0.146 ms
64 bytes from 172.18.0.2: seq=1 ttl=64 time=0.237 ms
64 bytes from 172.18.0.2: seq=2 ttl=64 time=0.196 ms
64 bytes from 172.18.0.2: seq=3 ttl=64 time=0.202 ms
64 bytes from 172.18.0.2: seq=4 ttl=64 time=0.197 ms
^C
--- hello-dock ping statistics ---
5 packets transmitted, 5 packets received, 0% packet loss
round-trip min/avg/max = 0.146/0.195/0.237 ms
/ #
```

>为了使自动 DNS 解析正常工作，必须为容器分配自定义名称。使用随机生成的名称将不起作用。

## Docker中从网络分离容器

命令如下：

```shell
docker network disconnect <network identifier> <container identifier>
```

```shell
➜  ~  sudo docker network inspect --format='{{range .Containers}}{{.Name}}{{end}}' mynet
apline-boxhello-dock
Disconnect a container from a network
➜  ~  sudo docker network disconnect mynet hello-dock
➜  ~  sudo docker network inspect --format='{{range .Containers}}{{.Name}}{{end}}' mynet
apline-box
```

## 删除 Docker 中的网络

命令如下：

```shell
docker network rm <network identifier>
```

要从系统中删除 `mynet` 网络，可以执行以下命令：

```shell
docker network rm mynet
```

也可以使用 `network prune` 命令从系统中删除所有未使用的网络。该命令还具有 `-f` 或 `--force` 和 `-a` 或 `--all` 选项。

