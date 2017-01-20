---
layout: post
title: Deploying your blog with Jekyll
category: Tutorials
tags: ["linux", "jekyll", "tutorial"]
comments: true
---
I have just moved from wordpress to Jekyll as my new blogging platform, and it is pretty fun. I now can write my blog posts in Markdown in my favourite text editor, have my blog version controlled with git, and deploy my site with a simple git push.

Deploying Jekyll is easy, but it can as well be confusing to newbs. Thus, I am writing this tutorial.

# Choosing a Jekyll theme

There are lots of [free Jekyll themes](http://jekyllthemes.org/) out there for you to choose with a little bit of googling. The theme I am currently using is [Jekyll Mon Cahier](https://github.com/btquanto/jekyll_mon_cahier). If you're incline to develop your own theme, well, you can do it later after getting used to Jekyll first.

For the sake of this tutorial, I will be using [Jekyll Mon Cahier](https://github.com/btquanto/jekyll_mon_cahier).

```
git clone https://github.com/btquanto/jekyll_mon_cahier.git ~/blog
```

# Installing and running Jekyll

You can either choose to install Jekyll, or use a pre-built Docker image

## Installing Jekyll to your machine

1. Jekyll is a ruby gem, thus installing ruby is the pre-requisite

    ```
    curl -L https://get.rvm.io | bash -s stable --ruby=2.0.0
    ```

2. Now is the actuall installing Jekyll part

    ```
    gem install jekyll
    ```

3. Running your site is easy

    ```
    cd ~/blog
    jekyll serve 
    ```

    Learn more about the `jekyll serve` command with `jekyll serve -h`

4. Checkout your blog at [localhost:4000](http://localhost:4000/)

## Using docker

1. Using docker is even neater, but the pre-requisite is that you have to install [Docker](https://docker.io/) first. Checkout the instructions on [Docker's website](https://docker.io/)

2. Then, just pull a pre-built docker image for jekyll. I am using [btquanto/docker-jekyll](https://hub.docker.com/r/btquanto/docker-jekyll/) here.

    ```
    docker pull btquanto/docker-jekyll
    ```
3. Just run your site (follow the instructions of whichever image that you use)

    ```
    cd ~/blog
    docker run -u `id -u $USER` -d \
        --name jekyll \
        -v `pwd`:/src \
        -p 4000:4000 \
        btquanto/docker-jekyll jekyll serve -H 0.0.0.0
    ```