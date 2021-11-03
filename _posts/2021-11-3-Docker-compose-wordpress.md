---
layout: post
title: Docker-compose. Wordpress.
categories: [Docker]
---
In [the last article](/Docker-run-wordpress), we launched wordpress using docker commands. Now we will try to automate it with docker-compose.

[Docker-compose](https://docs.docker.com/compose/) is a tool for defining and running multi-container Docker applications. All configuration is described in a special file and we need to use only one command: `docker-compose up`.

If you use Linux, you need to install Docker-compose separately https://docs.docker.com/compose/install/.

Ok! Let's go!

First, we put the environment variable in separate files, because very often configuration is stored in version control system and we don't want to compromise passwords.

Create two files: 
1. mysql.env
```bash
MYSQL_DATABASE=wpdb
MYSQL_USER=wpuser
MYSQL_PASSWORD=wppass
MYSQL_RANDOM_ROOT_PASSWORD=1
```
2. wordpress.env
```bash
WORDPRESS_DB_HOST=mysql:3306
WORDPRESS_DB_USER=wpuser
WORDPRESS_DB_PASSWORD=wppass
WORDPRESS_DB_NAME=wpdb
```

Now, we need to create file with services configuration: `docker-compose.yaml`. We will use three blocks:
1. Services - for describing containers
2. Networks
3. Volumes
Networks and volumes are very simple, because we only create them and use default configuration.

```yaml
volumes:
  mysql: {}
  uploads: {}

networks:
  back: {}
```

Then describe our containers in `services` section.
First, fluentd container:
```yaml
fluentd:
  image: fluentd:latest
  volumes:
    - type: bind
      source: /data/logs
      target: /fluentd/log
  ports:
    - "24224:24224"

mysql:
  depends_on:
    - fluentd
  image: mysql:5.7
  restart: always
  volumes:
    - mysql:/var/lib/mysql
  logging:
    driver: fluentd
    options:
      fluentd-address: 127.0.0.1:24224
  networks:
    back:
      aliases:
        - mysql
  env_file:
    - ./mysql.env

wordpress:
  depends_on:
    - fluentd
    - mysql
  image: wordpress:latest
  restart: always
  volumes:
    - uploads:/var/www/html
  logging:
    driver: fluentd
    options:
      fluentd-address: 127.0.0.1:24224
  networks:
    - back
  ports:
    - "80:80"
  env_file:
    - ./wordpress.env
```

It very seems like the docker commands. But I added some new options:
1. depends_on - it needs for launch order
2. restart - it tells docker restart container if it has crashed
3. networks.back.aliases - service's name in network

And that's all! Now we can run `docker-compose up` and our services launch.

All code in [github](https://github.com/Marbok/docker-compose-wordpress-blog).