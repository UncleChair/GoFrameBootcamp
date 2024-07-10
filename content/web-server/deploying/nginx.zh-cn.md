---
title: Nginx
type: docs
weight: 3
next: web-server/conclusion/
---

[`Nginx`](https://nginx.org/en/) 是一个免费、开源、高性能的 HTTP 服务器和反向代理，你可以用它来提供静态文件服务，配置 SSL 或创建负载均衡服务。

我们之前提到的 `守护进程` 和 `Docker` 模式的应用都监听了一个本地端口，你可以使用 `Nginx` 代理转发部分请求给应用，并自行处理其他静态文件请求。

下面是一份配置文件示例：

```nginx
server {
    listen       80;
    server_name  your-domain;

    access_log   /var/log/api-access.log;
    error_log    /var/log/api-error.log;

    location ^~ /public {
        access_log off;
        expires    1d;
        root       /var/www/public;
        try_files  $uri @backend;
    }

    location / {
        try_files $uri @backend;
    }

    location @backend {
        proxy_pass                 http://127.0.0.1:8000;
        proxy_redirect             off;
        proxy_set_header           Host             $host;
        proxy_set_header           X-Real-IP        $remote_addr;
        proxy_set_header           X-Forwarded-For  $proxy_add_x_forwarded_for;
    }
}
```

记得将 `proxy_pass` 修改为你的应用监听端口。
