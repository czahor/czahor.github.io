---
layout: post
title: The joys of Docker Compose for development
image: docker-compose.jpg
---

At IJR, we use Docker extensively in production and love it: from powering our API servers, our caching layers, our frontend web servers, and more. But I think the best part about Docker -- the part that will save developers the most time and frustration -- is using Docker Compose locally, during the process of development.

Why? Because mature, scalable production environments look incredibly similar from the inside -- i.e., in your code -- yet radically different from the outside. This can lead to hours of frustration, where you'll uncover learnings such as the `set()` command in version 3.2.2 differs oh-so-slightly-yet-enough-to-break-everything in version 3.2.3.

In order to replicate your production environment locally, you'll have to create several different services, all running at the same time, on different ports, and some of them might have to be able to talk to each other. That could be a real pain in the butt, and using only Docker (without Compose), you'd have to run commands like `docker run -p someport:someport someservice`, with slight variations, over and over again. You could get slightly fancy and alias the commands in your bash profile. But if you want to be ultra fancy, all you need is a docker-compose.yaml file that looks something like this:

```yaml
version: '3'

services:
  elasticsearch:
    image: elasticsearch:5.1
    ports:
      - "9200:9200"

  dynamodb:
    build:
      context: ./
      dockerfile: dynamodb.dockerfile
    ports:
      - "8000:8000"

  memcached:
    image: memcached:1.4
    ports:
      - "11211:11211"

  redis:
    image: redis:3.2.4
    ports:
      - "6379:6379"

  mysql:
    image: mysql:5.6
    ports:
      - "3306:3306"
    environment:
      MYSQL_ROOT_PASSWORD: password
      MYSQL_DATABASE: ijr_development
      MYSQL_USER: user
      MYSQL_PASSWORD: password
    volumes:
      - ./bin/create-databases.sh:/docker-entrypoint-initdb.d/create-databases.sh

  realtime:
    build:
      context: ../realtime
    environment:
      REDIS_HOST: redis
    ports:
      - "3003:3003"
    depends_on:
      - redis
    links:
      - redis
```

As long as you have Docker Compose up and running, now all you have to do is run `docker-compose up -d` and everything fires up! Pretty cool, huh.

For some of the services -- `elasticsearch`, `memcached`, `redis`, `mysql` -- all that happens is an image is downloaded from Docker Hub, started, and bound to certain port. For `dynamodb`, a custom Dockerfile is used, only because I couldn't find a suitable Dynamodb image on Dockerhub. And for `realtime`, which is a proprietary codebase of ours, the Dockerfile located in a relative directory is used.

(Tip: if you're using your own codebase, and you modify it, you'll have to tell Docker to rebuild the image. We do this by running `docker-compose stop realtime`, then `docker-compose build realtime`, then `docker-compose up -d`.)

Do you use Docker Compose? Is there something better out there? If so, let us know in the [Hacker News](http://news.ycombinator.com) comments.
