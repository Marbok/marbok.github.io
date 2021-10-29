---
layout: post
title: Docker. Run Container.
categories: [Docker, Task]
---
I had a small task - run container with parameters:
1. Name is `web`
2. Image is `nginx:latest`
3. The container must be available on port `9099`
4. The `/var/log/nginx` directory inside the container must be mounted to the `/data/nginx/logs` directory on the host system
5. When starting the container, the environment variable `logging` = `true` should be added

Complete instruction for starting the container:
```bash
docker run -d --name web -p 9099:80 -v /data/nginx/logs:/var/log/nginx \
           -e logging=true nginx:latest
```

Explanation:
- **docker run** - start container
- **-d** - detached mode
- **--name web** - set name
- **-p 9099:80** - port forwarding - 9099 is host's port, 80 is container's port
- **-v /data/nginx/logs:/var/log/nginx** - mount directory, analog: `--mount type=mount,src=/data/nginx/logs,dst=/var/log/nginx`
- **-e logging=true** - set environment variable
- **nginx:latest** - nginx is image, latest is tag(sometimes version)