---
title: nginx学习
tag: 服务器
date: 2022-1-21
---

主要配置 conf
更改配置以后需要重启
> docker restart nginx
```js
server {
  // 根据你的需求改变此端口
  listen 80;  //也可以是1.2.3.4:80的形式

  // 多个主机名可以用空格隔开，当然这个信息也是需要按照你的需求而改变的。
  server_name  star.yourdomain.com *.yourdomain.com www.*.yourdomain.com;

  //或者可以使用：_ * (具体内容参见本维基其他页面)
  root /PATH/TO/WEBROOT/$host;
  alias 
  error_page  404     // http://yourdomain.com/errors/404.html;
  access_log  logs/star.yourdomain.com.access.log;

  location / {
    root   /PATH/TO/WEBROOT/$host/;  // 项目的存放地址
    index  index.html; // 打开的根目录
  }

  // 直接支持静态文件 (从配置上看来不是直接支持啊)
  location ~* ^.+.(jpg|jpeg|gif|css|png|js|ico|html)$ {
    access_log        off;
    expires           30d;
  }
  location ~ .php$ {
    // 如果需要，你可以为不同的FCGI进程设置不同的服务信息
    fastcgi_pass   127.0.0.1:YOURFCGIPORTHERE;
    fastcgi_index  index.php;
    fastcgi_param  SCRIPT_FILENAME  /PATH/TO/WEBROOT/$host/$fastcgi_script_name;
    fastcgi_param  QUERY_STRING     $query_string;
    fastcgi_param  REQUEST_METHOD   $request_method;
    fastcgi_param  CONTENT_TYPE     $content_type;
    fastcgi_param  CONTENT_LENGTH   $content_length;
    fastcgi_intercept_errors on;
  }
  location ~ /\.ht {
    deny  all;
  }
 }
```
