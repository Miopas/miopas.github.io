---
layout: post
title: nginx 实现端口转发
categories: [Linux, nginx]
description: some word here
keywords: keyword1, keyword2
---

公司内网的服务器只有 80 端口是可以外部访问，最近于遇到需要部署多个服务给外部访问，可以通过 `nginx` 做个端口转发来实现。

系统：Centos 7.x

## 1. 安装 nginx

参考教程：[Centos下 Nginx安装与配置](https://www.jianshu.com/p/d5114a2a2052)

按着教程走基本没问题，记得改版本号。

其中 `zlib` 之类的库可以用 `yum` 安装。`nginx` 需要从源码编译安装。

Ps. 可以用 `ldconfig -p | grep xxx` 来检查对应的库是否已安装。

## 2. 启动服务

两种方式：
1) 在 nginx 目录下运行：./sbin/nginx

用这种方式启动的话，需要用同样的方式来停止服务:
```shell
./sbin/nginx -s stop
```

2) 运行 `service nginx start` （但是需要先设置这个启动的 nginx 的路径）

同理，停止服务需要运行:
```shell
service nginx start
```


## 3. 配置端口转发 

修改在安装目录下的 `conf/nginx.conf` 的配置，找到 `http {…}` 的部分，在里面修改：

```json
        location /app1 {
                proxy_redirect off;
                proxy_pass http://localhost:7474/app1/;
        }

        location /app2 {
                proxy_redirect off;
                proxy_pass http://localhost:5001/app2/;
        }
```

**注意：**这个配置是从上到下顺序生效的，所以 `location / {..}` 的配置要放在最后，不然 `location /app1 {…}` 的配置不能正常工作。


Reference:

[nginx 配置从零开始](http://oilbeater.com/nginx/2014/12/28/nginx-conf-from-zero.html)

[【nginx配置】nginx做非80端口转发](http://www.hoohack.me/2015/12/10/nginx-non80-port-forward)



## 4. Rewrite url 

遇到一个情况，如果某个服务在解析 url 路径的时候从根路径开始解析，按照上面的方式，只能这么配置：
```json
        location / {
                proxy_redirect off;
                proxy_pass http://localhost:5001/;
        }
```

这样做，如果有另一个服务也是按照这种方式解析，就无法配置了。有另一种方式，用 `rewrite` 来重写 url，配置如下：

```json
        location /app3 {
                rewrite /app3/(.*)  /$1  break;
                proxy_set_header Host $host;
                proxy_redirect off;
                proxy_pass http://localhost:5001/;
        }
```

这样就能满足需求啦。

Reference:

[Nginx reverse proxy + URL rewrite](https://serverfault.com/questions/379675/nginx-reverse-proxy-url-rewrite)

