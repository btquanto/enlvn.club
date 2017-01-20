---
layout: post
title: Learning Docker - Part 1 - Getting started
is_recommended: true
category: Tutorials
tags: ["linux", "docker", "tutorial"]
comments: true
---

# The problems I had

I have been writing a bunch of web application skeletons for later usage, and have been using *vagrant + chef* for automatic local environment deployment.

At first, things were cool. Everytime I needed to deploy a separate environment for development, I just needed to clone the project, the cookbooks submodules, then ran `vagrant up`. It took a while to get the virtual machine up and running, and obviously running multiple VMs was not a really healthy choice for my RAM, but it did its job.

Then, a few months later, and I started another project, and the some chef recipes failed, and I had to fix the recipes to get everything up and running again. Some cookbooks were open sourced, and had not been updated for quite some time, so I had to fork the cookbook repo to my github in order to save the changes.

Then, a few weeks later, I trained an undergraduate junior on working one of the frameworks that I have written a skeleton for. I didn't want to stress much into system administration stuff, and wanted to demonstrate a good example of separated development environment, so I just gave her the skeleton repo. Again, it needed some fixes in order to work.

And so on. The more I work with *vagrant + chef*, the more I realize that the problem in using chef recipes is that it will at some point fail, and fixing recipes is not always straight forward.

Thus, I have been looking for an alternative

# Why Docker

Docker has been around for a while, but I have just truly started learning it recently, and it exceeds my expectation.

There are many videos and blogs introducing what Docker is, what is the differences between containerization and virtualization, so I won't be repeating them. I'll just list the reasons why I find Docker fascinating.

1. The problem with recipes is (almost) gone. Although docker images are usually built with a `Dockerfile`, and building an image from the same `Dockerfile` may still fail at some point in the future, the image itself is sharable and reusable.
2. Thus, when developing an application, I just have to create the isolated development environment once, package it into a docker image, and use the image to create a docker container that has everything ready.
3. The same image can be reused for different application skeletons. Well, the same can be said for *vagrant + chef*. However, it can be done in a much simpler manner with `docker`.
4. I can add more changes, and create a new image from a base image, extending my application.
5. It's lightweight, much more lightweight than VMs, and I have no problems runnign multiple containers at the same time.

I'll add more reasons later when I can think of something else

# Installing docker

You should checkout [Docker's website](https://www.docker.io/)

# First contact

First contact with Docker will be deploying a Jekyll page.

1. Clone a jekyll template. I'm using [Jekyll Mon Cahier](https://github.com/btquanto/jekyll_mon_cahier) in this example. It's a pretty cool template, so make sure to check it out.

    ```
    $ git clone https://github.com/btquanto/jekyll_mon_cahier.git ./jekyll_site
    $ cd ./jekyll_site
    ```

2. I don't want to get involve in building a new jekyll image yet, so I am using [btquanto/docker-jekyll](https://hub.docker.com/r/btquanto/docker-jekyll/) from docker hub.

    ```
    $ sudo docker pull btquanto/docker-jekyll
    ```

3. Create and run a jekyll docker container

    ```
    $ sudo docker run -u `id -u $USER` -d --name jekyll \
        -v `pwd`:/src \
        -p 4000:4000 \
        btquanto/docker-jekyll jekyll serve -H 0.0.0.0
    ```

4. Check your [localhost:4000](http://localhost:4000). In most cases, it should work. Yey!
5. When I was using the theme [Lanyon](https://github.com/poole/lanyon/). So, let's see what's the error, by starting the container again.

    ```
    $ sudo docker start -a jekyll
    ```

    And the error shown is that I don't have the gem `redcarpet`

    ```
    Dependency Error: Yikes! It looks like you don't have redcarpet or one of its dependencies installed.
    ```

6. Let's install `redcarpet` by upgrading the `btquanto/docker-jekyll` image
    1. Remove the failed container

        ```
    $ sudo docker rm jekyll
        ```

    2. Make a Dockerfile with the following content

        ```
    FROM btquanto/docker-jekyll
    MAINTAINER btquanto@gmail.com
    RUN gem install redcarpet
        ```

    3. Build the docker image

        ```
    $ sudo docker build -t my_jekyll_image .
        ```

7. Recreate the jekyll container with the new image

    ```
    $ sudo docker run -u `id -u $USER` -d \
        --name jekyll \
        -v `pwd`:/src \
        -p 4000:4000 \
        my_jekyll_image jekyll serve -H 0.0.0.0
    ```

8. My site is not available at [localhost:4000](http://localhost:4000). Yey!
