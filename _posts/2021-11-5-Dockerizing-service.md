---
layout: post
title: Dockerizing service.
categories: [Docker]
---
In last article we use docker for image form [dockerhub](https://hub.docker.com/). But what do we need to do for our custom application?

Of course, we will create our own images. 

I created MVP for blog-platform. This application include three services:
1. [Frontend](https://github.com/Marbok/blog-engine/tree/main/front) - react application. It uses npm for building and nginx like server.
2. [Backend](https://github.com/Marbok/blog-engine/tree/main/back) - java application with Spring and maven like build system.
3. Database - MySql.

First, we need to create two containers for frontend and backend. We can describe image in Dockerfile and docker create new image for us.

Create Dockerfile for backend.

```dockerfile
# base image
FROM maven:3.6.3-openjdk-11-slim 

# copy all files in directory in container
COPY . .
# say docker, that we use this ports
EXPOSE 8080
# docker execute this command when the container starts
# The command run application
ENTRYPOINT ["mvn", "spring-boot:run"]
```

Create Dockerfile for frontend. It's more interesting case: We will build the application in one container and run it in another.

```dockerfile
# base image for build
FROM node:10-alpine as builder

#copy dependencies description in container
COPY package.json package-lock.json ./

# install dependencies
RUN npm install \
# create project directory
    && mkdir /blog-ui \
# move dependencies in project directory
    && mv ./node_modules ./blog-ui

# set work directory
WORKDIR /blog-ui

# copy all files in directory in container
COPY . .

# create argument for build
ARG REACT_APP_HOST

# build application
RUN npm run build

# base image for running the application
FROM nginx:alpine

# copy configuration for nginx
COPY ./.nginx/nginx.conf /etc/nginx/nginx.conf

# remove static files
RUN rm -rf /usr/share/nginx/html/*

# copy new static files in nginx
COPY --from=builder /blog-ui/build /usr/share/nginx/html

# say docker, that we use next ports
EXPOSE 80

# run nginx in container
ENTRYPOINT ["nginx", "-g", "daemon off;"]
```

`ARG REACT_APP_HOST` is unobvious command. When we use environment variables in frontend, we don't have its in runtime. Build system replaces this vars in code on values. Therefore, we will not be able to use the usual environment variables in 'docker-compose.yaml`, as we did in the last [article](/Docker-compose-wordpress).

Let's describe the architecture of containers:
1. Run frontend:
    - build image
    - set url on back service.
    - mapping port 80 to 3000
2. Run backend:
    - build image
    - mapping port 8080 to 8080
    - set environment variables
    - set volume for maven (fir reusing libs)
3. Run mysql:
    - set volume for data
    - set environment variables
4. Mysql and backend use separate network

Next step, create `docker-compose.yaml`:

```yaml
services:

  mysql:
    image: mysql:5.7
    volumes:
      - mysql:/var/lib/mysql
    networks:
      back:
        aliases:
          - mysql
    environment:
      MYSQL_DATABASE: blog
      MYSQL_USER: bloguser
      MYSQL_PASSWORD: blogpass
      MYSQL_RANDOM_ROOT_PASSWORD: 1

  back:
    depends_on:
        - mysql
    build: back/ # docker create image form this directory
    networks:
      - back
    volumes:
      - maven-repo:/root/.m2
    ports:
      - "8080:8080"
    environment:
      # this vars is used in application
      HTTP_PORT: 8080
      SPRING_DATASOURCE_URL: jdbc:mysql://mysql:3306/blog
      SPRING_DATASOURCE_USERNAME: bloguser
      SPRING_DATASOURCE_PASSWORD: blogpass
      JWT_SECRET: test

  front:
    build:
      context: front/ # docker create image form this directory
      args:
        REACT_APP_HOST: localhost:8080 # use args for pass url on backend
    ports:
      - "3000:80"

networks:
  back:

volumes:
  maven-repo:
  mysql:
```

And when we run command: `docker-compose up --build` - docker start our application.
All code is [here](https://github.com/Marbok/blog-engine).

## A series of articles about docker:
1. [Docker. Run Container.](/Docker-small-task)
2. [Docker. Container and two networks.](/Docker-two-networks)
3. [Docker. Logging.](/Docker-logging)
4. [Docker. Wordpress.](/Docker-run-wordpress)
5. [Docker-compose. Wordpress.](/Docker-compose-wordpress)
6. [Dockerizing service.](/Dockerizing-service)