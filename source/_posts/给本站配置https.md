---
title: 给本站配置一级域名以及https
date: 2018-01-19 20:38:07
tags: nginx
categories: blog
description: 想给这个站点配置自己的域名,本来是https的,但是配置好了后发现那把绿色的小锁不见了,刚好手头也有个阿里的服务器,就想着配置个nginx代理,使用https访问本站
---
##### 配置一级域名
有关域名的配置部分挺简单的,登录阿里云,将域名以前的解析配置都删除掉,新增两个A记录,主机记录一个配置"@"直接解析主域名,一个使用"www",解析www,记录值就填写服务器的ip地址.基本上不用等待,直接就能使用了.

##### 配置https
配置好了以后,再配置https,这个需要申请一个https的证书,阿里云有免费的证书可以申请,申请好后配置到nginx里面,就可以了.

nginx主要配置内容记录如下:
```conf
server {
    listen 443 ssl;
    listen 80;
    server_name yaoboqi.cn;
    ssl on;
    root html;
    index index.html index.htm;

    ssl_certificate  ../cert/214354456760134.pem;
    ssl_certificate_key ../cert/214354456760134.key;
    ssl_session_timeout 5m;
    ssl_ciphers ECDHE-RSA-AES128-GCM-SHA256:ECDHE:ECDH:AES:HIGH:!NULL:!aNULL:!MD5:!ADH:!RC4;
    ssl_protocols TLSv1 TLSv1.1 TLSv1.2;
    ssl_prefer_server_ciphers on;
    
    error_page 497 https://yaoboqi.cn;    

    location / {
	proxy_pass https://zonzie.github.io;

        root html;
        index index.html index.htm;

        proxy_set_header Host $host;
        proxy_set_header X-Real-IP $remote_addr;
        proxy_set_header REMOTE-HOST $remote_addr;
        proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
    }
}
```

当网站只允许https访问时,nginx会报出497错误码,这里利用nginx的497状态码将默认的http访问后出现的错误页面重定向到https://yaoboqi.cn这个域名上.
网站的流量统计用了leancloud,需要将新的域名添加到安全域名列表里,不然访问量是看不到的,对leancloud的请求会报出401未授权的错误.