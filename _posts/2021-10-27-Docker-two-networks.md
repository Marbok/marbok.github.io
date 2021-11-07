---
layout: post
title: Docker. Container and two networks.
categories: [Docker]
---
This is a small hint for the case when you need to connect a container to two networks.
Example:
```bash
docker network create cache # create network cache
docker network create front # create network front
docker create --name redis redis:latest # create container with name redis
docker network connect cache redis # connect network cache to container
docker network connect front redis # connect network front to container
docker start redis # start container
```

## A series of articles about docker:
1. [Docker. Run Container.](/Docker-small-task)
2. [Docker. Container and two networks.](/Docker-two-networks)
3. [Docker. Logging.](/Docker-logging)
4. [Docker. Wordpress.](/Docker-run-wordpress)
5. [Docker-compose. Wordpress.](/Docker-compose-wordpress)
6. [Dockerizing service.](/Dockerizing-service)