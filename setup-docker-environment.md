# Setting up your php development environment with Docker

For our development environment we will be using two Docker containers.  One container for 
php and one container for the nginx web server. You will need Docker installed on your 
host before continuing.

## Directory Structure

On the host you've selected, make a top-level directory for your environment.  I will be using 
an Ubuntu 20.04 host and my top-level directory will be ~/learning-php.  

Create the top-level directory and cd into it.
```
~$ cd; mkdir learning-php 
~$ cd learning-php/
~/learning-php$ ll
total 8
drwxrwxr-x  2 rob rob 4096 May 31 15:33 ./
drwxr-xr-x 18 rob rob 4096 May 31 15:33 ../
~/learning-php$ 
```

Create a directory for our source code.
```
~/learning-php$ mkdir src 
~/learning-php$ ll
total 12
drwxrwxr-x  3 rob rob 4096 May 31 15:36 ./
drwxr-xr-x 18 rob rob 4096 May 31 15:33 ../
drwxrwxr-x  2 rob rob 4096 May 31 15:36 src/
~/learning-php$ 
```

Create a directory for our nginx configuration file.
```
~/learning-php$ mkdir nginx 
~/learning-php$ ll
total 16
drwxrwxr-x  4 rob rob 4096 May 31 15:40 ./
drwxr-xr-x 18 rob rob 4096 May 31 15:33 ../
drwxrwxr-x  2 rob rob 4096 May 31 15:40 nginx/
drwxrwxr-x  2 rob rob 4096 May 31 15:36 src/
~/learning-php$ 
```

## Create a Docker Network

In order for the Nginx container to be able to communicate with the PHP container we need a 
user-defined bridge network.  The contiainers will attach to this network for communicating.

```
~/learning-php$ docker network create learning-php-network 
f9ab8d53ee378a6105db89d59de6e36a2870c8d6b7e3f05e5fb6fee75bfd66af
~/learning-php$ docker network list
NETWORK ID     NAME                   DRIVER    SCOPE
0a2ad150c95c   bridge                 bridge    local
33fc980d2b3b   host                   host      local
f9ab8d53ee37   learning-php-network   bridge    local
f2d094862a7c   none                   null      local
~/learning-php$ 
```

## PHP-FPM Docker Container

Run the PHP container with the following command:
```
~/learning-php$ docker run -d --rm --name learning-php --net learning-php-network -w /var/www -v `pwd`/src:/var/www php:8.0.2-fpm 
ad7143840bab9cf95b1722f14693c34206edd00972db97ff9c0f20c7baf93900
~/learning-php$ docker ps
CONTAINER ID   IMAGE           COMMAND                  CREATED         STATUS         PORTS      NAMES
ad7143840bab   php:8.0.2-fpm   "docker-php-entrypoi…"   9 seconds ago   Up 7 seconds   9000/tcp   learning-php
~/learning-php$ 
```
Note that the learning-php container is listening on port 9000 on the learning-php-network network. 


## Nginx Docker Container

Add the following nginx.conf file to the nginx directory.
```
~/learning-php$ cat nginx/nginx.conf 
server {
    listen 80;
    index index.php;
    error_log  /var/log/nginx/error.log;
    access_log /var/log/nginx/access.log;
    error_page 404 /index.php;
    root /var/www;
    location ~ \.php$ {
        try_files $uri =404;
        fastcgi_pass learning-php:9000;
        fastcgi_index index.php;
        include fastcgi_params;
        fastcgi_param SCRIPT_FILENAME $document_root$fastcgi_script_name;
    }
    location / {
        try_files $uri $uri/ /index.php?$query_string;
        gzip_static on;
    }
}
~/learning-php$ 
```
Note the `fastcgi_pass` directive is set to the name of the php container (learning-php) and the port it is listening on (9000).

Run the Nginx container with the following command:
```
~/learning-php$ docker run -d --rm --name learning-nginx --net learning-php-network -p 8000:80 -v `pwd`/src:/var/www -v `pwd`/nginx:/etc/nginx/conf.d nginx:1.19-alpine
93c4266893878b492adf52a349719e9ec80f1f8065083d2ecf64f98ee2adf81f
~/learning-php$ 
~/learning-php$ docker ps
CONTAINER ID   IMAGE               COMMAND                  CREATED          STATUS          PORTS                                   NAMES
93c426689387   nginx:1.19-alpine   "/docker-entrypoint.…"   10 seconds ago   Up 10 seconds   0.0.0.0:8000->80/tcp, :::8000->80/tcp   learning-nginx
ad7143840bab   php:8.0.2-fpm       "docker-php-entrypoi…"   19 minutes ago   Up 19 minutes   9000/tcp                                learning-php
~/learning-php$ 
```


## Testing the Setup

Add the following index.php file to the src directory.
```
~/learning-php$ cat src/index.php 
<?php

//phpinfo();

echo "<pre>";
print_r($_SERVER['PHP_VERSION']);
echo "</pre>\n";

?>
~/learning-php$ 
```

Finally, do a GET request to port 8000 with curl or a web browser.
```
~/learning-php$ curl http://127.0.0.1:8000 
<pre>8.0.2</pre>
~/learning-php$ 
```


You're now ready to start coding in php in the src directory. 







