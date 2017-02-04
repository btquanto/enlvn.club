---
title: Learning Docker - Part 2 - Docker cheatsheet
category: Tutorials
tags: ["linux", "docker", "tutorial"]
---

# Config to run docker without sudo

By default, docker needs to be run with root privilege, but we don't want that for various reasons. Follow these steps to allow running docker without sudo:

1. Add a group named docker:

    ```
    sudo groupadd docker
    ```

2. Add the user you want to run docker without sudo to group docker

    ```
    sudo usermod -aG docker ${USER}
    ```

3. Restart docker daemon:

    ```
    sudo service docker restart
    ```

4. Relogin, or running the following command:

    ```
    newgrp docker
    ```

**Note:** _This allows running docker without using the `sudo` command. However, files created by docker by default are still owned by root. To truly avoid giving root privilege to docker, specify option `-u <user id>` when running `docker run`. For example:_

```
docker run -u `id -u $USER` <image>
```

_Or_

```
docker run -u $UID <image>
```

_If you are using `docker-compose`, then you should specify the option `user` in your `docker-compose.yml` file. For example:_

``` yml
version: '3'
services:
  jekyll:
    user: $UID
    image: btquanto/docker-jekyll
    ports:
      - 4000:4000
    volumes:
      - ./:/src
    command: jekyll serve -H 0.0.0.0 --draft
```

# Basic docker commands

1. List all images:

    ```
    docker images
    ```

2. List all running containers:

    ```
    docker ps
    ```

    List all containers:

    ```
    docker ps -a
    ```

3. Create and run a new container:

    ```
    docker run <image name>
    ```

    Create a new container, and give the container a name

    ```
    docker run --name <container name> <image name>
    ```

    Create a new container under interactive mode, and run bash

    ```
    docker run -ti <image name> /bin/bash
    ```

    Create a new container in detached mode

    ```
    docker run -d <image name>
    ```

    Run docker container under a different user (for security)

    ```
    docker run -u `id -u <username>` <image name>
    ```

    Run docker container under current user

    ```
    docker run -u $UID <image name>
    ```

    Well, you can combine these together, for example:

    ```
    docker run -u $UID -dti --name ubuntu1604 ubuntu:16.04 /bin/bash
    ```

4. Attach to a container:

    ```
    docker attach <container id/ name>
    ```

5. Detach from a container:

    ```
    <Ctrl> + P,Q
    ```

6. Start a container:

    ```
    docker start <container id/ name>
    ```

    Start and attach to a container

    ```
    docker start -a <container id/ name>
    ```

7. Stop a container:

    ```
    docker stop <container id/ name>
    ```

8. Remove a container:

    ```
    docker rm <container id/ name>
    ```

9. Remove an image:

    ```
    docker rmi <image name>
    ```

10. Create an image from a container:

    ```
    docker commit <container id/ name> <new image name>
    ```
