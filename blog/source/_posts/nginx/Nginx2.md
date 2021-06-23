---
title: Nginx--HTTP负载均衡
index_img: /img/Nginx.assets/nginx.png
tags: [Nginx]
categories: [web服务器]
date: 2020-08-29 17:30:00
---

## HTTP负载均衡

### http 中定义 upstream , upstream 中定义server

```shell
upstream backend {
    server backend1.example.com weight=5;
    server 127.0.0.1:8080       max_fails=3 fail_timeout=30s;
    server unix:/tmp/backend3;

    server backup1.example.com  backup;
}
```

* upstream -> 定义一组servers监听不同的端口，监听TCP和UNIX-domain sockets (http://unix:/tmp/backend.socket:/uri/) 可以混合使用。默认情况下使用轮巡方式进行负载均衡，如上example，根据权重不同，每7个请求会有5个请求分发在backend1.example.com服务器上，另外两个请求分给其他的服务器。



### http 中定义虚拟服务器 server

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

### 负载均衡方式

* Round Robin（轮询）默认方式，根据`权重`在服务器之间均匀分发请求。

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

### 服务器权重 Server Weights

​	默认的轮询分发请求是根据服务器权重来的，服务器默认权重为1。

```shell
upstream backend {
    server backend1.example.com weight=5;
    server backend2.example.com;
    server 192.0.0.1 backup;
}
```

以上例子中三个服务器，第一个权重为5，第二三个未定义权重参数默认为1，第三个服务器`backup`标记为备用服务器，除非前两个服务器都不可用，否则第三个服务器是不会被分发请求的。因此以上例子，如果有6个请求，则会分给第一个服务器5个，第二个服务器1个。

### 服务器缓慢启动Server Slow-Start

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

### Session持久化

Session持久化可以用来将同一个用户的请求分发到同一个服务器。

NginxPlus支持三种session持久化方式。使用`sticky `指令指定持久化方式。Nginx的Session持久化方式，通过`hash`或者`ip_hash`来实现。

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
  Nginx Plus 在第一次接收到客户端的请求的时候会为其指定一个路由，后续的请求会与服务器的路由参数比较，从而分发到正确的代理服务器，路由信息是从`cookie`或者`uri`中获取的。

  ```shell
  upstream backend {
      server backend1.example.com route=a;
      server backend2.example.com route=b;
      sticky route $route_cookie $route_uri;
  }
  ```

  

* Sticky learn

  ```shell
  upstream backend {
     server backend1.example.com;
     server backend2.example.com;
     sticky learn
         create=$upstream_cookie_examplecookie
         lookup=$cookie_examplecookie
         zone=client_sessions:1m
         timeout=1h;
  }
  ```

  `create`参数表示如何创建session，示例为根据传来的名为examplecookie的cookie来创建。

  `lookup`参数表示如何查找session，示例为在名为examplecookie中查找session。

  `zone`参数表示session存放的区域名称和大小，示例为区域名为client_session大小为1m。

  `timeout`为session过期时间。

  ​		与前两种方式相比较，这种持久化方式要更加复杂一些，因为当前持久化方式不需要客户端存放cookie信息，cookie信息是存放在服务器的共享区域内。

  在集群中session存放的位置是可以共享的，但是要满足以下条件：

  * 共享存储区域的名字得相同，也就是`zone`后的参数得一致。
  * 开启[zone_sync](https://nginx.org/en/docs/stream/ngx_stream_zone_sync_module.html?&amp;_ga=2.266232719.1684610560.1600414511-39955037.1597994482#zone_sync)共享区域同步功能。
  * 指定`sync`参数。

### 限制连接数量
使用`max_conns`来指定最大连接数，如果满了，进入队列进行处理。

```shell
upstream backend {
    server backend1.example.com max_conns=3;
    server backend2.example.com;
    queue 100 timeout=70;
}
```

如果队列满了，或者是超时未获得连接，则客户端是会收到错误信息的。

> 如果其他的工作进程中有打开的[keepalive](https://nginx.org/en/docs/http/ngx_http_upstream_module.html?&_ga=2.203866896.1684610560.1600414511-39955037.1597994482#keepalive)连接，那么max_conns限制会被忽略。因此，多个工作进程共享内存，max_conns是有可能超过指定数量的。



1. 多个工作进程间数据共享

   * 如果负载均衡`upstream`没有指定`zone`，每个工作进程从服务器组拷贝的`配置信息`以及`相关计数`是相互独立的。配置信息将无法动态进行修改。

   * 指定`zone`参数则服务器组的配置信息将不再是每个工作进程独自一份，配置信息会放在共享内存中，这样就可以动态修改配置信息，并同步相关计数。

   * `健康检查`以及`动态配置`功能的开启是需要指定`zone`参数的。配置`zone`参数是比较有用的，生产上不可能一个工作进程去处理请求的。如果当前工作进程挂了，其他工作进程是无法全部立刻知道你挂了（确定挂掉是要在fail_timeout时间内，失败重试的次数必须等于max_fails* 辅助工作进程数量），部分工作线程还是会给你发请求的。

     不指定`zone`最小连接数（least connections）这种负载均衡方法在低负载下是没法用的。因为计数是独立的，工作进程拿到的是自己保存的计数，一个进程分发完请求给最小活跃连接数的服务器后，服务器活跃连接数变了只有发请求的工作进程知道，其他工作进程是不知道的，可能根据旧的信息，再次给这个服务器分发请求。

   * 没有官方推荐的配置，因为大家实际情况不一样请自行设置。

     >session持久化方式未sticky_route，开启单个健康检查的情况下，256k的内存可以存放的服务器指标：
     >
     >128 server 每个服务器定义为 IP地址:端口
     >
     >88 server 每个服务器定义为 主机名:端口，主机名解析为单个IP地址
     >
     >12 server 每个服务器定义为 主机名:端口，主机名解析为多个IP地址

2. 使用DNS修改HTTP负载均衡的配置
   服务器组的配置信息可以在运行时使用DNS来修改。根据DNS定时解析域名，自动同步，不需要重启。

   ```shell
   http {
       resolver 10.0.0.1 valid=300s ipv6=off;
       resolver_timeout 10s;
       server {
           location / {
               proxy_pass http://backend;
           }
       }
       upstream backend {
           zone backend 32k;
           least_conn;
           # ...
           server backend1.example.com resolve;
           server backend2.example.com resolve;
       }
   }
   ```

   `resolver` 指定DNS，valid配置解析IP时间间隔。ipv6=off 只解析ipv4，默认两种都解析。

   `resolve` 指定哪个域名要被解析。
   如果一个域名被解析为多个IP列表，则会在这些IP列表之间进行负载均衡。并且IP列表改变后，会立即在新的IP列表间进行负载均衡。