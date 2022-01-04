---
title: Deploy Flask App to DigitalOcean Apps
date: 2021-10-25T10:19:50+01:00
draft: true
categories:
  - Cloud Native
  - DevOps
  - All
tags:
  - Flask
  - Github Actions
  - DigitalOcean
  - DigitalOcean Aps
---

### Introduction

### Configure Github repository

![flask-basic-github-docker-apps](/images/2021-11-25-1.png)

First, just add a `README.md` file. It's important that your repository is created and a branch is available

```bash
~/Flask-Basic-CICD-GithubActions-DigitalOcean-Apps ❯ git init
~/Flask-Basic-CICD-GithubActions-DigitalOcean-Apps ❯ git branch -m main
~/Flask-Basic-CICD-GithubActions-DigitalOcean-Apps ❯ git add .
~/Flask-Basic-CICD-GithubActions-DigitalOcean-Apps ❯ git commit -m "Initial Commit"
~/Flask-Basic-CICD-GithubActions-DigitalOcean-Apps ❯ git push -u origin main
```

![flask-basic-github-docker-apps](/images/2021-11-25-2.png)

### Configure DigitalOcean

In the DigitalOcean console, create an App.

![flask-basic-github-docker-apps](/images/2021-11-25-3.png)

Choose Github

![flask-basic-github-docker-apps](/images/2021-11-25-4.png)

![flask-basic-github-docker-apps](/images/2021-11-25-5.png)

![flask-basic-github-docker-apps](/images/2021-11-25-6.png)

Change the `Run Command`

```bash
gunicorn --worker-tmp-dir /dev/shm app:app
```

Change the port

![flask-basic-github-docker-apps](/images/2021-11-25-7.png)

Give the app a name

![flask-basic-github-docker-apps](/images/2021-11-25-8.png)

Select the Basic plan
![flask-basic-github-docker-apps](/images/2021-11-25-9.png)

Click on `Launch Basic app`

### Deploying the app

![flask-basic-github-docker-apps](/images/2021-11-25-10.png)

![flask-basic-github-docker-apps](/images/2021-11-25-11.png)

![flask-basic-github-docker-apps](/images/2021-11-25-12.png)

![flask-basic-github-docker-apps](/images/2021-11-25-13.png)

![flask-basic-github-docker-apps](/images/2021-11-25-14.png)
