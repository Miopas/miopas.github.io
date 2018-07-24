---
layout: post
title: nginx 实现浏览器文件下载服务
categories: [Linux, nginx]
description: some word here
keywords: keyword1, keyword2
---

这里记录如何用 `nginx` 搭建一个简易的 file server，实现在浏览器上进行文件的**下载**操作。

要实现文件**下载**功能非常非常容易，不需要写任何前端的东西，只需要用 `nginx` 本身的配置文件就可以实现。

依然是在配置文件 `conf/nginx.conf` 下的 `http { server {...} }` 的部分，如下配置这样一个 `location`:

```
location /myfiles {
    alias /export/share/test/; 	# 文件存放目录，注意要以 '/' 结尾；
    index index.html;  		    # 如果文件存放目录有 index.html，会跳转到 index.html；
    autoindex on;               # 自动列出目录下的文件；
    autoindex_exact_size off;   # 文件大小按 G、M 的格式显示，而不是 Bytes；
}
```

然后，这就做完了。(!)

运行 `nginx` 之后，在浏览器上打开 `http://***/myfiles/`。（替换 `***` 的部分为你的 Server IP/域名/localhost :)）

如果 `index.html` 存在，会自动跳转到 `index.html` 页面：

![pic02](https://github.com/Miopas/miopas.github.io/raw/master/_posts/nginx_file_server_picture_02.jpg)

如果 `index.html` 不存在，则自动会列出文件目录下的文件。例如，现在可以看到这个目录下的 `test.txt` 文件:

![pic01](https://github.com/Miopas/miopas.github.io/raw/master/_posts/nginx_file_server_picture_01.jpg)


点击文件名即可下载。命令行爱好者也可以用 `wget` 下载，还可以断点续传哟。ヾ(=･ω･=)o


**Reference:**

[ngx_http_autoindex_module](http://nginx.org/en/docs/http/ngx_http_autoindex_module.html#autoindex)


题外话：某年某月某日，同组的大佬得知浏览器可以用 80 端口访问服务器以后就开始搞事，于是我就跟着学习了一些 server 相关的东西。当个社会人就是这点好啊。

