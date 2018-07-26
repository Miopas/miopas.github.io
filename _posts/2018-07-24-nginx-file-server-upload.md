---
layout: post
title: nginx 实现浏览器文件上传服务
categories: [Linux, nginx]
description: some word here
keywords: keyword1, keyword2
---

既然能利用 `nginx` 实现在浏览器上进行文件下载，那么应该也能在浏览器上进行文件的**上传**操作。基于这样的想法，研究了一下如何用 `nginx` 搭建一个文件上传服务。

`nginx` 实现文件上传不止一种方式，这里我是利用了 `nginx-upload-module` 这个第三方模块来实现的。

## 1. 安装

#### 1.1, 引入 `nginx-upload-module` 模块
引入 `nginx-upload-module` 模块需要重新编译 `nginx`。首先从 `github` 获取源码:
```shell
git clone -b 2.2 https://github.com/vkholodkov/nginx-upload-module
```
**注意**，`clone` 的是 `2.2` 分支版本。

然后，进入 `nginx` 源码目录，在运行 `./configure` 加上 `--with-ld-opt` 和 `--add-module` 参数：
```shell
cd /usr/local/nginx-1.9.9/

./configure --prefix=/usr/local/nginx \
	    --with-ld-opt='-lssl -lcrypto' --add-module=/usr/local/nginx-1.9.9/nginx-upload-module
```
其中的 `/usr/local/nginx-1.9.9/nginx-upload-module` 路径替换为你克隆的 `nginx-upload-module` 仓库的路径。

#### 1.2, 安装 php-fpm

因为我是用 `php` 脚本实现了这个功能，因此还需要启动一个监听 `php` 脚本的服务。这里用到了 `php-fpm`。

参考教程：[nginx php-fpm安装配置](https://wizardforcel.gitbooks.io/nginx-doc/content/Text/6.5_nginx_php_fpm.html)

> nginx本身不能处理PHP，它只是个web服务器，当接收到请求后，如果是php请求，则发给php解释器处理，并把结果返回给客户端。

这里备份一下安装脚本：
```shell
yum -y install gcc automake autoconf libtool make

yum -y install gcc gcc-c++ glibc

yum -y install libmcrypt-devel mhash-devel libxslt-devel \
	libjpeg libjpeg-devel libpng libpng-devel freetype freetype-devel libxml2 libxml2-devel \
	zlib zlib-devel glibc glibc-devel glib2 glib2-devel bzip2 bzip2-devel \
	ncurses ncurses-devel curl curl-devel e2fsprogs e2fsprogs-devel \
	krb5 krb5-devel libidn libidn-devel openssl openssl-devel


wget http://cn2.php.net/distributions/php-5.4.7.tar.gz
tar zvxf php-5.4.7.tar.gz
cd php-5.4.7
./configure --prefix=/usr/local/php  --enable-fpm --with-mcrypt \
--enable-mbstring --disable-pdo --with-curl --disable-debug  --disable-rpath \
--enable-inline-optimization --with-bz2  --with-zlib --enable-sockets \
--enable-sysvsem --enable-sysvshm --enable-pcntl --enable-mbregex \
--with-mhash --enable-zip --with-pcre-regex --with-mysql --with-mysqli \
--with-gd --with-jpeg-dir

make all install

cd /usr/local/php
cp etc/php-fpm.conf.default etc/php-fpm.conf
vi etc/php-fpm.conf

```

编译完成后，进入 `/usr/local/php` 目录，运行 `./sbin/php-fpm` 启动服务。

## 2. 配置 `location`

在 `nginx` 的安装目录下找到 `conf/nginx.conf`，依然是在 `http {... server {...}}` 的地方配置 `location`：

```js
# upload
client_max_body_size 100g; # 这个配置表示最大上传大小，但是我没有验证过是否能传 100g 的文件

# Upload form should be submitted to this location
location /upload {
        # Pass altered request body to this location
        upload_pass /upload.php;

        # 开启resumable
        upload_resumable on;

        # Store files to this directory
        # The directory is hashed, subdirectories 0 1 2 3 4 5 6 7 8 9 should exist
        # 记得修改目录的读写权限
        upload_store /export/tmp/upload 1;
        upload_state_store /export/tmp/upload/state;

	# Allow uploaded files to be read by all
        upload_store_access all:r;

        # Set specified fields in request body
        upload_set_form_field "${upload_field_name}_name" $upload_file_name;
        upload_set_form_field "${upload_field_name}_content_type" $upload_content_type;
        upload_set_form_field "${upload_field_name}_path" $upload_tmp_path;

        # Inform backend about hash and size of a file
        upload_aggregate_form_field "${upload_field_name}_md5" $upload_file_md5;
        upload_aggregate_form_field "${upload_field_name}_size" $upload_file_size;

        upload_pass_form_field "^submit$|^description$";
}

location ~ \.php$ {
   # fastcgi_pass   unix:/run/php-fpm/php-fpm.sock;
   fastcgi_pass   127.0.0.1:9000;
   fastcgi_index  index.php;
   # fastcgi_param  SCRIPT_FILENAME  /scripts$fastcgi_script_name;
   fastcgi_param   SCRIPT_FILENAME    $document_root$fastcgi_script_name;

   include        fastcgi_params;
}

```

这个配置参考了博客 [NGINX as a file server](https://www.yanxurui.cc/posts/server/2017-03-21-NGINX-as-a-file-server/)，其中我仅修改了 `client_max_body_size`、`upload_pass`、`upload_store`、`upload_state_store` 和 `upload_store_access` 这几个参数，以适应我的任务。


## 3. 测试

在浏览器中输入 `http://***/upload.php`，可以看到这个界面：
![pic-01](https://github.com/Miopas/miopas.github.io/raw/master/_posts/2018-07-24-nginx-file-server-upload-pic01.png)

点击 `upload` 按钮后，跳转到：
![pic-02](https://github.com/Miopas/miopas.github.io/raw/master/_posts/2018-07-24-nginx-file-server-upload-pic02.png)


上传成功。

## 4. Tips

* `php-fpm` 的 git 仓库的说明文档里的方式安装方式，是打补丁的方式。我按照这个安装步骤各种出错，建议别用；
* 修改了 `location /upload {...}` 里面的配置以后，不仅要重启 `nginx` 服务，还需要重启 `php-fpm` 服务，新的配置才能生效；
* 在浏览器中输入的 url 是 `http://***/upload.php` 而不是 `http://***/upload`；
* `nginx-upload-module` 模块为了避免文件冲突，会做一个 `file hashing`，最后上传的文件的存放路径不是 `upload_store` 配置的目录，而是它的一个子目录；而且文件名会变成一串数字。如何解决，请看[nginx-upload-module 上传文件如何保持原文件名](https://miopas.github.io/2018/07/26/nginx-file-server-upload-keep-file-name/)


**Reference:**

[Nginx upload module manual](http://www.grid.net.ru/nginx/upload.en.html)

[nginx-upload-module github](https://github.com/fdintino/nginx-upload-module/tree/2.2)

[NGINX as a file server](https://www.yanxurui.cc/posts/server/2017-03-21-NGINX-as-a-file-server/)

[nginx php-fpm安装配置](https://wizardforcel.gitbooks.io/nginx-doc/content/Text/6.5_nginx_php_fpm.html)
