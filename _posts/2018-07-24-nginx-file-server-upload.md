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
TODO

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
![pic-01]()


## 4. Tips

* `php-fpm` 的 git 仓库的说明文档里的方式安装方式，是打补丁的方式。我按照这个安装步骤各种出错，建议别用；
* 修改了 `location /upload {...}` 里面的配置以后，不仅要重启 `nginx` 服务，还需要重启 `php-fpm` 服务，新的配置才能生效；
* 在浏览器中输入的 url 是 `http://***/upload.php` 而不是 `http://***/upload`；
* `nginx-upload-module` 模块为了避免文件冲突，会做一个 `file hashing`，最后上传的文件的存放路径不是 `upload_store` 配置的目录，而是它的一个子目录；而且文件名会变成一串数字。~~如何解决，请看[nginx-upload-module 上传文件如何保持原文件名](还没写~~


**Reference:**

[Nginx upload module manual](http://www.grid.net.ru/nginx/upload.en.html)

[nginx-upload-module github](https://github.com/fdintino/nginx-upload-module/tree/2.2)

[NGINX as a file server](https://www.yanxurui.cc/posts/server/2017-03-21-NGINX-as-a-file-server/)

