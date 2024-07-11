---
title: Nginx
type: docs
weight: 3
prev: web-server/deploying/
next: web-server/conclusion/
---

[`Nginx`](https://nginx.org/en/) is a free, open-source, high-performance HTTP server and reverse proxy, you could use it to serve static files, configure SSL or create load balance service.

Both `Daemon` mode and `Docker` mode we mentioned before is listening on a certain local port. You could use `Nginx` to proxy some requests to them, and handle static files by itself.

Here is an example config file:

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

Change the `proxy_pass` to your application listening port if needed.
