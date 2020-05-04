---
title: Remove all Docker images and containers
date: 2016-04-10T20:19:50+01:00
draft: false
categories:
  - Cloud Native
  - All
tags:
  - docker
---

> Quick note: the original post dates from 10-04-2016 but got updated at 02-05-2020 and it was essentially written from scratch again.

### Introduction

This post just as a reminder how to delete all containers and images. While experimenting with Docker I continuously needs these commands and instead of always Googling them I might be better of just writing a small post. Exactly the reason why I once decided to blog.

### Remove all containers
The following command will remove all the containers:

```bash
root@ubuntu-demo:/home/cloud-user# docker rm `docker ps -qa`
```

### Remove all images
The following command will remove all the images from your machine:

```bash
root@ubuntu-demo:/home/cloud-user# docker rmi $(docker images -q)
```

### Remove all ghost images from Docker-Machine

Sometimes the creation of a host fails and it leaves a ghost entry when doing “docker-machine ls”. Below command will completely remove all hosts so be careful when using it.

```bash
root@ubuntu-demo:/home/cloud-user# docker-machine rm -f $(docker-machine ls -q);
```

Very small post but over the years it's probably the one I referred back to most of the time.