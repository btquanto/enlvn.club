---
layout: post
title: Learning Docker - Part 4 - Creating a LEMP stack using docker
category: Tutorials
tags: ["linux", "docker", "tutorial"]
comments: true
---

In this tutorial, I am demonstrating how easy it is to create a stack (a development/production environment) of your own using `docker-compose`. I am going to create a LEMP stack (Linux - nginx - MariaDB - PHP 5.6). I'm not eager to write my own image yet because, well, there are already images available for nginx, MariaDB, and PHP 5.6. What I want is to link them together to create my own service.

1. Go to [docker hub](https://hub.docker.com/) and search for [nginx](https://hub.docker.com/_/nginx/), [mariadb](https://hub.docker.com/_/mariadb/), and [php](https://hub.docker.com/_/php/), and choose the images that I want to use to compose my stack
    * nginx:1.11.8-alpine
    * mariadb:5.5
    * php:5.6-fpm-alpine

    The reason I choose alpine image over other images is that alpine images are usually much smaller.

2. Create a project folder for the LEMP stack

    ```
    mkdir LEMP
    cd LEMP
    ```

3. Create a `www` folder to contain PHP source code, and make a `phpinfo()` test script

    ```
    mkdir www
    echo "<?php phpinfo(); ?>" > ./www/index.php
    ```

4. Create a folder for containing log files

    ```
    mkdir logs
    ```

5. Create a `./nginx/site.conf` file containing the server's configuration

    ``` nginx
    server {
        listen 80;

        root /www;

        index index.html index.htm index.php;

        location / {
            try_files $uri $uri/ /index.php?$args;
        }

        location ~ \.php$ {
            try_files $uri =404;
            include fastcgi_params;
            fastcgi_pass phpfpm:9000;
            fastcgi_index index.php;
            fastcgi_param SCRIPT_FILENAME $document_root$fastcgi_script_name;
        }
    }
    ```

    This is pretty much a standard configuration for an nginx-phpfpm site. Except this line:

    ```
    fastcgi_pass phpfpm:9000;
    ```

    We are going to link the nginx container with the phpfpm container, thus `phpfpm` will be available in the nginx container's `hosts`

6. Create a `docker-compose.yml` file with the following content

    ``` yaml
    version: '2'
    services:
    mariadb:
        image: mariadb:5.5
        environment:
        - MYSQL_ROOT_PASSWORD=password123
        - MYSQL_DATABASE=yii
        - MYSQL_USER=yii
        - MYSQL_PASSWORD=abcd1234
    phpfpm:
        image: php:5.6-fpm-alpine
        volumes:
        - ./www:/www
        links:
        - mariadb
    nginx:
        image: nginx:1.11.8-alpine
        volumes:
        - ./nginx:/etc/nginx/conf.d
        - ./logs:/var/nginx/logs
        - ./www:/www
        ports:
        - 80:80
        links:
        - phpfpm
        command: /bin/sh -c "nginx -g 'daemon off;'"
    ```

    Things here are pretty straightforward:
    * We have 3 services in our stack: `mariadb`, `nginx`, and `phpfpm`
    * What `environment` does is obvious
    * `./nginx` will be mounted to nginx's sites folder
    * `./logs` will be mounted to nginx's logs folder
    * `./www` will be mounted on both `nginx` and `phpfpm` containers, so the php process will also have access to the same file paths as nginx
    * `mariadb` is linked to `phpfpm`, and `phpfpm` is linked to nginx. There's no need for nginx to link with mariadb, since static files don't make database connections

7. Check your folder's structure

    ```
    $ tree ./
    ./
    ├── docker-compose.yml
    ├── logs
    ├── nginx
    │   └── site.conf
    └── www
        └── index.php
    ```

8. Run the stack

    ```
    docker-compose up -d
    ```

9. Go to [localhost](http://localhost/) and check if your `phpinfo()` script has been successfully executed
10. Testing database connection by replacing the content of `./www/index.php` with the following

    ``` php
    <?php
    $servername = "mariadb";
    $username = "yii";
    $password = "abcd1234";

    // Create connection
    $conn = new mysqli($servername, $username, $password);

    // Check connection
    if ($conn->connect_error) {
        die("Connection failed: " . $conn->connect_error);
    }
    echo "Connected successfully";
    ?>
    ```

    There should be an error message that says something like this

    ```
    Fatal error: Class 'mysqli' not found in /www/index.php on line 7
    ```

    We'll just have to install `mysqli`. Luckily, it isn't too complicated. The instructions are already on the [php docker hub repo](https://hub.docker.com/_/php/)

    ```
    docker exec lemp_phpfpm_1 docker-php-ext-install mysqli
    ```

    Restart the service

    ```
    docker-compose restart
    ```

11. Now, again, connect to [localhost](http://localhost/). It should say `Connected successfully`
