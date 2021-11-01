---
layout: post
title: Docker. Logging.
categories: [Docker]
---
Experiment with Docker continue!
Now I need to configure logs using [Fluentd](https://en.wikipedia.org/wiki/Fluentd).
I have a next technical task:
1. Create `/data/logs` directory on host system with 777 permitions.
2. Run container with name `fluentd` with next parameters:
    - The container must be available on port `24224`
    - `/fluentd/log` directory inside the container must be mounted to the `/data/logs` on host system
    - Image is `fluentd:latest`
3. Run container with name `nginx` with next parameters:
    - Image is `nginx:latest`
    - Logging driver is `fluentd`
    - Fluentd server's address is `127.0.0.1:24224'

As a result, we have following commands:
```bash
mkdir /data/logs  # create directory
chmod 777 /data/logs/  # set permitions
docker run -d --name fluentd -v /data/logs/:/fluentd/log \ 
           -p 24224:24224 fluentd:latest # run fluentd
docker run -d --name nginx --log-driver=fluentd \
           --log-opt fluentd-address=127.0.0.1:24224 \
           -p 8080:80 nginx:latest # run nginx
```