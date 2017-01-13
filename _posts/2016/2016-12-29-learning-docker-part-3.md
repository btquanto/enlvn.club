---
layout: post
title: Learning Docker - Part 3 - Using docker-compose
category: Tutorials
tags: ["linux", "docker", "tutorial"]
---

A problem with creating docker containers is that the docker run command can get a little bit complicated, especially when there are even more configurations. Similarly, when your application would need multiple different docker containers in order to run, then running containers individually and linking them together can be a mess to newbies.
Docker-compose solves that problem. With a `docker-compose.yml` file, you define the services that make up your app, and run them with just one single simple command.

# Installation

Follow instructions on [Docker's website](https://www.docker.io/)

# Deploying a wordpress website with docker-compose

1. Create a folder containing your project
    ```
    mkdir wordpress_mysql
    cd wordpress_mysql
    ```
2. Create a `docker-compose.yml` file with the following content
    ```yaml
    version: 2
    services:
        mysql:
            image: mysql:latest
            environment:
                MYSQL_ROOT_PASSWORD: somepassword
        wordpress:
            image: wordpress:latest
            ports:
                - 8080:80
            depends_on:
                - "mysql"
            environment:
                WORDPRESS_DB_PASSWORD: somepassword
    ```
3. Initialize the service with `docker-compose`
    ```
    docker-compose up -d
    ```
    It's basically done. However, since the `mysql` container takes some time to initialize, the wordpress container fails when connecting to the mysql database, you have to restart the container
    ```
    docker-compose down
    docker-compose up -d
    ```
4. Visit your [localhost:8080](https://localhost:8080/). It works!!!
