---
title: Docker-Machine and AWS
date: 2016-04-12T20:19:50+01:00
draft: false
categories:
  - Cloud Native
  - Public Cloud
  - All
tags:
  - docker-machine
  - AWS
---
> Quick note: the original post dates from 14-04-2016 but got updated at 01-05-2020 with latest versions and it was essentially written from scratch again.

### Introduction

In [this](https://blog.wimwauters.com/devops/2016-04-11-docker-machine_virtualbox/) post, we experimented a bit with Docker Machine and Virtualbox. We were able to successfully launch a docker host on Virtualbox. It would be interesting to try this now also on AWS. Note that for this post, I’m assuming you already have an AWS account.

### Getting all info from AWS

You will need to gather some information from AWS:

* your AWS Access Key ID
* your AWS Secret Access Key
* your region in which you want to launch your instance
* your VPC id for that region

##### Getting AWS Access Key and Secret Access Key

In your AWS console, go to ‘Identity & Access Management’, then either create a user or click on an existing user. 

![aws](/images/2016-04-12-1.png)

As we gave the user programmatic access, they will show up in the last screen. You can download them (they will only be displayed once).

![aws](/images/2016-04-12-2.png)

Then, go to your local machine (in my case the MAC) and create a file ~/.aws/credentials with the following content:

```
[default]
    aws_access_key_id = <access_key>
    aws_secret_access_key = <secret_key>
```

Of course, change the placeholders with the value of your own credentials.

##### Getting your AWS region

By default, the AWS driver creates new instances in region us-east-1 (North Virginia). As I live in Europe, I prefer something closer. You can do this by specifying a different region by using the –amazonec2-region flag. For that, you will need to know the official name for your region. The easiest is to go to here and check under ‘Available Regions’

##### Getting your AWS VPC ID

AWS creates your EC2 instances (by default) in a default VPC. So you will also need that one. To do so, go to your region (in my case Ireland (eu-west-1) and go to the VPC dashboard. Click on the VPC and take a note of the VPC-ID. Again, you will need this one later on.


### Using Docker Machine

We did quite some preparation work, time has come now to get started with the docker-machine command. We will create an docker ready EC2 instance in the Ireland region. Do this as follows:

```bash
wauterw@WAUTERW-M-65P7 ~ % docker-machine create --driver amazonec2 --amazonec2-vpc-id vpc-6d351c0b --amazonec2-region eu-west-1 aws-docker1
Running pre-create checks...
Creating machine...
(aws-docker1) Launching instance...
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
To see how to connect your Docker Client to the Docker Engine running on this virtual machine, run: docker-machine env aws-docker1
```
Wait about a minute or 2 and you will see that an EC2 instance with name aws-docker1 is spawning on AWS. Let me show a screenshot:

![aws](/images/2016-04-12-3.png)

The whole process takes about a minute or 5 before the docker-machine command is finished completely installing docker on the host, etc…

### Experimenting with Docker Machine

```bash
wauterw@WAUTERW-M-65P7 ~ % docker-machine ls
NAME                     ACTIVE   DRIVER         STATE     URL                         SWARM   DOCKER     ERRORS
aws-docker1              -        amazonec2      Running   tcp://54.77.9.141:2376              v19.03.8
docker-host-virtualbox   -        virtualbox     Running   tcp://192.168.99.100:2376           v19.03.5
wauterw@WAUTERW-M-65P7 ~ %
```
When you create a new machine, your command shell automatically connects to it. In case this is not so, you’ll have to run eval $(docker-machine env aws-docker1). How I got that one? See below…

```bash
wauterw@WAUTERW-M-65P7 ~ % docker-machine env aws-docker1
export DOCKER_TLS_VERIFY="1"
export DOCKER_HOST="tcp://54.77.9.141:2376"
export DOCKER_CERT_PATH="/Users/wauterw/.docker/machine/machines/aws-docker1"
export DOCKER_MACHINE_NAME="aws-docker1"
# Run this command to configure your shell:
# eval $(docker-machine env aws-docker1)
wauterw@WAUTERW-M-65P7 ~ % eval $(docker-machine env aws-docker1)
```
From now on, every docker command you will supply is running on the AWS host called ‘aws-docker1’. Let’s try things a bit…:

```bash
wauterw@WAUTERW-M-65P7 ~ % docker run hello-world
Unable to find image 'hello-world:latest' locally
latest: Pulling from library/hello-world
0e03bdcc26d7: Pull complete
Digest: sha256:8e3114318a995a1ee497790535e7b88365222a21771ae7e53687ad76563e8e76
Status: Downloaded newer image for hello-world:latest

Hello from Docker!
```
Let's now see if the container has run. From your local MAC, do the following:

```bash
wauterw@WAUTERW-M-65P7 ~ % docker ps -a
CONTAINER ID        IMAGE               COMMAND             CREATED             STATUS                      PORTS               NAMES
eb2b6924759a        hello-world         "/hello"            34 seconds ago      Exited (0) 32 seconds ago                       friendly_hodgkin
```
This works from our local machine but how can we be sure that we are running the above docker command against the AWS docker host (and not our local machine). Therefore, we need to login to our docker host on AWS through SSH.

### SSH into the AWS instance

If you look at the AWS console, you will see that the instance aws-docker1 has a keypair called ‘aws-docker1’. The issue is that you cannot download it. If you browse through keypairs, it’s clear that there is no option to download keypairs that have been generated previously. So how to get into the instance then? Luckily docker-machine has an ‘ssh’ subcommand that allows us to get access to the instance. 

```bash
wauterw@WAUTERW-M-65P7 ~ % docker-machine ssh aws-docker1
Welcome to Ubuntu 16.04.4 LTS (GNU/Linux 4.4.0-1052-aws x86_64)
```

```bash
ubuntu@aws-docker1:~$ sudo su
root@aws-docker1:/home/ubuntu# docker ps
CONTAINER ID        IMAGE               COMMAND             CREATED             STATUS              PORTS               NAMES
root@aws-docker1:/home/ubuntu# docker ps -a
CONTAINER ID        IMAGE               COMMAND             CREATED             STATUS                     PORTS               NAMES
eb2b6924759a        hello-world         "/hello"            2 minutes ago       Exited (0) 2 minutes ago                       friendly_hodgkin
```
You'll see that this output is exactly the same as the output from our local machine in the previous section. Pretty neat, no?

### Control host via docker-machine

Let's begin with listing all machines under control of docker-machine:

```bash
wauterw@WAUTERW-M-65P7 ~ % docker-machine ls
NAME                     ACTIVE   DRIVER         STATE     URL                         SWARM   DOCKER     ERRORS
aws-docker1              *        amazonec2      Running   tcp://54.77.9.141:2376              v19.03.8
docker-host-virtualbox   -        virtualbox     Running   tcp://192.168.99.100:2376           v19.03.5
```
Next, let's stop the docker host on AWS.

```bash
wauterw@WAUTERW-M-65P7 ~ % docker-machine stop aws-docker1
Stopping "aws-docker1"...
Machine "aws-docker1" was stopped.
```

If you then go to your AWS console, you’ll see the instance was stopped.

![aws](/images/2016-04-12-4.png)

Obviously, we’re also able to remove a remote docker host. Do the following:

```bash
wauterw@WAUTERW-M-65P7 ~ % docker-machine rm aws-docker1
About to remove aws-docker1
WARNING: This action will delete both local reference and remote instance.
Are you sure? (y/n): y
Successfully removed aws-docker1
```
You will then see that the ‘aws-docker1’ host on AWS is in terminated state (and will soon disappear):

![aws](/images/2016-04-12-5.png)