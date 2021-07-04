---
title: Docker_P3
excerpt: 编写开发Dockerfile、使用绑定挂载、使用匿名卷、多阶段构建、忽略不必要的文件
index_img: /img/docker/docker.png
tags: [Docker]
categories: [容器]
date: 2021-06-28 16:30:00
---

# Docker_P3

>以JavaScript应用为类，学习Docker在实际开发中的使用。
>
>可以在[这个仓库](https://github.com/fhsinchy/docker-handbook-projects/)中找到示例项目的代码，欢迎 ⭐。

## 如何编写开发Dockerfile

先从仓库拉代码，代码机构如下，我们将构建hello-dock项目：

```shell
➜  docker-handbook-projects (master) ls -l
total 68
-rw-r--r-- 1 chen chen  1211 Jul  4 10:17 LICENSE
-rw-r--r-- 1 chen chen  1338 Jul  4 10:17 README.md
drwxr-xr-x 2 chen chen  4096 Jul  4 10:17 custom-nginx
-rw-r--r-- 1 chen chen 38405 Jul  4 10:17 docker-handbook-github.png
drwxr-xr-x 5 chen chen  4096 Jul  4 10:17 fullstack-notes-application
drwxr-xr-x 4 chen chen  4096 Jul  4 10:17 hello-dock
drwxr-xr-x 3 chen chen  4096 Jul  4 10:17 notes-api
drwxr-xr-x 2 chen chen  4096 Jul  4 10:17 rmbyext
```

```shell
➜  hello-dock (master) ls -l
total 16
-rw-r--r-- 1 chen chen  324 Jul  4 10:17 index.html
-rw-r--r-- 1 chen chen  258 Jul  4 10:17 package.json
drwxr-xr-x 2 chen chen 4096 Jul  4 10:17 public
drwxr-xr-x 4 chen chen 4096 Jul  4 10:17 src
```

编写Dockerfile:

```dockerfile
FROM node:lts-alpine
EXPOSE 3000
USER node
RUN mkdir -p /home/node/app
WORKDIR /home/node/app
COPY ./package.json .
RUN npm install
COPY . .
CMD ["npm", "run", "dev"]
```

* node项目，以node为基础镜像，可以去Docker Hub查询需要的版本，以上版本是针对Alpine的长期支持版。
* `USER`指定用户为node，默认情况下，Docker 以 root 用户身份运行容器。 但是根据 [Docker and Node.js Best Practices](https://github.com/nodejs/docker-node/blob/master/docs/BestPractices.md)，这有安全隐患。因此，最好是尽可能以非 root 用户身份运行。node 镜像附带一个名为 `node` 的非 root 用户，可以使用 `USER` 指令将其设置为默认用户。
* `RUN mkdir -p /home/node/app`创建目录，Linux非root用户的工作目录默认为`/home/<user name>`。
* `WORKDIR`指定工作目录为`/home/node/app`。
* 先复制pacjage.json文件，用命令`npm install`安装依赖。然后再将项目其他文件复制进来。
* CMD 设置默认命令为`npm run dev`，此处为`exec`形式。

构建镜像：

```shell
➜  hello-dock (master) sudo docker image build --tag uhuang/hello-dock:dev --file ./Dockerfile.dev .        ✱
Sending build context to Docker daemon  30.21kB
Step 1/9 : FROM node:lts-alpine
 ---> c2ce8f082a62
Step 2/9 : EXPOSE 3000
 ---> Using cache
 ---> b3f64d55a2ed
Step 3/9 : USER node
 ---> Using cache
 ---> 5975d2407d4c
Step 4/9 : RUN mkdir -p /home/node/app
 ---> Using cache
 ---> 8bead4c5e7be
Step 5/9 : WORKDIR /home/node/app
 ---> Using cache
 ---> 9fbb7ef4c816
Step 6/9 : COPY ./package.json .
 ---> Using cache
 ---> 9eb745e57567
Step 7/9 : RUN npm install
 ---> Using cache
 ---> 1eb171c1ac95
Step 8/9 : COPY . .
 ---> Using cache
 ---> 118251b84039
Step 9/9 : CMD ["npm", "run", "dev"]
 ---> Using cache
 ---> 53a1851c5630
Successfully built 53a1851c5630
Successfully tagged uhuang/hello-dock:dev
```

运行镜像，访问页面：

```shell
➜  hello-dock (master) sudo docker container run --rm --detach --publish 3000:3000 --name hello-dock uhuang/he
llo-dock:dev
abccec9be7f494b5138d54502a592640914216c9be112eb899f7f25b200e1bb4
```

![image-20210704104315825](/img/docker/image-20210704104315825.png)

## 在Docker中使用绑定挂载

> 在容器中使用挂载可以直接从容器中引用本地文件系统，从而无需复制本地文件。

可以在运行容器的时候(`docker container run`， `docker container start`)绑定挂载`--volume`or`-v`，命令如下：

```shell
--volume <local file system directory absolute path>:<container file system directory absolute path>:<read write access>
```

挂载本地目录重启uhuang/hello-dock:dev镜像

```shell
docker container run \                                                          ✱
	   --rm \
  	   --publish 3000:3000 \
       --name hello-dock \
       --volume $(pwd):/home/node/app \
       uhuang/hello-dock:dev
```

```shell
➜  hello-dock (master) sudo docker container run \                                                          ✱
> --rm \
> --publish 3000:3000 \
> --name hello-dock \
> --volume $(pwd):/home/node/app \
> uhuang/hello-dock:dev

> hello-dock@0.0.0 dev /home/node/app
> vite

sh: vite: not found
npm ERR! code ELIFECYCLE
npm ERR! syscall spawn
npm ERR! file sh
npm ERR! errno ENOENT
npm ERR! hello-dock@0.0.0 dev: `vite`
npm ERR! spawn ENOENT
npm ERR!
npm ERR! Failed at the hello-dock@0.0.0 dev script.
npm ERR! This is probably not a problem with npm. There is likely additional logging output above.
npm WARN Local package.json exists, but node_modules missing, did you mean to install?

npm ERR! A complete log of this run can be found in:
npm ERR!     /home/node/.npm/_logs/2021-07-04T02_54_19_757Z-debug.log
```

以上启动命令省略了`--detach`选项，可以看到启动报错，原因是node项目的依赖存放在根目录的`node_modules`目录中，如下：

```shell
➜  ~  sudo docker exec -it hello-dock ash
~/app $ ls -l
total 128
-rw-r--r--    1 root     root           324 Jul  4 02:17 index.html
drwxr-sr-x    1 node     node          4096 Jul  4 03:38 node_modules
-rw-r--r--    1 node     node        107276 Jul  4 02:38 package-lock.json
-rw-r--r--    1 root     root           258 Jul  4 02:17 package.json
drwxr-xr-x    2 root     root          4096 Jul  4 02:17 public
drwxr-xr-x    4 root     root          4096 Jul  4 02:17 src
~/app $
```

被挂载的本地目录替换，本地目录是没有`node_modules`依赖的。此问题可以用匿名卷解决。

## 在Docker中使用匿名卷

使用匿名卷语法：

```shell
--volume <container file system directory absolute path>:<read write access>
```

可以理解为即使用本地文件系统，也使用容器文件系统。

添加匿名卷如下：

```shell
docker container run /
       --rm /
       --detach /
       --publish 3000:3000 /
       --name hello-dock /
       --volume $(pwd):/home/node/app /
       --volume /home/node/app/node_modules  /
       uhuang/hello-dock:dev
```

```shell
➜  hello-dock (master) sudo docker container run --rm --detach --publish 3000:3000 --name hello-dock --volume $(pwd):/home/node/app --volume /home/node/app/node_modules  uhuang/hello-dock:dev           ✱
226ee0909eb3dd840f8e02f5be27a4dbf0eba9841fba7ac577885ce972c25f35
➜  hello-dock (master) sudo docker container ls                                                                                                                                                           ✱
CONTAINER ID   IMAGE                   COMMAND                  CREATED         STATUS        PORTS                                       NAMES
226ee0909eb3   uhuang/hello-dock:dev   "docker-entrypoint.s…"   2 seconds ago   Up 1 second   0.0.0.0:3000->3000/tcp, :::3000->3000/tcp   hello-dock
```

## 在Docker中执行多阶段构建

>在开发模式下，`npm run serve` 命令启动一个开发服务器，该服务器将应用程序提供给用户。该服务器不仅提供文件，还提供热重载功能。
>
>在生产模式下，`npm run build` 命令将所有 JavaScript 代码编译为一些静态 HTML、CSS 和 JavaScript 文件。要运行这些文件，不需要 node 或任何其他运行时依赖项。只需要一个像 `nginx` 这样的服务器。
>
>按照以上思路构建生产模式的镜像：
>
>* 以node镜像为基础将项目编译静态文件。
>* 在node镜像的上装nginx，提供静态文件。
>
>以上思路有个问题，即node镜像比较大，并且node相关的依赖文件环境，对于最终生产环境来说是没有用的，我们完全可以在生产模式丢掉node相关依赖：
>
>* 以node为基础将项目编译为静态文件。
>* 把编译好的静态文件复制到nginx镜像中。

Dockerfile:

```dockerfile
FROM node:lts-alpine as builder

WORKDIR /app

COPY ./package.json ./
RUN npm install

COPY . .
RUN npm run build

FROM nginx:stable-alpine

EXPOSE 80

COPY --from=builder /app/dist /usr/share/nginx/html
```

* `FROM node:lts-alpine as builder` 中`as`指定别名，相当于变量，下文可以引用。
* `COPY --from=builder /app/dist /usr/share/nginx/html`中`--from`指定最初的构建镜像别名，从中复制文件到nginx。

构建镜像运行：

```shell
➜  hello-dock (master) sudo docker image build --tag uhuang/hello-dock:prd --file Dockerfile.prd .                                                                                                        ✱
Sending build context to Docker daemon  30.21kB
Step 1/9 : FROM node:lts-alpine as builder
 ---> c2ce8f082a62
Step 2/9 : WORKDIR /app
 ---> Using cache
 ---> ac43a9d70795
Step 3/9 : COPY ./package.json ./
 ---> Using cache
 ---> 890d6e4ea609
Step 4/9 : RUN npm install
 ---> Using cache
 ---> bf79ea514c9b
Step 5/9 : COPY . .
 ---> Using cache
 ---> 8382e8d4b236
Step 6/9 : RUN npm run build
 ---> Using cache
 ---> dce0ae8b77dd
Step 7/9 : FROM nginx:stable-alpine
 ---> e1ccef1fb908
Step 8/9 : EXPOSE 80
 ---> Using cache
 ---> fd53a267fb2b
Step 9/9 : COPY --from=builder /app/dist /usr/share/nginx/html
 ---> Using cache
 ---> 29f67e89417c
Successfully built 29f67e89417c
Successfully tagged uhuang/hello-dock:prd

➜  hello-dock (master) sudo docker container run --rm --detach --publish 8888:80 --name hello-dock-prd uhuang/hello-dock:prd                                                                              ✱
520be6184ddec93e2da3b21092b21f989e42ab8ec8b159f05fc957a6b4831d6a
```

![image-20210704121509111](/img/docker/image-20210704121509111.png)

> 如果要构建具有大量依赖关系的大型应用程序，那么多阶段构建可能会非常有用。如果配置正确，则可以很好地优化和压缩分多个阶段构建的镜像。

## 忽略不被要的文件

>类似于git的`.gigiginore`文件，在``docker`中`.dockerignore` 文件包含要从镜像构建中排除的文件和目录的列表。

```shell
.git
*Dockerfile*
*docker-compose*
node_modules
```

* `.dockerignore` 文件必须位于构建上下文中。这里提到的文件和目录将被 `COPY` 指令忽略。但是，如果执行绑定挂载，则 `.dockerignore` 文件将对此无效。

