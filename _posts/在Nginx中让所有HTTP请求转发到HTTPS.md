---
title: 在Nginx中让所有HTTP请求转发到HTTPS
date: 2017-10-23 20:50:24
tags:
- nginx
- http
- https
categories:
- 运维
---

# 背景

在启用HTTPS协议的网站上，通常会让用户始终以 `https://` 访问，那怎样来实现这个需求呢？首次想到的是在应用代码中实现，但如果后端应用比较多时添加会比较繁琐，另外应用层应该专注业务实现，所以这并不是最合理的方案。试想如果能够将请求流量阻挡的越早肯定会越合理，所以可以在Nginx中配置转发逻辑。

# 前提

在做这件事情的前提是你已经完成证书安装，通过 `https://` 协议能够正常访问网站。

<!-- more -->

# 步骤


1. 找到Nginx配置文件或网站的Server配置子文件，再添加一个Server用来监听 `80` 端口，如果用户不以 `https://` 协议访问网站，则请求会被 `80` 端口监听到。
    ```nginx
      server {
        listen 80 default_server;
        listen [::]:80 default_server;
        server_name example.com www.example.com;
      }
    ```
1. 添加301状态到响应中，诱使浏览器自动进行跳转。
    ```nginx
      server {
        listen 80 default_server;
        listen [::]:80 default_server;
        server_name example.com www.example.com;
        return 301 https://$server_name$request_uri;
      }
    ```
1. 保存之后重启Nginx服务，如果遇到错误可查询日志确认问题，我在配置是就遇到 `default_server` 被重复定义的问题，此时只要把多余的监听删除即可。
