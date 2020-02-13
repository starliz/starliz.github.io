---
layout: posts
title: "Assignment 5, LAMP stack"
date: 2017-02-17
---
#### Linux Basics (dat8tf063-29)

#### HAAGA-HELIA University of Applied Sciences.

#### Last modified on 26.02.2017

### LAMP inside a docker:

I was hesitant about installing LAMP into my home directory and at the same time wanted to preserve results after exercising, but I got a hint to use docker. So I decided to try it for this homework. [The task details are here](http://terokarvinen.com/2017/agenda-for-linux-basics-dat8tf063-29-spring-2017#comment-22169).

All resulting docker files can be found in [my liz-docker-lamp repo](https://github.com/starliz/liz-docker-lamp).

"Docker containers wrap a piece of software in a complete filesystem that contains everything needed to run 
and isolate applications from one another and the underlying infrastructure".

"Docker provides the ability to package and run an application in a loosely isolated environment called a container."

It has client-server architecture, so user interacts with daemon via client (``docker`` CLI utility).

A Docker **image** is a read-only template with instructions for creating a Docker container. Each image consists of a series of layers. - **build** component.

A Docker **container** is a runnable instance of a Docker image. - **run** component (start, stop, move, or delete).

A docker **registry** is a library of images. A registry can be public or private. - **distribution** component.



### Docker Installation:

First according to [manual](https://docs.docker.com/engine/installation/linux/fedora/) I checked that I don't have `` docker docker-common container-selinux`` and ``docker-selinux`` packages.

Example:

````
[liz@localhost ~]$ dnf list docker-selinux
Last metadata expiration check: 0:01:50 ago on Mon Feb 20 20:07:00 2017.
Available Packages
docker-selinux.x86_64                2:1.10.3-9.git667d6d1.fc24                 fedora
````
"Available" tells me that package is not installed. Same way I made sure that I've got ``dnf-plugins-core``.

To setup repo for installing and updating docker I did:

````
sudo dnf config-manager --add-repo https://docs.docker.com/engine/installation/linux/repo_files/fedora/docker.repo
````

Updating the dnf package index:

````
[liz@localhost ~]$ sudo dnf makecache fast
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
[liz@localhost ~]$ sudo systemctl start docker
````

Verify that docker is installed correctly by running the hello-world image. 
When the container runs, it prints an informational message and exits:

````
[liz@localhost ~]$ sudo docker run hello-world
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
[liz@localhost ~]$ cat /etc/group | grep docker
docker:x:972:
[liz@localhost ~]$ sudo usermod -aG docker liz
[sudo] password for liz: 
[liz@localhost ~]$ cat /etc/group | grep docker
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

Since the task was to set up LAMP step by step, I couldn't just take ready made solution and run it, but I had to try to create my own image.

The examples of Dockerfile and some parts for supervisord.conf I took [here](https://github.com/tutumcloud/lamp) and [here](https://github.com/nickistre/docker-lamp/tree/ubuntu-14.04)

NOTE: I didn't bother with user pages, just put index files to default root directory because it runs in docker isolated anyway.

This is my initial Dockerfile (Each instruction creates a new layer in the image.):

````
# use some ubuntu blueprint image
FROM ubuntu:trusty
# Install packages
# Avoid need for manual intervention or the interactive post-install configuration dialog.
ENV DEBIAN_FRONTEND noninteractive
# Get information on the newest versions of packages and their dependencies.
RUN apt-get update -y
# "apt-get -y" - Assume "yes" as answer to all prompts and run non-interactively. 
RUN apt-get -y install apache2 curl
````
Next step is to try building. On my laptop I create directory, put my Dockerfile there and build "liz/lamp":

````
mkdir liz-docker-lamp
cd ./liz-docker-lamp/
docker build -t liz/lamp .
````
It executed all preset steps, downloaded Ubuntu image and installed additional packages & apache2.

Useful commands to administrate:
* ``docker --help`` 
* ``docker run -i --rm liz/lamp /bin/bash`` - "-i" run interactive (Keep STDIN open even if not attached) and run bash process, "-rm" to automatically remove the container when it exits.
    * Commands useful [inside] container: 
    * ``ss -ltpn`` - display listening sockets.
    * ``curl http://localhost/`` -  to request default http page.
* ``docker run -p 8080:80 liz/lamp`` - port -p external:internal, about exposing ports see[expose-incoming-ports](https://docs.docker.com/engine/reference/run/#expose-incoming-ports)
* ``docker ps -a`` - Show all containers (default shows just running)
* ``docker images`` = ``docker image ls``
* ``docker image prune`` - Remove unused images
* ``docker image rm`` - Remove one or more images

Example how to clean up running containers:
````
[liz@localhost liz-docker-lamp]$ docker ps -a
CONTAINER ID        IMAGE               COMMAND                  CREATED             STATUS                      PORTS               NAMES
e85a76bb3e10        cc4d85dd0241        "/bin/sh -c 'apt-g..."   24 hours ago        Exited (100) 24 hours ago                       gifted_archimedes
[liz@localhost liz-docker-lamp]$ docker rm gifted_archimedes
gifted_archimedes
[liz@localhost liz-docker-lamp]$ docker ps -a
CONTAINER ID        IMAGE               COMMAND             CREATED             STATUS              PORTS               NAMES
````

"The Docker image is read-only. When Docker runs a container from an image, it adds a read-write layer on top of the image in which needed app runs."


### BASH, SSH and etc for convenience and interaction

[Mostly copied from here](https://github.com/nickistre/docker-lamp/blob/ubuntu-14.04/Dockerfile)

Adding to Dockerfile:
````
# install supervisord
RUN apt-get install -y supervisor
RUN mkdir -p /var/log/supervisor
ADD supervisord.conf /etc/
````
and
````
RUN apt-get install -y bash-completion vim tmux openssh-server openssh-client passwd
RUN mkdir -p /var/run/sshd
RUN sed -ri 's/PermitRootLogin without-password/PermitRootLogin yes/g' /etc/ssh/sshd_config
RUN echo 'root:xxx' | chpasswd
EXPOSE 22
CMD ["supervisord", "-n"]
````
Supervisord allows its users to monitor and control processes. Traditionally a Docker container runs a single process when it is launched, but if you want to run more than one process in a container, you can use ``supervisord``. See more in [Docker documentation](https://docs.docker.com/engine/admin/using_supervisord/)

And to supervisord.conf:
````
[program:sshd]
command = /usr/sbin/sshd -D
````
Trying to build and run:
````
docker build -t liz/lamp .
docker run --rm -p 2222:22 liz/lamp
````
Try SSH connection:
````
[liz@localhost ~]$ ssh -p 2222 root@localhost
root@localhost's password: 
Welcome to Ubuntu 14.04 LTS (GNU/Linux 4.4.0-59-generic x86_64)
...
root@add8ea906cd9:~# 
^C
root@792fe79475ea:~# logout
Connection to localhost closed.
````

### Further Dockerfile development: programming Apache to start when running docker:

I put to my docker directory ("liz-docker-lamp" where Dockerfile is):
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

apache2.conf - I've lost track where I took this specific config version, but important is that it has in the end: ``ServerName localhost``

supervisord.conf: 
````
[supervisord]
nodaemon=true                ; start in foreground 
[program:apache2]
command=/bin/bash -c "source /etc/apache2/envvars && exec /usr/sbin/apache2 -DFOREGROUND"
````
Now web server can be run inside a container and accessed via normal Browser (<http://localhost:8080/>) on the same laptop.

````
[liz@localhost liz-docker-lamp]$ docker build -t liz/lamp .
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

![apache-runs-in-docker.png]({{ site.url }}/assets/01-apache-runs-in-docker.png)


### NEXT: homepages/index file

I don't create directory in user's home, since apache in docker runs as root, I put my custom index.html to /var/www/html/index.html (found this path in config). And hence it can be accessed by same address "http://localhost:8080/".

![lamp-custom-html.png]({{ site.url }}/assets/02-lamp-custom-html.png)

TODO: TEST IT BY IP

###  MYSQL

Added to Dockerfile installation of:
* ``mysql-server`` - MySQL server
* ``mysql-client`` - MySQL client
* ``pwgen`` - to create password for mysql user.

Other additions to Dockerfile:
````
ADD start-mysqld.sh /start-mysqld.sh
ADD my.cnf /etc/mysql/conf.d/my.cnf
ADD create_mysql_admin_user.sh /create_mysql_admin_user.sh
RUN rm -rf /var/lib/mysql/*
ENV MYSQL_PASS foobar
ADD mysql-setup.sh /mysql-setup.sh
RUN chmod 755 /*.sh
VOLUME  ["/etc/mysql", "/var/lib/mysql" ]
EXPOSE 3306

ADD run.sh /run.sh
RUN chmod 755 /*.sh

CMD ["/run.sh"] #Instead of   CMD ["supervisord", "-n"]
````

I put files to docker directory:

run.sh:
````
#!/bin/bash
exec supervisord -n
````

start-mysql.sh:
````
#!/bin/bash
service mysql start
````

my.cnf:   --- user and password can/may be stored here.
````
[mysqld]
bind-address=0.0.0.0
````

And ``create_mysql_admin_user.sh`` - creates admin user with password from variable MYSQL_PASS, grants privileges.

``mysql-setup.sh`` -  creates database "busplan", grants all to "busman" with password "southparks2e17", creates table "plan" and fills it with "Step one, ..., PROFIT!!!"

To supervisord.conf:
````
[program:mysqld]
command = /usr/bin/pidproxy /var/run/mysqld/mysqld.pid /usr/bin/mysqld_safe
````

### TRY MYSQL CRUD: Create, Read, Update, Delete.

Building image:
````
[liz@localhost liz-docker-lamp]$ docker build -t liz/lamp .
Sending build context to Docker daemon 23.04 kB
Step 1/27 : FROM ubuntu:trusty
...
Step 27/27 : CMD /run.sh
...
Successfully built 20baf749aa64

````

Run:

````
[liz@localhost liz-docker-lamp]$ docker run --rm -i -p 2222:22 liz/lamp
=> An empty or uninitialized MySQL volume is detected in /var/lib/mysql
=> Installing MySQL ...
=> Done!
=> Waiting for confirmation of MySQL service startup
=> Creating MySQL admin user with random password
=> Done!
========================================================================
You can now connect to this MySQL Server using:

    mysql -uadmin -pfoobar -h<host> -P<port>

Please remember to change the above password as soon as possible!
MySQL user 'root' has no password but only allows local connections
========================================================================
/usr/lib/python2.7/dist-packages/supervisor/options.py:295: UserWarning: Supervisord is running as root and it is searching for its configuration file in default locations (including its current working directory); you probably want to specify a "-c" argument specifying an absolute path to a configuration file for improved security.
  'Supervisord is running as root and it is searching '
2017-02-25 12:30:17,804 CRIT Supervisor running as root (no user in config file)
2017-02-25 12:30:17,807 INFO supervisord started with pid 1
2017-02-25 12:30:18,810 INFO spawned: 'sshd' with pid 433
2017-02-25 12:30:18,812 INFO spawned: 'mysqld' with pid 434
2017-02-25 12:30:18,815 INFO spawned: 'apache2' with pid 435
2017-02-25 12:30:20,008 INFO success: sshd entered RUNNING state, process has stayed up for > than 1 seconds (startsecs)
2017-02-25 12:30:20,009 INFO success: mysqld entered RUNNING state, process has stayed up for > than 1 seconds (startsecs)
2017-02-25 12:30:20,009 INFO success: apache2 entered RUNNING state, process has stayed up for > than 1 seconds (startsecs)
2017-02-25 13:29:41,740 CRIT reaped unknown pid 842)

````

In terminal on laptop (providing root user password):
````
[liz@localhost ssh]$ ssh -p 2222 root@localhost
root@localhost's password: 
Welcome to Ubuntu 14.04 LTS (GNU/Linux 4.4.0-59-generic x86_64)
...
root@f22a31b4ecda:~# mysql -uadmin -pfoobar 
Welcome to the MySQL monitor.  Commands end with ; or \g.
...
mysql> 
mysql> CREATE DATABASE busplan CHARACTER SET utf8;
Query OK, 1 row affected (0.00 sec)
mysql> SHOW DATABASES;
+--------------------+
| Database           |
+--------------------+
| information_schema |
| busplan            |
| mysql              |
| performance_schema |
+--------------------+
4 rows in set (0.00 sec)

mysql> GRANT ALL ON busplan.* TO busman@localhost IDENTIFIED BY 'southparks2e17';
Query OK, 0 rows affected (0.00 sec)
mysql> exit
Bye
root@f22a31b4ecda:~# mysql -ubusman -psouthparks2e17
Welcome to the MySQL monitor.  Commands end with ; or \g.
...
mysql>  SHOW DATABASES;
+--------------------+
| Database           |
+--------------------+
| information_schema |
| busplan            |
+--------------------+
2 rows in set (0.00 sec)

mysql> USE busplan;
Database changed
mysql> CREATE TABLE plans (id INT AUTO_INCREMENT PRIMARY KEY, name VARCHAR(1024));
Query OK, 0 rows affected (0.02 sec)
mysql> DESCRIBE plans;
+-------|---------------|------|-----|---------|----------------+
| Field | Type          | Null | Key | Default | Extra          |
+-------|---------------|------|-----|---------|----------------+
| id    | int(11)       | NO   | PRI | NULL    | auto_increment |
| name  | varchar(1024) | YES  |     | NULL    |                |
+-------|---------------|------|-----|---------|----------------+
2 rows in set (0.01 sec)
mysql> INSERT INTO plans(name) VALUES ("Step one");
Query OK, 1 row affected (0.01 sec)

mysql> INSERT INTO plans(name) VALUES ("Step two");
Query OK, 1 row affected (0.01 sec)

mysql> INSERT INTO plans(name) VALUES ("????");
Query OK, 1 row affected (0.01 sec)

mysql> INSERT INTO plans(name) VALUES ("PROFIT!!!");
Query OK, 1 row affected (0.01 sec)

mysql> SELECT * FROM plans;
+----|-----------+
| id | name      |
+----|-----------+
|  1 | Step one  |
|  2 | Step two  |
|  3 | ????      |
|  4 | PROFIT!!! |
+----|-----------+
4 rows in set (0.00 sec)

````

Successfully created database "busplan", granted all to "busman" with password "southparks2e17", created table "plan" and filled it with "Step one, ..., PROFIT!!!"

TODO: update and delete is easy and hints can be found in [this page](http://terokarvinen.com/2016/mysql-install-and-one-table-database-sql-crud-tutorial-for-ubuntu)


### The PHP app page

Added to Dockerfile installation of:
* ``libapache2-mod-php5`` -  Apache PHP5 module
* ``php5-mysql`` - To get MySQL support in PHP

NOTE: php7 is latest

Additions to Dockerfile:
````
#Environment variables to configure php
ENV PHP_UPLOAD_MAX_FILESIZE 10M
ENV PHP_POST_MAX_SIZE 10M

RUN rm /var/www/html/index.htm*
ADD index.php /var/www/html/
````

Content of ``run.sh`` file I copied from: <https://github.com/tutumcloud/lamp/blob/master/run.sh>

A first part of index.php I took from from <https://github.com/fermayo/hello-world-lamp/blob/master/index.php> (Apache license 2.0) and modified it a bit.

Another template I used is [here](http://terokarvinen.com/2016/read-mysql-database-with-php-php-pdo).

Here is the content of index.php:[liz-docker-lamp/.../index.php](https://github.com/starliz/liz-docker-lamp/blob/master/index.php). It has two parts at the same time, one is simply PHP "Hello world" and second reads from database and prints it's content:

![lamp-runs-and-php-reads-mysq.png]({{ site.url }}/assets/03-lamp-php-reads-mysql.png)


Logs
* /var/log/apache2/*.log 
* /var/log/mysql/error.log 
* /etc/os-release 
* /proc/uptime

see [here](https://github.com/starliz/liz-docker-lamp/blob/master/info.txt), it is written to /var/www/html/info.txt  (in container).

* * *

### Used articles:

[Out-of-the-box LAMP image (PHP+MySQL) usage](https://github.com/tutumcloud/lamp)

[Docker installation](https://docs.docker.com/engine/installation/linux/fedora/)

[Some docker post-installation set up](https://docs.docker.com/engine/installation/linux/linux-postinstall/)

[Understanding Docker](https://docs.docker.com/engine/understanding-docker/)

[Dockerfile reference](https://docs.docker.com/engine/reference/builder/)

[Some other ideas for docker files](https://github.com/nickistre/docker-lamp/tree/ubuntu-14.04)

[Docker cheat sheet](https://github.com/wsargent/docker-cheat-sheet)

[MySQL tutorial](http://terokarvinen.com/2016/mysql-install-and-one-table-database-sql-crud-tutorial-for-ubuntu)

[Read MySQL with PHP](http://terokarvinen.com/2016/read-mysql-database-with-php-php-pdo)

Alternative way is to use [Django-Apache-Python](http://terokarvinen.com/2017/django-on-apache-with-python-3-on-ubuntu-16-04)

And some more tutorials from Tero Karvinen:
* <http://terokarvinen.com/2008/install-apache-web-server-on-ubuntu-4>
* <http://terokarvinen.com/2016/mysql-install-and-one-table-database-sql-crud-tutorial-for-ubuntu>
* <http://terokarvinen.com/2012/change-mysql-password-dpkg-reconfigure-mysql-server-5-1-ubuntu-debian-error-1045>
* <http://php.net/manual/en/tutorial.forms.php>
* <http://terokarvinen.com/2016/read-mysql-database-with-php-php-pdo>
* <http://terokarvinen.com/2016/instant-firewall-sudo-ufw-enable>


##### “ Copyright 2017 Lidia Zalevskaya, GNU General Public License, version 3 or later.”
