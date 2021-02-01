---
title: Deploy Flask Application to Heroku
date: 2021-01-31T20:19:50+01:00
draft: true
categories:
  - Cloud Native
  - DevOps
  - All
tags:
  - Docker
  - Docker-compose
  - Flask
  - Heroku
---

### Introduction

In [this](https://blog.wimwauters.com/devops/2021-02-01-FlaskBasic) post we have created a very basic Flask application. Now we want to deploy it onto Docker. Let's see how this can be accomplished. I'm going to assume that you have followed along that post or have done a git clone of the application.

Also, if you want to follow along with this post, you should install `Docker Desktop`. You can download it [here](https://www.docker.com/products/docker-desktop).


### Create Heroku account

![flask-basic](/images/2021-02-05-1.png)


### Create Heroku application


![flask-basic](/images/2021-02-05-2.png)

![flask-basic](/images/2021-02-05-3.png)


### Flask application

~/Flask/Flask-Basic-Heroku main ?1 ❯ pip3 install flask
~/Flask/Flask-Basic-Heroku main ?1 ❯ pip3 install gunicorn 
~/Flask/Flask-Basic-Heroku main ?1 ❯ pip3 freeze > requirements.txt 


### Procfile

```bash
web: gunicorn wsgi:app
```

### Login to heroku

```bash
~Flask/Flask-Basic-Heroku master !1 ❯ heroku login 
 ›   Warning: heroku update available from 7.44.0 to 7.47.11.
heroku: Press any key to open up the browser to login or q to exit: 
Opening browser to https://cli-auth.heroku.com/auth/cli/browser/d06a957b-1050-48da-aacd-473e68404fbb?requestor=SFMyNTY.g2gDbQAAAA45NC4xMDQuMTE0LjEyMm4GAK4mjll3AWIAAVGA.Y99SRx68GHEp8zvLkMm-h3cp70GQkkvbtPwEM0oTtq0
Logging in... done
Logged in as w***@gmail.com
```


### Git

![flask-basic](/images/2021-02-05-4.png)

```
~/Flask/Flask-Basic-Heroku main !2 ?4 ❯ git init 
Initialized empty Git repository in /Users/wauterw/SynologyDrive/Programming/blog-hugo-netlify-code/Flask/Flask-Basic-Heroku/.git/
~/Flask/Flask-Basic-Heroku master +7 !1 ❯ heroku git:remote -a flask-basic-heroku 
set git remote heroku to https://git.heroku.com/flask-basic-heroku.git

```



### Deploy to Heroku

```bash
~/Flask/Flask-Basic-Heroku master +7 !1 ❯ git add .
~/Flask/Flask-Basic-Heroku master +7 ❯ git commit -am "Initial commit" 
[master (root-commit) 79f53f9] Initial commit
 7 files changed, 71 insertions(+)
 create mode 100644 .gitignore
 create mode 100644 Dockerfile
 create mode 100644 Procfile
 create mode 100644 app/app.py
 create mode 100644 app/templates/index.html
 create mode 100644 requirements.txt
 create mode 100644 wsgi.py
~/Flask/Flask-Basic-Heroku master ❯ git push heroku master 
Enumerating objects: 11, done.
Counting objects: 100% (11/11), done.
<TRUNCATED>
remote:        https://flask-basic-heroku.herokuapp.com/ deployed to Heroku
remote: 
remote: Verifying deploy... done.
To https://git.heroku.com/flask-basic-heroku.git
 * [new branch]      master -> master
```


![flask-basic](/images/2021-02-05-5.png)

