---
layout: post
title: Using Python in web development with Apache2 and mod_python
is_recommended: false
category: Experiment
tags: ["python", "apache2", "mod_python"]
---

**Target system**: Ubuntu 14.04 LTS
**Disclaimer**: This is not a good practice. The experiment is for learning purpose only

# Installing necessary tools and libraries

1. Installing: `apache2` and `libapache2-mod-python`
    ```
    sudo apt-get install apache2 libapache2-mod-python -y
    ```
2. Check installation complete by going to `http://localhost/`
3. Check if mod python has been enabled by checking the existent of `/etc/apache2/mods-enabled/python.load`
4. If the file in step 3 does not exist, add the following line to `/etc/apache2/apache2.conf`
    ```
    LoadModule python_module /usr/lib/apache2/modules/mod_python.so
    ```

# Adding test script

Add `index.py` to a folder (Let's say `/var/www/html/`) with the following content
```python
from mod_python import apache

def handler(request):
    request.content_type = "text/html"
    request.write("Hello World")
    return apache.OK
```

# Adding a server config

Add a file `/etc/apache2/sites-enabled/server.conf` or modify the default `/etc/apache2/sites-enabled/000-default.conf` with the following content
```apache
<VirtualHost *:80>
    DocumentRoot /var/www/html

    <Directory /var/www/html>
        SetHandler mod_python
        PythonHandler index
        PythonDebug On
    </Directory>
</VirtualHost>
```
Restart apache server
```
sudo service apache2 restart
```

# Test

Go to `http://localhost/`. Press `F5` to refresh if necessary. It should show
```
Hello World
```
