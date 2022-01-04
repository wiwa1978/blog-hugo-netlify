---
title: Deploy Flask App to DigitalOcean using Docker
date: 2021-10-20T10:19:50+01:00
draft: true
categories:
  - Cloud Native
  - DevOps
  - All
tags:
  - Flask
  - Github Actions
  - DigitalOcean
---

### Introduction

### Configure Github repository

![flask-basic-github-docker-droplet](/images/2021-11-20-3.png)

![flask-basic-github-docker-droplet](/images/2021-11-20-4.png)

### Commit our application

Explain the application

```bash
~/Flask-Basic-CICD-GithubActions-DigitalOcean-Droplet ❯ git init
~/Flask-Basic-CICD-GithubActions-DigitalOcean-Droplet ❯ git add .
~/Flask-Basic-CICD-GithubActions-DigitalOcean-Droplet ❯ git commit -m "Initial commit"
~/Flask-Basic-CICD-GithubActions-DigitalOcean-Droplet ❯ git remote add origin https://github.com/wiwa1978/Flask-Basic-CICD-GithubActions-DigitalOcean-Droplet.git
~/Flask-Basic-CICD-GithubActions-DigitalOcean-Droplet ❯ git push origin main
```

### Configure DigitalOcean

Create DigitalOcean token
![flask-basic-github-docker-droplet](/images/2021-11-20-8.png)
Make a copy of the token as we will need to create a secret in our Github repository (later section).

Create a droplet on DigitalOcean via Doctl:

```bash
~/Flask-Basic-CICD-GithubActions-DigitalOcean-Droplet ❯ doctl compute droplet create Flask-server --image docker-18-04 --size s-1vcpu-1gb --region ams3  --wait
```

If all goes well, you will receive an email with the username and password for this droplet. You will need to login to the droplet using ssh. You could use `ssh root@IPaddress` for this using the password that was mentioned in the email. Next change your password using the console and store this as a secret in your Github repository (later section).

![flask-basic-github-docker-droplet](/images/2021-11-20-1.png)

Create container registry

![flask-basic-github-docker-droplet](/images/2021-11-20-6.png)

![flask-basic-github-docker-droplet](/images/2021-11-20-7.png)

### Create secrets on Github repo

![flask-basic-github-docker-droplet](/images/2021-11-20-5.png)

The `DOCKER_USERNAME` and `DOCKER_PASSWORD` are in fact the API token you created on Digitalocean

### Add Github actions workflow

![flask-basic-github-docker-droplet](/images/2021-11-20-9.png)

Pull the main.yml file describing the workflow:

```bash
~/Flask-Basic-CICD-GithubActions-DigitalOcean-Droplet ❯ git pull origin main
remote: Enumerating objects: 6, done.
remote: Counting objects: 100% (6/6), done.
remote: Compressing objects: 100% (3/3), done.
remote: Total 5 (delta 1), reused 0 (delta 0), pack-reused 0
Unpacking objects: 100% (5/5), 1.25 KiB | 256.00 KiB/s, done.
From https://github.com/wiwa1978/-Flask-Basic-CICD-GithubActions-DigitalOcean-Droplet
 * branch            main       -> FETCH_HEAD
   d9f9dc8..c2d7f06  main       -> origin/main
Updating d9f9dc8..c2d7f06
Fast-forward
 .github/workflows/main.yml | 36 ++++++++++++++++++++++++++++++++++++
 1 file changed, 36 insertions(+)
 create mode 100644 .github/workflows/main.yml
```

### Update Github actions workflow

Show the workflow:

```yaml

```

```bash
~/Flask-Basic-CICD-GithubActions-DigitalOcean-Droplet ❯ git add .
~/Flask-Basic-CICD-GithubActions-DigitalOcean-Droplet ❯ git commit -m "Update workflow"
[main 9057d79] Update workflow
 1 file changed, 54 insertions(+), 16 deletions(-)
 wauterw@WAUTERW-M-65P7  ~/SynologyDrive/Programming/blog-hugo-netlify-code/Flask/Flask-Basic-CICD-GithubActions-DigitalOcean-Droplet   main  git push origin main
Enumerating objects: 9, done.
Counting objects: 100% (9/9), done.
Delta compression using up to 12 threads
Compressing objects: 100% (3/3), done.
Writing objects: 100% (5/5), 1.33 KiB | 1.33 MiB/s, done.
Total 5 (delta 1), reused 0 (delta 0), pack-reused 0
remote: Resolving deltas: 100% (1/1), completed with 1 local object.
To https://github.com/wiwa1978/-Flask-Basic-CICD-GithubActions-DigitalOcean-Droplet.git
   c2d7f06..9057d79  main -> main
```

![flask-basic-github-docker-droplet](/images/2021-11-20-10.png)

![flask-basic-github-docker-droplet](/images/2021-11-20-11.png)

You will see that the image got uploaded to the DigitalOcean registry:

![flask-basic-github-docker-droplet](/images/2021-11-20-12.png)

### Showing off the result

![flask-basic-github-docker-droplet](/images/2021-11-20-13.png)

### Updating our app

```
 Simple Flask application using Tailwind and deployed through Github Actions to a DigitalOcean droplet running a Docker container -- version 2
```

Add the docker stop and docker rm commands:

```
script: |
           # login docker
           docker login -u $DOCKER_USER -p $DOCKER_PASSWORD registry.digitalocean.com

           docker stop $(echo $IMAGE_NAME)

           docker rm $(echo $IMAGE_NAME)

           # Run a new container from a new image
           docker run -d \
           --restart always \
           -p 80:5000 \
           --name $(echo $IMAGE_NAME) \
           $(echo $REGISTRY)/$(echo $IMAGE_NAME):$(echo $GITHUB_SHA | head -c7)
```

![flask-basic-github-docker-droplet](/images/2021-11-20-14.png)
