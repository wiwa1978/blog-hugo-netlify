---
title: Deploy Flask App to Heroku using direct Github integration
date: 2021-10-07T10:19:50+01:00
draft: true
categories:
  - Cloud Native
  - DevOps
  - All
tags:
  - Flask
  - Elastic Beanstalk
  - Github Actions
---

### Introduction

There are different ways to deploy Flask apps to Heroku. In this post, we will make use of

![flask-basic-github-heroku](/images/2021-11-07-1.png)

### Configure Github repository

```bash
~/Flask-Basic-CICD-GithubActions-Heroku ❯ git init
Initialized empty Git repository in /Users/wauterw/SynologyDrive/Programming/blog-hugo-netlify-code/Flask/Flask-Basic-CICD-Heroku/.git/
~/Flask-Basic-CICD-GithubActions-Heroku ❯ git branch -m main
~/Flask-Basic-CICD-GithubActions-Heroku ❯ git add README.md
~/Flask-Basic-CICD-GithubActions-Heroku ❯ git commit -m "Initial commit"
[main (root-commit) ef0f8e5] Initial commit
 1 file changed, 1 insertion(+)
 create mode 100644 README.md
~/Flask-Basic-CICD-GithubActions-Heroku ❯ git remote add origin https://github.com/wiwa1978/flask-basic-cicd-heroku.git
~/Flask-Basic-CICD-GithubActions-Heroku ❯ git push origin mainEnumerating objects: 3, done.
Counting objects: 100% (3/3), done.
Delta compression using up to 12 threads
Compressing objects: 100% (2/2), done.
Writing objects: 100% (3/3), 308 bytes | 308.00 KiB/s, done.
Total 3 (delta 0), reused 0 (delta 0), pack-reused 0
To https://github.com/wiwa1978/flask-basic-cicd-heroku.git
 * [new branch]      main -> main
```

![flask-basic-github-heroku](/images/2021-11-07-2.png)

### Configure Heroku

Create new app

![flask-basic-github-heroku](/images/2021-11-07-3.png)

Go toe the `Deploy` tab and connect yout Github repository:

![flask-basic-github-heroku](/images/2021-11-07-4.png)

![flask-basic-github-heroku](/images/2021-11-07-5.png)

Also enable Automatic Deploys

![flask-basic-github-heroku](/images/2021-11-07-6.png)

### Commit our application

```bash
~/Flask-Basic-CICD-GithubActions-Heroku ❯ git add .
~/Flask-Basic-CICD-GithubActions-Heroku ❯ git commit -m "Adding application"
[main 83c38b7] Adding application
 8 files changed, 72 insertions(+)
 create mode 100644 .gitignore
 create mode 100644 Dockerfile
 create mode 100644 Procfile
 create mode 100644 app/__pycache__/app.cpython-39.pyc
 create mode 100644 app/app.py
 create mode 100644 app/templates/index.html
 create mode 100644 requirements.txt
 create mode 100644 wsgi.py
~/Flask-Basic-CICD-GithubActions-Heroku ❯ git push origin main
Enumerating objects: 14, done.
Counting objects: 100% (14/14), done.
Delta compression using up to 12 threads
Compressing objects: 100% (9/9), done.
Writing objects: 100% (13/13), 2.11 KiB | 721.00 KiB/s, done.
Total 13 (delta 0), reused 0 (delta 0), pack-reused 0
To https://github.com/wiwa1978/flask-basic-cicd-heroku.git
   ef0f8e5..83c38b7  main -> main
```

![flask-basic-github-heroku](/images/2021-11-07-7.png)

### Check Heroku Deployment

![flask-basic-github-heroku](/images/2021-11-07-8.png)

### Showing off the result

![flask-basic-github-heroku](/images/2021-11-07-9.png)

### Updating our app

Update the index.html file. I added `version 2` to the tagline under the title.

```bash
~/Flask-Basic-CICD-GithubActions-Heroku ❯ git add .
~/Flask-Basic-CICD-GithubActions-Heroku ❯ git commit -m "Update application"
[main 034cf0e] Update application
 1 file changed, 1 insertion(+), 1 deletion(-)
~/Flask-Basic-CICD-GithubActions-Heroku ❯ git push origin main
Enumerating objects: 9, done.
Counting objects: 100% (9/9), done.
Delta compression using up to 12 threads
Compressing objects: 100% (4/4), done.
Writing objects: 100% (5/5), 405 bytes | 405.00 KiB/s, done.
Total 5 (delta 3), reused 0 (delta 0), pack-reused 0
remote: Resolving deltas: 100% (3/3), completed with 3 local objects.
To https://github.com/wiwa1978/flask-basic-cicd-heroku.git
   83c38b7..034cf0e  main -> main
```

You will notice that a build is in progress

![flask-basic-github-heroku](/images/2021-11-07-10.png)

And if all went well you should see the updated application.

![flask-basic-github-heroku](/images/2021-11-07-11.png)

Code can be found [here](https://github.com/wiwa1978/blog-hugo-netlify-code/tree/main/Flask/Flask-Basic-CICD-Heroku)
