---
title: Deploying your blog with Jekyll
category: Tutorials
tags: ["linux", "jekyll", "tutorial"]
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
    docker run -u $UID -d \
        --name jekyll \
        -v `pwd`:/src \
        -p 4000:4000 \
        btquanto/docker-jekyll jekyll serve -H 0.0.0.0
    ```

# Deploying your blog

1. Generate a new ssh key without specifying any password. Save the key to `~/.ssh/id_rsa_np`

    ```
    ssh-keygen
    ```

2. Go to [github.com](https://github.com) and add the public key of the newly generated key pair to your account.

3. Open `~/.ssh/config`, and add the following content

    ```
    Host github.com-np
        HostName github.com
        User git
        IdentityFile ~/.ssh/id_rsa_np
        IdentitiesOnly yes
    ```

4. Create some folders for deploying your blog

    ```
    sudo mkdir /var/www/
    sudo mkdir /var/www/blog
    sudo chown $USER:$USER /var/www/blog
    mkdir /var/www/site
    mkdir /var/www/scripts
    ```

5. Clone your blog

    ```
    git clone git@github.com-np:<username>/<your-blog>.git /var/www/blog/src
    ```

6. Add a upstart script for serving your website, save it in `/etc/init/jekyll-watch.conf`

    ```
    description "Start jekyll watch for generating blog"
    author "Quan To <btquanto@gmail.com>"

    start on runlevel [2345]
    stop on runlevel [!2345]

    exec su <username> -lc "jekyll build -ws /var/www/blog/src -d /var/www/blog/site"
    respawn
    ```

    Remember to replace `<username>` with your username

    Then start the upstart job

    ```
    sudo start jekyll-watch
    ```

7. Go to your DNS, create an **A record**, and point to your server.

8. Create an nginx config in `/etc/nginx/sites-available/jekyll-blog`

    ``` nginx
    server {
        listen 80;

        server_name your.domain.name;

        root /var/www/blog/site/;
        index index.html index.htm index.php;

        location / {
        expires off;
            try_files $uri $uri/ =404;
        }

        error_page 404 /404.html;
    }
    ```

    Remember to adjust the file appropriately

    Enable the configuration file using a symlink

    ```
    sudo ln -s /etc/nginx/sites-available/jekyll-blog /etc/nginx/sites-enabled/jekyll-blog
    ```

    Reload nginx

    ```
    sudo service nginx reload
    ```

    Now you can access your website at **your.domain.name**

9. Write a script for updating your `src` folder, put it in `/var/www/blog/scripts/update.sh`

    ```
    #!/bin/bash
    SRC_DIR="/var/www/blog/src"
    cd $SRC_DIR
    git fetch
    git reset origin/master --hard
    ```

    Give it execution permission

    ```
    chmod +x /var/www/blog/scripts/update.sh
    ```

10. On your local machine, add an alias for convenient use

    ```
    echo "alias update_blog=\"ssh user@host /var/www/blog/scripts/update.sh\"" >> ~/.bashrc
    ```

That's pretty much done. Now, everytime you want your server to fetch new blog posts from github and regenerate your blog, just call `udpate_blog`.

