---
layout: posts
title: "Assignment 5, LAMP stack"
date: 2017-02-17
---
#### Linux Basics (dat8tf063-29)

#### HAAGA-HELIA University of Applied Sciences.

### LAMP inside a docker:

I was in doubts about installing LAMP into my home directory, but I got a hint to use docker.
So I decided to use it for this homework. 

"Docker containers wrap a piece of software in a complete filesystem that contains everything needed to run 
and isolate applications from one another and the underlying infrastructure".

"Docker provides the ability to package and run an application in a loosely isolated environment called a container."

It has client-server architecture, so user interacts with daemon via client (``docker`` CLI utility).

A Docker **image** is a read-only template with instructions for creating a Docker container. Each image consists of a series of layers. - **build**

A Docker **container** is a runnable instance of a Docker image. - **run**, start, stop, move, or delete.

A docker **registry** is a library of images. A registry can be public or private. - **distribution**



### Docker Installation:

First according to manual I checked that I don't have `` docker docker-common container-selinux`` and ``docker-selinux`` packages.

Example:

````
[liz@localhost log]$ dnf list docker-selinux
Last metadata expiration check: 0:01:50 ago on Mon Feb 20 20:07:00 2017.
Available Packages
docker-selinux.x86_64                2:1.10.3-9.git667d6d1.fc24                 fedora
````
"Available" tells me that package is not installed. Same way I made sure that I've got ``dnf-plugins-core``.

To setup repo for installing and updating docker:

````
sudo dnf config-manager --add-repo https://docs.docker.com/engine/installation/linux/repo_files/fedora/docker.repo
````

Updating the dnf package index:

````
[liz@localhost log]$ sudo dnf makecache fast
...
Metadata cache created.
````
Installing:

````
[liz@localhost log]$ sudo dnf install docker-engine
...
========================================================================================================================================================================
 Package                                          Arch                             Version                                  Repository                             Size
========================================================================================================================================================================
Installing:
 audit-libs-python                                x86_64                           2.7.1-1.fc24                             updates                                95 k
 docker-engine                                    x86_64                           1.13.1-1.fc24                            docker-main                            19 M
 docker-engine-selinux                            noarch                           1.13.1-1.fc24                            docker-main                            33 k
 libsemanage-python                               x86_64                           2.5-6.fc24                               updates                               109 k
 policycoreutils-python                           x86_64                           2.5-16.fc24                              updates                               403 k
 python-IPy                                       noarch                           0.81-15.fc24                             fedora                                 42 k

Transaction Summary
========================================================================================================================================================================
...
 Fingerprint: 5811 8E89 F3A9 1289 7C07 0ADB F762 2157 2C52 609D
 From       : https://yum.dockerproject.org/gpg
Is this ok [y/N]: y
Key imported successfully
...
Complete!
````

Start Docker:

````
[liz@localhost log]$ sudo systemctl start docker
````

Verify that docker is installed correctly by running the hello-world image. 
When the container runs, it prints an informational message and exits:

````
[liz@localhost log]$ sudo docker run hello-world
...
latest: Pulling from library/hello-world
...
Status: Downloaded newer image for hello-world:latest

Hello from Docker!
This message shows that your installation appears to be working correctly.

To generate this message, Docker took the following steps:
 1. The Docker client contacted the Docker daemon.
 2. The Docker daemon pulled the "hello-world" image from the Docker Hub.
 3. The Docker daemon created a new container from that image which runs the
    executable that produces the output you are currently reading.
 4. The Docker daemon streamed that output to the Docker client, which sent it
    to your terminal.
````

Docker daemon always runs as root, it binds to Unix socket, this socket is owned by the user root and other users can only access it using sudo. 
When the docker daemon starts, it makes the ownership of the Unix socket read/writable by the docker group.
I found already existing group called docker and added my user to it. (NB: The docker group grants privileges equivalent to the root user.)

````
[liz@localhost log]$ cat /etc/group | grep docker
docker:x:972:
[liz@localhost log]$ sudo usermod -aG docker liz
[sudo] password for liz: 
[liz@localhost log]$ cat /etc/group | grep docker
docker:x:972:liz
````
Then logged out and logged in back in so that my group membership got re-evaluated.
Verifying that docker commands work without sudo:

````
$ docker run hello-world
...
Hello from Docker!
This message shows that your installation appears to be working correctly.
...
````

### Crafting initial Dockerfile for building custom image

Since the task was to set up LAMP step by step, I couldn't just take ready made sollution and run it, but I had to try to create my own image.

