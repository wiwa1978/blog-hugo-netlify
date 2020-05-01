---
title: Docker-Machine and DigitalOcean
date: 2016-04-14T20:19:50+01:00
draft: false
categories:
  - Cloud Native
  - Public Cloud
  - All
tags:
  - docker-machine
  - DigitalOcean
---
> Quick note: the original post dates from 14-04-2016 but got updated at 01-05-2020 with latest versions and it was essentially written from scratch again.

### Introduction

In this post we started of with Docker Machine and Virtualbox. Then we moved on to something more complex, we launched a docker host on AWS post. Just for fun, I wanted to try it also on DigitalOcean and documented it in this post. Rather straightforward as you will notice soon.

### Getting all info

I’m assuming you already have an DigitalOcean account. You will need to create a token on DigitalOcean. The procedure is very well explained [here](https://docs.docker.com/machine/examples/ocean/).

![DO](/images/2016-04-14-1.png)

### Using Docker-Machine

Creating a docker host on Digitalocean is very straightforward. See below the command to achieve this:

```
wauterw@WAUTERW-M-65P7 blog-hugo-netlify % docker-machine create --driver digitalocean --digitalocean-access-token 469***12 docker-1-digitalocean
```
You will see that in a minimum amount of time a Droplet is created on DigitalOcean.

![DO](/images/2016-04-14-2.png)

It will take some time for docker-machine to provision the server:

```
wauterw@WAUTERW-M-65P7 blog-hugo-netlify % docker-machine create --driver digitalocean --digitalocean-access-token 469***12 docker-1-digitalocean
Running pre-create checks...
Creating machine...
(docker-1-digitalocean) Creating SSH key...
(docker-1-digitalocean) Creating Digital Ocean droplet...
(docker-1-digitalocean) Waiting for IP address to be assigned to the Droplet...
Waiting for machine to be running, this may take a few minutes...
Detecting operating system of created instance...
Waiting for SSH to be available...
Detecting the provisioner...
Provisioning with ubuntu(systemd)...
Installing Docker...
Copying certs to the local machine directory...
Copying certs to the remote machine...
Setting Docker configuration on the remote daemon...
Checking connection to Docker...
Docker is up and running!
To see how to connect your Docker Client to the Docker Engine running on this virtual machine, run: docker-machine env docker-1-digitalocean
```
We can then launch Docker containers on this newly provisioned DigitalOcean host, but we refer to previous tutorials on Docker Machine on how to do this exactly.

### SSH into the droplet using Docker-Machine

```
wauterw@WAUTERW-M-65P7 blog-hugo-netlify % docker-machine env docker-1-digitalocean
export DOCKER_TLS_VERIFY="1"
export DOCKER_HOST="tcp://159.203.118.24:2376"
export DOCKER_CERT_PATH="/Users/wauterw/.docker/machine/machines/docker-1-digitalocean"
export DOCKER_MACHINE_NAME="docker-1-digitalocean"
# Run this command to configure your shell: 
# eval $(docker-machine env docker-1-digitalocean)
wauterw@WAUTERW-M-65P7 blog-hugo-netlify % eval $(docker-machine env docker-1-digitalocean)
wauterw@WAUTERW-M-65P7 blog-hugo-netlify % docker-machine ssh docker-1-digitalocean
Welcome to Ubuntu 16.04.6 LTS (GNU/Linux 4.4.0-169-generic x86_64)
***Truncated***
```

```
wauterw@WAUTERW-M-65P7 ~ % docker --version
Docker version 19.03.8, build afacb8b
wauterw@WAUTERW-M-65P7 ~ % base=https://github.com/docker/machine/releases/download/v0.16.0 &&
  curl -L $base/docker-machine-$(uname -s)-$(uname -m) >/usr/local/bin/docker-machine &&
  chmod +x /usr/local/bin/docker-machine
  % Total    % Received % Xferd  Average Speed   Time    Time     Time  Current
                                 Dload  Upload   Total   Spent    Left  Speed
100   639  100   639    0     0   1646      0 --:--:-- --:--:-- --:--:--  1646
100 32.0M  100 32.0M    0     0  3499k      0  0:00:09  0:00:09 --:--:-- 2329k
wauterw@WAUTERW-M-65P7 ~ % docker-machine --version
docker-machine version 0.16.0, build 702c267f

```