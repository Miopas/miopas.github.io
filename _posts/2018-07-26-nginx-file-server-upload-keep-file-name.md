---
layout: post
title: nginx-upload-module 上传文件如何保持原文件名
categories: [Linux, nginx]
description: some word here
keywords: keyword1, keyword2
---

用 `nginx-upload-module` 模块实现往服务器上传文件的时候，文件名会变成一串数字。这是 `nginx-upload-module` 模块为了避免文件冲突，做了一个 `file hashing`。

## 1. 关于 `file hashing` 

先说说这个 `file hashing` 是啥。

在配置 `location` 的时候有这样一个行：
```
    upload_store /export/tmp/upload 1;
```
这里的 `1` 就是为了 `file hashing` 配置的。

在官方文件里可以看到：
```
upload_store

Syntax:	upload_store <directory> [<level 1> [<level 2> ] ... ]
Default:	none
Context:	server,location

Specifies a directory to which output files will be saved to. The directory could be hashed. In this case all subdirectories should exist before starting NGINX.
```
也就是说上传后的文件，不是储存在 `<directory>` 目录下，而是会根据文件的哈希值，储存在 `<directory>` 的一个子目录下。

例如这个例子中，文件名变成了 `0000000006`，储存路径变成了 `<directory>/6/`：
![pic01](https://github.com/Miopas/miopas.github.io/raw/master/assets/images/posts/2018-07-26-nginx-file-server-upload-keep-file-name-pic01.png)

为什么要做 `file hashing` 呢？主要是为了避免冲突。

如果没有 `file hashing`，当两个人**同时**上传一个**同名**文件的时候，一个人的文件会覆盖另一个人的文件。`file hashing` 的机制让两个文件存在不同的路径下，不存在覆盖的问题。

在 [Git Issue](https://github.com/fdintino/nginx-upload-module/issues/69) 中提到：

> The uploaded file names are designed to avoid collisions. What would happen if two people uploaded files with the same name at the same time? One person's file would overwrite the other person's.
>
> You should rename the files at the "backend" layer, not in the module source code.


## 2. 恢复原文件名

那么，如果我希望保持原文件名应该怎么做呢？

我的做法是在 `php` 脚本里加上一步 `mv` 文件的操作，用 `rename` 函数实现：
```php
if ($_POST){
    echo "<h2>Uploaded files:</h2>";
    echo "<table border=\"2\" cellpadding=\"2\">";

    echo "<tr><td>Name</td><td>Location</td><td>Content type</td><td>MD5</td><td>Size</td><td>Scp Command</td><td>Wget Command</tr>";

    for ($i=1;$i<=$slots;$i++){
        $key = $header_prefix.$i;
        if (array_key_exists($key."_name", $_POST) && array_key_exists($key."_path",$_POST)) {
            $tmp_name = $_POST[$key."_path"];
            $name = $_POST[$key."_name"];
            $content_type = $_POST[$key."_content_type"];
            $md5 = $_POST[$key."_md5"];
            $size = $_POST[$key."_size"];

            $final_path = "/export/share/upload";
            if (rename($tmp_name, "$final_path/$name")) {
		    echo "SUCCESS!";
	    } else {
		    echo "FAIL!";
	    }

	    $scp_cmd = "scp team@***:/export/share/upload/$name .";
	    $wget_cmd = "wget http://***/files/upload/$name";
	    echo "<tr><td>$name</td><td>$final_path</td><td>$content_type</td><td>$md5</td><td>$size</td><td>$scp_cmd</td><td>$wget_cmd</td>";
        }
    }

    echo "</table>";
}
```

测试结果：
![pic02](https://user-images.githubusercontent.com/10841135/42874011-a5de0554-8ab2-11e8-8b24-78534d95ab76.png)


这就搞定啦。完整的 `php` 脚本在这里[可以](https://gist.github.com/Miopas/09df8dbf3b85ec61069f9a12d729bb27)得到。


## Reference

[Git Issue: Change names of temporary files to original](https://github.com/fdintino/nginx-upload-module/issues/69)

[What is the use of Hash in Nginx Upload module?](https://stackoverflow.com/questions/14118723/what-is-the-use-of-hash-in-nginx-upload-module)

[Move a file into a different folder on the server](https://stackoverflow.com/questions/19139434/php-move-a-file-into-a-different-folder-on-the-server)

[nginx-upload-module wiki](https://www.nginx.com/resources/wiki/modules/upload/)
