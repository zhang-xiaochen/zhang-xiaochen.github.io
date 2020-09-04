---
title: Nginx
tags: [Nginx]
categories: [web服务器]
date: 2020-08-29 17:30:00
---
# Nginx

>**Nginx**（发音同“engine X”）是异步框架的[网页服务器](https://zh.wikipedia.org/wiki/網頁伺服器)，也可以用作[反向代理](https://zh.wikipedia.org/wiki/反向代理)、[负载平衡器](https://zh.wikipedia.org/wiki/负载均衡)和[HTTP缓存](https://zh.wikipedia.org/wiki/HTTP缓存)。该软件由[伊戈尔·赛索耶夫](https://zh.wikipedia.org/wiki/伊戈爾·賽索耶夫)创建并于2004年首次公开发布[[6\]](https://zh.wikipedia.org/wiki/Nginx#cite_note-Mobily-6)。2011年成立同名公司以提供支持[[7\]](https://zh.wikipedia.org/wiki/Nginx#cite_note-D-7)。2019年3月11日，Nginx公司被[F5 Networks](https://zh.wikipedia.org/w/index.php?title=F5_Networks&action=edit&redlink=1)以6.7亿美元收购[[8\]](https://zh.wikipedia.org/wiki/Nginx#cite_note-8)。
>
>Nginx是免费的[开源软件](https://zh.wikipedia.org/wiki/开源软件)，根据类[BSD许可证](https://zh.wikipedia.org/wiki/BSD许可证)的条款发布。一大部分Web服务器使用Nginx[[9\]](https://zh.wikipedia.org/wiki/Nginx#cite_note-9)，通常作为[负载均衡器](https://zh.wikipedia.org/wiki/负载均衡)。[[10\]](https://zh.wikipedia.org/wiki/Nginx#cite_note-10)

* Nginx可以部署在网络上使用[FastCGI](https://zh.wikipedia.org/wiki/FastCGI)脚本、[SCGI](https://zh.wikipedia.org/wiki/简单通用网关接口)处理程序、[WSGI](https://zh.wikipedia.org/wiki/Web服务器网关接口)应用服务器或[Phusion Passenger](https://zh.wikipedia.org/w/index.php?title=Phusion_Passenger&action=edit&redlink=1)模块的动态[HTTP](https://zh.wikipedia.org/wiki/HTTP)内容，并可作为软件[负载均衡器](https://zh.wikipedia.org/wiki/负载均衡)。
* Nginx使用异步事件驱动的方法来处理请求。Nginx的模块化事件驱动架构[[12\]](https://zh.wikipedia.org/wiki/Nginx#cite_note-aosabook-12)可以在高负载下提供更可预测的性能[[13\]](https://zh.wikipedia.org/wiki/Nginx#cite_note-Configuration-13)。
* Nginx是一款面向性能设计的HTTP服务器，相较于[Apache](https://zh.wikipedia.org/wiki/Apache_HTTP_Server)、[lighttpd](https://zh.wikipedia.org/wiki/Lighttpd)具有占有[内存](https://zh.wikipedia.org/wiki/内存)少，稳定性高等优势。
* Nginx不采用每客户机一线程的设计模型，而是充分使用异步逻辑从而削减了上下文调度开销，所以并发服务能力更强。
* Nginx 主进程ID会被写进nginx.pid文件中，位于/usr/local/nginx/logs或/var/run路径下。![image-20200820163554437](/img/Nginx.assets/image-20200820163554437.png)
* Nginx日志路径`/usr/local/nginx/logs` 或 `/var/log/nginx`![image-20200820185305153](/img/Nginx.assets/image-20200820185305153.png)

## 简单入门

### 安装

>ubuntu 20.04

```shell
# 安装
~ ❯ sudo apt install nginx
~ ❯ whereis nginx                                                                    7s 09:59:46 AM
nginx: /usr/sbin/nginx /usr/lib/nginx /etc/nginx /usr/share/nginx /usr/share/man/man8/nginx.8.gz
# 版本
~ ❯ nginx -v                                                                            09:59:52 AM
nginx version: nginx/1.14.0 (Ubuntu)
~ ❯ cd /etc/nginx
```

* 配置文件`nginx.conf` 路径为`/usr/local/nginx/conf`， `/etc/nginx`或 `/usr/local/etc/nginx`。![](/img/Nginx.assets/image-20200820110747003.png)

* Nginx 有**一个主进程**和**多个工作进程**。
* 主进程主要用来**读取验证配置**以及**维护工作进程**。
* 工作进程用来**处理请求**。
* Nginx使用**事件驱动**和**基于操作系统**在工作进程之间高效分发请求。
* 工作进程数量定义在配置文件中(默认自动分配的为当前**cpu核心数**)![](/img/Nginx.assets/image-20200820133704669.png)

### 启动命令

~~~shell
# 查询当前nginx服务状态
/etc/nginx ❯ sudo service nginx status                                                  10:53:10 AM
 * nginx is not running
 # 启动
/etc/nginx ❯ sudo service nginx start                                                   10:53:22 AM
 * Starting nginx nginx
~~~

访问localhost

![image-20200820111650637](/img/Nginx.assets/image-20200820111650637.png)

常用命令 nginx -s signal

signa取值：

* stop -> 快速关机
* quit -> 处理完当前请求 关机 
* reload -> 重新加载配置文件
* reopen -> 重新打开日志

查询nginx进程：

```shell
ps -ax | grep nginx
```

![image-20200820164024523](/img/Nginx.assets/image-20200820164024523.png)

> `nginx -s quit` 会等待当前请求处理完成再去关机 该命令需要启动当前Ngnix的用户执行

> `nginx -s reload` 主线程接收到重载配置文件的信号，会验证加载使用新的配置信息。如果加载成功，则会开启新的工作进程去接收请求，通知旧的工作进程shut down， 旧的进程接收到通知后会停止接收新的请求，并且将手里的请求处理完成，然后去exit。否则主进程回滚修改，继续使用旧的配置信息。

> 以下命令也可以平滑关闭Nginx
>
> ~~~she
> kill -s QUIT 1628
> ~~~

### 配置文件结构

初始默认配置文件：

```she
user www-data;
worker_processes auto;
pid /run/nginx.pid;
include /etc/nginx/modules-enabled/*.conf;

events {
        worker_connections 768;
        # multi_accept on;
}

http {

        ##
        # Basic Settings
        ##

        sendfile on;
        tcp_nopush on;
        tcp_nodelay on;
        keepalive_timeout 65;
        types_hash_max_size 2048;
        # server_tokens off;

        # server_names_hash_bucket_size 64;
        # server_name_in_redirect off;

        include /etc/nginx/mime.types;
        default_type application/octet-stream;

        ##
        # SSL Settings
        ##

        ssl_protocols TLSv1 TLSv1.1 TLSv1.2; # Dropping SSLv3, ref: POODLE
        ssl_prefer_server_ciphers on;

        ##
        # Logging Settings
        ##

        access_log /var/log/nginx/access.log;
        error_log /var/log/nginx/error.log;

        ##
        # Gzip Settings
        ##

        gzip on;

        # gzip_vary on;
        # gzip_proxied any;
        # gzip_comp_level 6;
        # gzip_buffers 16 8k;
        # gzip_http_version 1.1;
        # gzip_types text/plain text/css application/json application/javascript text/xml application/xml application/xml+rss text/javascript;

        ##
        # Virtual Host Configs
        ##

        include /etc/nginx/conf.d/*.conf;
        include /etc/nginx/sites-enabled/*;
}


#mail {
#       # See sample authentication script at:
#       # http://wiki.nginx.org/ImapAuthenticateWithApachePhpScript
#
#       # auth_http localhost/auth.php;
#       # pop3_capabilities "TOP" "USER";
#       # imap_capabilities "IMAP4rev1" "UIDPLUS";
#
#       server {
#               listen     localhost:110;
#               protocol   pop3;
#               proxy      on;
#       }
#
#       server {
#               listen     localhost:143;
#               protocol   imap;
#               proxy      on;
#       }
#}
```



* Nginx由不同模块组成，模块以指令（参数）的方式定义在配置文件中。指令分简单指令和块级指令。

* 简单指令格式为：指令名+空格+参数值+；

![image-20200820165610355](/img/Nginx.assets/image-20200820165610355.png)

* 块级指令格式为：指令名 {简单指令}

![image-20200820165729596](/img/Nginx.assets/image-20200820165729596.png)

* #后为注释内容。

  > 可以认为整个配置文件用{}包裹，则其上下文暂时理解为作用域吧

* 建议按照不同功能模块将配置文件拆分为一组文件，放在`/etc/nginx/conf.d`文件夹中,在`nginx.conf`配置文件中使用`incloud`命令引入

  ![image-20200829113211025](/img/Nginx.assets/image-20200829113211025.png)

* Contexts 

  * events - 处理常规流量
  * http - 处理http流量
  * mail - 处理mail流量
  * stream - 流量TCP、UDP流量

  > 不在以上四种块级指令中定义的命令 则在主上下文中生效

* server 是定义在上下文中的虚拟服务器，每个上下文中都可以定义多个虚拟服务器，至于server处理何种流量取决于其上下文可以处理何种流量。

* 子上下文中的设置继承于父上下文，可以在子上下文中重新定义来覆盖，重写

### 静态资源服务器

```shell
http {
        server {
                location / {
                        root /home/chen/blog;
                }
                location /img/ {
                        root /home/chen/blog/public;
                }
        }
}
```

---



/home/chen/blog文件如下：

![image-20200820184348402](/img/Nginx.assets/image-20200820184348402.png)

http://localhost/package.json

![image-20200820184433253](/img/Nginx.assets/image-20200820184433253.png)

匹配到/home/chen/blog文件夹下的package.json文件

---

root /home/chen/blog/public文件如下：

![image-20200820184905687](/img/Nginx.assets/image-20200820184905687.png)

http://localhost/img/default.png

![image-20200820184930795](/img/Nginx.assets/image-20200820184930795.png)

匹配到/home/chen/blog/public文件夹下的./img/default.png文件

### 代理服务器

> **正向代理**：隐藏客户端，客户端通过代理服务器A访问真实的服务器B，服务器B并不知道访问自己的客户端是谁。
>
> **反向代理**：隐藏服务器，客户端访问代理服务器A，代理服务器将请求分发到真是的服务器BCD等，客户端只知道访问了服务器A，实际内容是BCD提供的。



> 客户端有无感知：
>
> **普通代理**：需要客户端指定代理服务器ip端口号，常用的vpn。
>
> **透明代理**：客户端无感知，企业服务路由GateWay。

* 简单静态资源代理

> 算是反向代理吧 将80端口的请求分发到8080

1. 配置文件新增server

```shll
server {
	listen 8080;
	root /home/chen/blog/public;
	location / {

	}
}
```

监听8080端口，将该端口的所有请求（/）匹配到服务器路径/home/chen/blog下的资源。

> 在之前静态资源服务器中，因为80是默认端口，可以不用listen指定端口。

资源目录/home/chen/blog/public

![image-20200821143422848](/img/Nginx.assets/image-20200821143422848.png)



请求：http://192.168.240.239:8080/index.html

![image-20200821143641539](/img/Nginx.assets/image-20200821143641539.png)

2. 修改上节的静态资源服务器

```shell
 server {
                location / {
                        proxy_pass http://localhost:8080;
                }
                location /img/ {
                        root /home/chen/blog/public;
                }
        }
```

80端口的请求会被代理到本地8080端口，也就是匹配1.中的服务器，进而匹配/home/chen/blog/public下面的资源，现在访问http://192.168.240.239/index.html会访问到/home/chen/blog/public路径下面的index.html页面（<u>修改之前的/home/chen/blog路径下面是没有index.html页面的</u>），如下图：

![image-20200821144416050](/img/Nginx.assets/image-20200821144416050.png)

3. 修改 图片location使其匹配特定文件

~~~shell
 server {
                location / { # location1
                        proxy_pass http://localhost:8080;
                }
                location ~\.(gif|jpg|png)$ { # location2
                        root /home/chen/blog/public/img;
                }
        }
~~~

~标识当前文件夹，后面匹配正则表达式。

访问http://192.168.240.239/default.png

* 注意当前匹配的是location2

![image-20200821150134637](/img/Nginx.assets/image-20200821150134637.png)

访问http://192.168.240.239/img/default.png

* 当前匹配的是location1

![image-20200821150613246](/img/Nginx.assets/image-20200821150613246.png)

> location1 代理到8080端口访问，从而映射/home/chen/blog/public文件夹，img文件夹刚好在此文件夹下面因此可以http://192.168.240.239/img/default.png访问，尝试访问其他子文件夹的文件则可以区分是走的哪一个location
>
> ~~~shell
>  location ~\.(gif|jpg|png|css)$ {
>  	root /home/chen/blog/public/img;
>  }
> ~~~
>
> http://192.168.240.239/css/main.css (location1 -> 代理到8080 -> /home/chen/blog/public -> css/main.css)
>
> ![image-20200821151514613](/img/Nginx.assets/image-20200821151514613.png)
>
> http://192.168.240.239/main.css (location2 -> /home/chen/blog/public/img -> main.css（404）)
>
> ![image-20200821151545926](/img/Nginx.assets/image-20200821151545926.png)

---

## Nginx处理请求过程

1. 先选定虚拟主机处理请求

资源目录文件如下：

![image-20200821164941794](/img/Nginx.assets/image-20200821164941794.png)

---

server配置：

```shell
server {
	listen 8080;
	root /home/chen/blog/public/css;
	location / {
	}
}
server {
	listen 8080;
	root /home/chen/blog/public/js;
	ocation / {
	}
}
server {
	listen 8080;
	root /home/chen/blog/public/img;
	location / {
	}
}
```



请求第一个server: http://192.168.240.239:8080/main.css

![image-20200821165142399](/img/Nginx.assets/image-20200821165142399.png)

请求第二个server: http://192.168.240.239:8080/main.js

![image-20200821165211353](/img/Nginx.assets/image-20200821165211353.png)

请求第三个server: http://192.168.240.239:8080/default.png

![image-20200821165327762](/img/Nginx.assets/image-20200821165327762.png)

> 以上配置server一样，nginx会默认选用第一个server.



---

server配置:

```shell
 server {
                listen 8080;
                root /home/chen/blog/public/css;
                location / {
                }
        }
        server {
                listen 8080 default_server; #default_server 指定默认端口号
                root /home/chen/blog/public/js;
                location / {
                }
        }
        server {
                listen 8080;
                root /home/chen/blog/public/img;
                location / {
                }
        }
```

请求第一个server: http://192.168.240.239:8080/main.css

![image-20200821170424132](/img/Nginx.assets/image-20200821170424132.png)

请求第二个server: http://192.168.240.239:8080/main.js

![image-20200821170439230](/img/Nginx.assets/image-20200821170439230.png)

请求第三个server: http://192.168.240.239:8080/default.png

![image-20200821170543463](/img/Nginx.assets/image-20200821170543463.png)

> default_server 指定默认server监听的端口 不同的监听端口可以设置不同的默认服务器

---

server配置：

```shell
 server {
                listen 8088;
                server_name www.test.com;
                root /home/chen/blog/public/css;
                location / {
                }
        }
        server {
                listen 8080 default_server;
                server_name www.test.com;
                root /home/chen/blog/public/js;
                location / {
                }
        }
        server {
                listen 8080;
                server_name www.test.com;
                root /home/chen/blog/public/img;
                location / {
                }
        }
```

请求第一个server: http://192.168.240.239:8080/main.css

![image-20200821171844568](/img/Nginx.assets/image-20200821171844568.png)

请求第二个server: http://192.168.240.239:8080/main.js

![image-20200821171906824](/img/Nginx.assets/image-20200821171906824.png)

请求第三个server: http://192.168.240.239:8080/default.png

![image-20200821171932993](/img/Nginx.assets/image-20200821171932993.png)

> 根据ip和端口号匹配server,根据host匹配server_name，未匹配上则走默认server

---

丢弃未定义主机名的请求：

```shell
server {
    listen       80;
    server_name  "";
    return       444;
}
```

## 虚拟主机名

server_name 可以用确定的名字、通配符、正则表达式定义。

```shell
server {
    listen       80;
    server_name  www.test.com; # 确定的名字
}
server {
    listen       80;
    server_name  *test.com; # 通配符 以test.com结尾的请求
}
server {
    listen       80;
    server_name  mail.*; # 通配符 以mail.开头的请求
}
server {
    listen       80;
    server_name  ~^(?<user>.+)\.example\.net$; # 正则
}
```

如果匹配到多个server_name则按照以下优先级：

1. 准确名字
2. 最长的以*起始的名字
3. 最长的以*结尾的名字
4. 正则匹配的名字



其他类型：

​	“” 非默认主机处理请求头不含host字段的请求

​	IP 以IP地址来请求服务器



* 确切名字和通配符名字存在哈希表中，哈希表和监听端口关联。nginx先找确切名字的哈希表，再找通配符名字的哈希表。正则主机名则是最后查找，并且是串行查找，效率最慢，因此最好用确定主机名。
* 定义了大量的名字或十分长的名字，则需要在*http*配置块中使用server_names_hash_max_size和server_names_hash_bucket_size指令进行调整。

## 请求处理方式

​	nginx支持多种请求处理方式，可以使用哪些处理方式取决于平台。平台支持多种方式，则nginx会默认选用最高效的方式处理。也可以用use指令明确指定处理方式。

nginx支持的请求处理方式：

- `select` — 标准方法。 在平台不支持更高效的方法时，nginx会自动编译此模块。 可以使用`--with-select_module`和`--without-select_module`编译选项 强行开启或禁止编译此模块。

- `poll` — 标准模块。 在平台不支持更高效的方法时，nginx会自动编译此模块。 可以使用`--with-poll_module`和`--without-poll_module`编译选项 强行开启或禁止编译此模块。

- `kqueue` — FreeBSD 4.1+、OpenBSD 2.9+、NetBSD 2.0和Mac OS X的高效方法。

- `epoll` — Linux 2.6+的高效方法。

  > 一些旧的发行版，比如SuSE 8.2，提供了补丁，在2.4内核上支持了epoll方法。

  

- `rtsig` — 实时信号，Linux 2.2.19+的高效方法。 系统级的事件队列默认有1024个信号的限制。在高负载的服务器上，将此限制上调可能是必须的。 调整的方法是改变`/proc/sys/kernel/rtsig-max`内核参数的值。 在Linux 2.6.6-mm2上，这个参数不存在，而且每个进程拥有自己的事件队列。 每个队列的长度由`RLIMIT_SIGPENDING`所限，并可使用worker_rlimit_sigpending指令修改。

  队列溢出时，nginx丢弃这个队列，并回退到`poll`连接处理方法，直到情况恢复正常为止。

- `/dev/poll` — Solaris 7 11/99+、HP/UX 11.22+ (eventport)、IRIX 6.5.15+和Tru64 UNIX 5.1A+的高效方法。

- `eventport` — 事件端口，Solaris 10的高效方法。

## Nginx控制命令

> 主进程的PID会在Nginx启动的时候写入到<u>/usr/local/nginx/logs</u>或<u>/var/run</u>路径下的nginx.pid文件。
>
> pid可以在<u>/etc/nginx/nginx.conf</u> 文件中用pid来指定。
>
> ```shell
> kill -[signal] [pid]
> ```
>
> 



主进程接收一下信号：

- `TERM`, `INT`: 立刻退出
- `QUIT`: 等待工作进程结束后再退出
- `KILL`: 强制终止进程
- `HUP`: 重新加载配置文件，使用新的配置启动工作进程，并逐步关闭旧进程。
- `USR1`: 重新打开日志文件
- `USR2`: 启动新的主进程，实现热升级
- `WINCH`: 逐步关闭工作进程

工作进程接收以下信号：

- `TERM`, `INT`: 立刻退出
- `QUIT`: 等待请求处理结束后再退出
- `USR1`: 重新打开日志文件

## 负载均衡

### HTTP 负载均衡

1. http 中定义 upstream , upstream 中定义server

```shell
upstream backend {
    server backend1.example.com weight=5;
    server 127.0.0.1:8080       max_fails=3 fail_timeout=30s;
    server unix:/tmp/backend3;

    server backup1.example.com  backup;
}
```

* upstream -> 定义一组servers监听不同的端口，监听TCP和UNIX-domain sockets (http://unix:/tmp/backend.socket:/uri/) 可以混合使用。默认情况下使用轮巡方式进行负载均衡，如上example，根据权重不同，每7个请求会有5个请求分发在backend1.example.com服务器上，另外两个请求分给其他的服务器。



2. http 中定义虚拟服务器 server

```shell
server {
    location / {
        proxy_pass http://backend;
    }
}
```

* proxy_pass -> 设置代理服务器的协议和地址，可以指定`http`和`https`协议，地址可以为域名或者IP地址，端口可选

其他可用相似参数：

* fastcgi_pass -> 设置FastCGI服务器，地址由域名或IP以及端口号组成

* memcached_pass -> 设置缓存服务器，地址由域名或IP以及端口号组成

* scgi_pass -> 设置SCGI 服务器，地址由域名或IP以及端口号组成

* uwsgi_pass -> 设置uwsgi服务器，地址由域名或IP以及端口号组成。协议可以指定`uwsgi`或`suwsgi`

3. 负载均衡方式

* Round Robin（轮询）默认方式，根据`权重`在服务器之间进行均匀分发请求。

  ```shell
  upstream backend {
     # no load balancing method is specified for Round Robin
     server backend1.example.com;
     server backend2.example.com;
  }
  ```

* Least Connections 将请求发送到`最少数量的活动链接`的服务器,同时考虑服务器`权重`。

  ```shell
  upstream backend {
      least_conn;
      server backend1.example.com;
      server backend2.example.com;
  }
  ```

* IP Hash 根据客户端IP地址确定向其发送请求的服务器。 在这种情况下，可以使用`IPv4地址的前三个八位字节`或`整个IPv6地址`来计算哈希值。 该方法保证了`来自相同地址的请求将到达同一服务器`，除非该请求不可用。

  ```shell
  upstream backend {
      ip_hash;
      server backend1.example.com;
      server backend2.example.com;
  }
  ```

  如果其中一台服务器需要暂时从负载平衡循环中删除，则可以使用`down`参数对其进行标记，以保留客户端IP地址的当前哈希值。 该服务器要处理的请求将自动发送到组中的下一个服务器：

  ```shell
  upstream backend {
      server backend1.example.com;
      server backend2.example.com;
      server backend3.example.com down;
  }
  ```

* Generic Hash 根据用户自定义key计算hash值，从而分发服务。key可以是字符串、变量或者两者组合。

  ```shell
  upstream backend {
      hash $request_uri consistent;
      server backend1.example.com;
      server backend2.example.com;
  }
  ```

  使用`consistent`参数启用`Ketama`一致性哈希算法。如果修改缓存中的服务器映射，这种方式只会有少数的密钥->服务器关系重新映射，从而减少缓存丢失。

* Random 请求会随机分发。如果指定`two`这个参数，那么会首先根据权重选则两个服务器，然后根据指定的以下方法进行请求分发。

  - `least_conn` – 活动链接数最少
  - `least_time=header` (NGINX Plus) – 服务器返回响应头平均的最小时间([`$upstream_header_time`](https://nginx.org/en/docs/http/ngx_http_upstream_module.html#var_upstream_header_time))
  - `least_time=last_byte` (NGINX Plus) – 服务器接收完整响应的平均最小时间 ([`$upstream_response_time`](https://nginx.org/en/docs/http/ngx_http_upstream_module.html#var_upstream_response_time))

  ```shell
  upstream backend {
      random two least_time=last_byte;
      server backend1.example.com;
      server backend2.example.com;
      server backend3.example.com;
      server backend4.example.com;
  }
  ```

  Random负载均衡适用于分布式系统，其会将请求在后台的指定服务的一组服务器中随机分发。如果是其他明确后台服务具体服务器的系统，应该使用其他方法，round robin(轮询)、laest connections(最少链接)或 least time(最少时间)。

  > 除了Round Robin，其他负载均衡方法要放在 upstream{}块中其他命令之前。

* Least Time （plus才有）对于每个请求，NGINX Plus选择平均延迟最低和活动连接数量最低的服务器，其中最低平均延迟是根据`least_time`指定的配置参数计算的。`last_time`取值如下：

  - `header` – 服务器接收第一个字节的时间
  - `last_byte` – 服务器接收完整响应时间
  - `last_byte inflight` – 服务器接收完整响应的时间，考虑请求不完整的情况

  ```shell
  upstream backend {
      least_time header;
      server backend1.example.com;
      server backend2.example.com;
  }
  ```

4. 服务器权重 Server Weights

​	默认的轮询分发请求是根据服务器权重来的，服务器默认权重为1。

```shell
upstream backend {
    server backend1.example.com weight=5;
    server backend2.example.com;
    server 192.0.0.1 backup;
}
```

以上例子中三个服务器，第一个权重为5，第二三个未定义权重参数默认为1，第三个服务器`backup`标记为备用服务器，除非前两个服务器都不可用，否则第三个服务器是不会被分发请求的。因此以上例子，如果有6个请求，则会分给第一个服务器5个，第二个服务器1个。

5. 服务器缓慢启动Server Slow-Start

Slow-Start功能可以防止刚恢复的服务器再次被请求冲击，这样会导致请求超时从而使其再次被标记为不可用。

NGINX Plus中开启Slow-Start功能的服务器，在其恢复后会将其权重从0缓慢增到设定值，如下配置：

```shell
upstream backend {
    server backend1.example.com slow_start=30s;
    server backend2.example.com;
    server 192.0.0.1 backup;
}
```

30s是将请求分发从0到恢复最大连接数的时间。

如果只有一台服务器，[`max_fails`](https://nginx.org/en/docs/http/ngx_http_upstream_module.html#max_fails), [`fail_timeout`](https://nginx.org/en/docs/http/ngx_http_upstream_module.html#fail_timeout), and [`slow_start`](https://nginx.org/en/docs/http/ngx_http_upstream_module.html#slow_start)参数设定的值将会被忽略，其不会被认为是不可用的。

6. Session持久化

Session持久化可以用来将同一个用户的请求分发到同一个服务器。

NginxPlus支持三种session持久化方式。使用`sticky `指令指定持久化方式。Nginx使用Session持久化使用`hash`或者`ip_hash`。

* Sticky cookie - Nginx Plus 对首次访问的请求添加cookie，并标记响应其请求的服务器。接下来的请求会匹配cookie从而分发请求到cookie对应标记的服务器。

  ```shell
  upstream backend {
      server backend1.example.com;
      server backend2.example.com;
      sticky cookie srv_id expires=1h domain=.example.com path=/;
  }
  ```

  srv_id是cookie的名字，expires指定cookie的时效为1小时，domain指定域名，path指定cookie存放的路径，以上是最简单的Session持久化方法。

* Sticky route

* Sticky learn



