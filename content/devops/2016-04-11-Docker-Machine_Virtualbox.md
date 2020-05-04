---
title: Docker-Machine and Virtualbox
date: 2016-04-11T20:19:50+01:00
draft: false
categories:
  - Cloud Native
  - Public Cloud
  - All
tags:
  - docker-machine
  - Virtualbox
---
> Quick note: the original post dates from 14-04-2016 but got updated at 01-05-2020 with latest versions and it was essentially written from scratch again.

### Introduction

Wanted to learn a bit more on Docker Machine. The idea of Docker Machine is to provision Docker hosts on remote systems. The definition on the Docker site is as follows:

> Docker Machine is a tool that lets you install Docker Engine on virtual hosts, and manage the hosts with docker-machine commands. You can use Machine to create Docker hosts on your local Mac or Windows box, on your company network, in your data center, or on cloud providers like AWS or Digital Ocean. Using docker-machine commands, you can start, inspect, stop, and restart a managed host, upgrade the Docker client and daemon, and configure a Docker client to talk to your host.

So the idea of this post is to have Docker Machine running on my MAC and use it to launch docker hosts on Virtualbox. This is actually quite nice. If you remember from all my previous Docker related posts, I was working on an Ubuntu machine I manually created on Openstack, I had to connect via SSH to this machine. Then I had to install and update the docker engine, and I had to run all commands from that host. If I wanted to provision an additional host, I had to repeat the same process over again. This is manageable for 1 or 2 hosts, but imaging you have 1000s of them. Seems like Docker machine could be a solution to that. In later posts, we will also try with cloud provider drivers such as AWS, DigitalOcean and Openstack

Let’s give it a try with Virtualbox first…

```bash
wauterw@WAUTERW-M-65P7 ~ % docker-machine create --driver virtualbox docker-host-virtualbox
Running pre-create checks...
(docker-host-virtualbox) Image cache directory does not exist, creating it at /Users/wauterw/.docker/machine/cache...
(docker-host-virtualbox) No default Boot2Docker ISO found locally, downloading the latest release...
(docker-host-virtualbox) Latest release for github.com/boot2docker/boot2docker is v19.03.5
(docker-host-virtualbox) Downloading /Users/wauterw/.docker/machine/cache/boot2docker.iso from https://github.com/boot2docker/boot2docker/releases/download/v19.03.5/boot2docker.iso...
(docker-host-virtualbox) 0%....10%....20%....30%....40%....50%....60%....70%....80%....90%....100%
Creating machine...
(docker-host-virtualbox) Copying /Users/wauterw/.docker/machine/cache/boot2docker.iso to /Users/wauterw/.docker/machine/machines/docker-host-virtualbox/boot2docker.iso...
(docker-host-virtualbox) Creating VirtualBox VM...
(docker-host-virtualbox) Creating SSH key...
(docker-host-virtualbox) Starting the VM...
(docker-host-virtualbox) Check network to re-create if needed...
(docker-host-virtualbox) Found a new host-only adapter: "vboxnet0"
(docker-host-virtualbox) Waiting for an IP...
Waiting for machine to be running, this may take a few minutes...
Detecting operating system of created instance...
Waiting for SSH to be available...
Detecting the provisioner...
Provisioning with boot2docker...
Copying certs to the local machine directory...
Copying certs to the remote machine...
Setting Docker configuration on the remote daemon...
Checking connection to Docker...
Docker is up and running!
To see how to connect your Docker Client to the Docker Engine running on this virtual machine, run: docker-machine env docker-host-virtualbox
```
Here is the docker host running on Virtualbox:

![docker-machine](/images/2016-04-11-1.png)

### Login to docker host

To test things a bit out, let's login to the docker host on Virtualbox:
```
wauterw@WAUTERW-M-65P7 ~ % docker-machine env docker-host-virtualbox
export DOCKER_TLS_VERIFY="1"
export DOCKER_HOST="tcp://192.168.99.100:2376"
export DOCKER_CERT_PATH="/Users/wauterw/.docker/machine/machines/docker-host-virtualbox"
export DOCKER_MACHINE_NAME="docker-host-virtualbox"
# Run this command to configure your shell:
# eval $(docker-machine env docker-host-virtualbox)
```
Run the command to configure the shell. Whatever you do now in your shell will effectively run on the docker host.

```bash
wauterw@WAUTERW-M-65P7 ~ % eval $(docker-machine env docker-host-virtualbox)
wauterw@WAUTERW-M-65P7 ~ % docker ps
CONTAINER ID        IMAGE               COMMAND             CREATED             STATUS              PORTS               NAMES
wauterw@WAUTERW-M-65P7 ~ % docker images
REPOSITORY          TAG
```