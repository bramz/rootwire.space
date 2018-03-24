---
author: "Brock Ramsey"
date: 2018-03-24
linktitle: Containerize Sqlite3 With Docker
menu:
  main:
    parent: tutorials
title: Containerize Sqlite3 With Docker
weight: 10
---
In this post I assume users have experience with docker and sqlite3. I suggest some reading of docker documentation at <a href="http://docs.docker.com">docs.docker.com</a>. Also, one may want to read up on usage of Sqlite3 as well if not familiar, located at <a href="https://sqlite.org/docs.html">https://sqlite.org/docs.html</a>. I use docker to maintain persitence environements for my projects, I use it for both sqlite and postgres.
## The Makefile
Create a file named `Makefile`with the following content.
```
prefix = /usr/local
install:; install bin/sqlite3 $(prefix)/bin
```
## Our Dockerfile
Here, we must create a docker file, and build our image. To build, execute: `docker build dir/with/dockerfile`.
```
FROM debian:latest

RUN apt-get update && \
    DEBIAN_FRONTEND=noninteractive apt-get -yq install sqlite3 && \
    rm -rf /var/lib/apt/lists/* /tmp/* /var/tmp/*

RUN mkdir -p /root/db

WORKDIR /root/db
ENTRYPOINT [ "sqlite3" ]
```
## Compose Alternative
Creating a `docker-compose.yml` and running with `docker-compose up`.
```
version: '3'

services:
  sqlite3:
    image: brockramz/sqlite3:latest
    stdin_open: true
    tty: true
    volumes:
      - ./db/:/root/db/
```
