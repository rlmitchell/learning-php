# Setting up your php development environment with Docker

For our development environment we will be using two docker containers.  One container for 
php and one container for the nginx web server. 

## Directory workspace

On the host you've selected, make a top-level directory for your environment.  I will be using 
a Ubuntu 20.04 host and my directory will be ~/learning-php.  

Create the directory and cd into it.
```
~$ cd; mkdir learning-php 
~$ cd learning-php/
~/learning-php$ ll
total 8
drwxrwxr-x  2 rob rob 4096 May 31 15:33 ./
drwxr-xr-x 18 rob rob 4096 May 31 15:33 ../
~/learning-php$ 
```


