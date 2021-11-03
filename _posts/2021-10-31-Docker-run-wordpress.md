---
layout: post
title: Docker. Wordpress.
categories: [Docker]
---
Now we try to run [Wordpress](https://en.wikipedia.org/wiki/WordPress) in Docker.

First, we create a small technical task:
1. We will use fluentd for logs. First container!
2. MySql will be used as a database. Also data will be stored in volume. Second container!
3. Wordpress. We will special volume for it's data. Third container!
4. MySql and Wordpress will be used separate network.

Okey! Let's go!

Run contailner with fluentd:
```bash
mkdir /data/logs # create directory with logs
chmod 777 /data/logs # set permissions
docker run -d --name fluentd \ # run the container with name fluentd
           -p 24224:24224 \ # port forwarding on host system
           --mount type=bind,src=/data/logs,dst=/fluentd/log \ # mount directory in container
           fluentd:latest # image's name
```

Create new network:
```bash
docker network create back
```

Run container with MySql:
```bash
docker volume create mysql # create volume for mysql
docker run -d --name mysql \ # run container with name mysql
           -e MYSQL_DATABASE=wpdb \ # set env var for db
           -e MYSQL_USER=wpuser \
           -e MYSQL_PASSWORD=wppass \
           -e MYSQL_RANDOM_ROOT_PASSWORD=1 \
           -v mysql:/var/lib/mysql \ # mount volume in container
           --net back \ # set network
           --log-driver=fluentd \ # set log driver
           --log-opt fluentd-address=127.0.0.1:24224 \ # set fluentd address
           mysql:5.7 # image's name
```

Run container with Wordpress:
```bash
docker volume create uploads # create volume for wordpress
docker run -d --name wordpress \ # run container with name wordpress
           -e WORDPRESS_DB_HOST=mysql \ # set env var for wordpress
           -e WORDPRESS_DB_USER=wpuser \
           -e WORDPRESS_DB_PASSWORD=wppass \
           -e WORDPRESS_DB_NAME=wpdb \
           -v uploads:/var/www/html \ # mount volume in container
           --net back \ # set network
           -p 80:80 \ # port forwarding on host system
           --log-driver=fluentd \ # set log driver
           --log-opt fluentd-address=127.0.0.1:24224 \ # set fluentd address
           wordpress:latest # image's name'
```

And that's all! 

If we go to the address localhost:80, we will see the wordpress start panel.
I think it's not comfortable to run multiple linked containers using only Docker.
In [the next article](/Docker-compose-wordpress) I'll show you how the guys form "Docker, Inc." solved this problem.