---
layout: post
title: Implement a self-signed SSL certificate to nginx
is_recommended: true
category: System Admin
tags: ["linux", "nginx", "ssl", "tutorial"]
comments: true
---

1. Generate `myapp.key`, and `myapp.crt`. The location is not really important, but make sure that the user that runs nginx has access to it.

    ```
sudo openssl req -x509 -nodes -days 365 -newkey rsa:2048 -keyout /path/to/myapp.key -out /path/to/myapp.crt
    ```

2. Then, add ssl configuration to your nginx site config

    ``` nginx
listen 443 ssl;
ssl_certificate       /path/to/myapp.crt;
ssl_certificate_key   /path/to/myapp.key;
    ```

    For example:

    ``` nginx
server {
    listen 80;
    listen 443 ssl;

    # Edit these as fit
    server_name           domain.dot.com;
    ssl_certificate       /path/to/myapp.crt;
    ssl_certificate_key   /path/to/myapp.key;

    root                  /path/to/my/app/root/foler;
    access_log            /path/to/log/folder/access.log;
    error_log             /path/to/log/folder/logs/error.log;

    location / {
        try_files         $uri @flask;
    }

    # My app is a flask app, served at localhost:8000
    location @flask {
        proxy_set_header   Host $http_host;
        proxy_set_header   X-Real-IP $remote_addr;
        proxy_set_header   X-Forwarded-For $proxy_add_x_forwarded_for;

        proxy_redirect     off;
        proxy_pass         http://127.0.0.1:8000;
    }
}
    ```
