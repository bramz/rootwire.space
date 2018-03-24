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
In this post I assume users have experience with docker and sqlite3. I suggest some reading of docker documentation at <a href="http://docs.docker.com">docs.docker.com</a>.
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
