---
layout: post
title: nginx 实现浏览器文件传输服务
categories: [Linux, nginx]
description: some word here
keywords: keyword1, keyword2
---


某年某月某日，同组的大佬得知浏览器可以用 80 端口访问服务器以后就开始搞事，于是我就跟着学习了一些 server 相关的东西。~~当个社会人就是这点好啊~~

这里记录一下如何用 nginx 搭建一个简易的 file server，实现在浏览器上进行文件的上传和下载操作。

ngnix 的安装过程就略过了，有需要可以看这里：[nginx 实现端口转发](https://miopas.github.io/2018/07/21/nginx-port80-proxy/)。

## 1. 下载文件 

要实现文件**下载**功能非常非常容易，不需要写任何前端的东西，只需要配置这样一个 `location`:
```js
        location /myfiles {
            alias /export/share/test/; 		# 文件存放目录，注意要以 '/' 结尾
            index index.html;  			# 
            autoindex on;                       # 自动列出目录下的文件
            autoindex_exact_size off;           # 文件大小按 G、M 的格式显示，而不是 Bytes
        }
```

这里是用到了 nginx 的 `ngx_http_autoindex_module` 模块，具体文档参见[这里](http://nginx.org/en/docs/http/ngx_http_autoindex_module.html#autoindex)。

在浏览器中打开（这里 `index.html` 不存在）


## 2. 上传文件 