[The example of Dockerfile I took here](https://github.com/tutumcloud/lamp/blob/master/Dockerfile) and some parts of supervisor config from [here: supervisord.conf](https://github.com/nickistre/docker-lamp/blob/ubuntu-14.04/supervisord.conf)

This is my initial Dockerfile (Each instruction creates a new layer in the image.):

````
# use some ubuntu blueprint image
FROM ubuntu:trusty
# Install packages
# Avoid need for manual intervention or the interactive post-install configuration dialog.
ENV DEBIAN_FRONTEND noninteractive
# Get information on the newest versions of packages and their dependencies.
RUN apt-get update
# "apt-get -y" - Assume "yes" as answer to all prompts and run non-interactively. 
RUN apt-get -y install apache2
````
Next step is to try building. I create directory, put my Dockerfile there and build "liz/lamp":

````
mkdir liz-docker-lamp
cd ./liz-docker-lamp/
docker build -t liz/lamp .
````
It executed all preset step, downloaded Ubuntu image and installed additional packages & apache2.

Useful commands to administrate:
* ``docker --help`` 
* ``docker run -i --rm liz/lamp /bin/bash`` - "-i" run interactive (Keep STDIN open even if not attached) and run bash process, "-rm" to automatically remove the container when it exits.
    * Commands to use [inside] container: 
    * ``ss -ltpn`` - display listening sockets.
    * ``curl http://localhost/`` -  to request default http page.
* ``docker run -p 8080:80 liz/lamp`` - port -p external:internal, about exposing ports see[expose-incoming-ports](https://docs.docker.com/engine/reference/run/#expose-incoming-ports)
* ``docker ps -a`` - Show all containers (default shows just running)
* ``docker images`` = ``docker image ls``

"The Docker image is read-only. When Docker runs a container from an image, it adds a read-write layer on top of the image in which needed app runs."

### Further Dockerfile development: programming Apache to start when running docker:

Copied to my docker directory (where Dockerfile is):
* ``start-apache2.sh``, 
* ``apache_default``, 
* ``apache2.conf`` 
* ``supervisord.conf``

start-apache2.sh:
````
#!/bin/bash
service apache2 start
````

apache_default: copied from [github.com/tutumcloud/lamp/.../apache_default](https://github.com/tutumcloud/lamp/blob/master/apache_default)

apache2.conf - WHERE THE HELL IT CAME FROM???? but important is that it has in the end: ``ServerName localhost``

supervisord.conf: 
````
[supervisord]
nodaemon=true                ; start in foreground 

[program:apache2]
command=/bin/bash -c "source /etc/apache2/envvars && exec /usr/sbin/apache2 -DFOREGROUND"
````
Now web server can be run inside a container and accessed via normal Browser on the same laptop.

````
[liz@localhost liz-docker-lamp]$ docker run -p 8080:80 liz/lamp
...
2017-02-21 20:43:34,079 CRIT Supervisor running as root (no user in config file)
2017-02-21 20:43:34,081 INFO supervisord started with pid 1
2017-02-21 20:43:35,084 INFO spawned: 'apache2' with pid 9
2017-02-21 18:03:02,247 INFO success: apache2 entered RUNNING state, process has stayed up for > than 1 seconds (startsecs)
````
Dockerfile at this point:

````
FROM ubuntu:trusty
ENV DEBIAN_FRONTEND noninteractive
RUN apt-get update -y

# Installing apache, curl ...
RUN apt-get -y install apache2 curl
RUN mkdir -p /var/lock/apache2 /var/run/apache2

# Add image configuration and scripts
ADD start-apache2.sh /start-apache2.sh

# use prepared configuration file for Apache
ADD apache2.conf /etc/apache2/apache2.conf
RUN chmod 755 /*.sh

# config to enable .htaccess
ADD apache_default /etc/apache2/sites-available/000-default.conf

# install supervisord
RUN apt-get install -y supervisor
RUN mkdir -p /var/log/supervisor
ADD supervisord.conf /etc/

# map apache 80 port
# EXPOSE informs Docker that the container listens on the specified network ports
EXPOSE 80

CMD ["supervisord", "-n"]
````

The default web page loads in Google Chrome:

![1 apache_runs_in_docker.png](_images/01_apache_runs_in_docker.png)

![2 apache-runs-in-docker.png](_images/01-apache-runs-in-docker.png)

![3 apacherunsindocker.png](_images/01apacherunsindocker.png)




### Used articles:

[Out-of-the-box LAMP image (PHP+MySQL) usage](https://github.com/tutumcloud/lamp)

[Docker installation](https://docs.docker.com/engine/installation/linux/fedora/)

[Some docker post-installation set up](https://docs.docker.com/engine/installation/linux/linux-postinstall/)

[Understanding Docker](https://docs.docker.com/engine/understanding-docker/)

[Docker cheat sheet](https://github.com/wsargent/docker-cheat-sheet)

##### “ Copyright 2017 Lidia Zalevskaya, GNU General Public License, version 3 or later.”
