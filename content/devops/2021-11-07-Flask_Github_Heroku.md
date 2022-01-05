---
title: Deploy Flask App to Heroku using Github
date: 2021-11-07T10:19:50+01:00
draft: False
categories:
  - Cloud Native
  - DevOps
  - All
tags:
  - Flask
  - Heroku
  - Github Actions
---

### Introduction

In [this](https://blog.wimwauters.com/devops/2021-02-05-flaskbasic-heroku/) post, we deployed a basic Flask application to Heroku by making use of Heroku git. In [this](https://blog.wimwauters.com/devops/2021-11-03-flask_githubactions_heroku/) post, we used Github and Github Actions to deploy our application. There's a different way still I want to touch on. It's not so much different compared to previous posts but well worth to dedicate a seperate post to it. In this post, we will make use of the existing Heroku - Git integration. Let's get started!

### Configure Github repository

If you have been reading through my previous "how to deploy a Flask application" posts, you'll know we start off with configuring a Github repository for our application. In my case, it’s called flask-basic-cicd-heroku.

![flask-basic-github-heroku](/images/2021-11-07-1.png)

And we go ahead and sync our local application code towards our Github repository.

```bash
~/Flask-Basic-CICD-Heroku ❯ git init
Initialized empty Git repository in /Flask/Flask-Basic-CICD-Heroku/.git/
~/Flask-Basic-CICD-Heroku ❯ git branch -m main
~/Flask-Basic-CICD-Heroku ❯ git add README.md
~/Flask-Basic-CICD-Heroku ❯ git commit -m "Initial commit"
[main (root-commit) ef0f8e5] Initial commit
 1 file changed, 1 insertion(+)
 create mode 100644 README.md
~/Flask-Basic-CICD-Heroku ❯ git remote add origin https://github.com/wiwa1978/flask-basic-cicd-heroku.git
~/Flask-Basic-CICD-Heroku ❯ git push origin mainEnumerating objects: 3, done.
Counting objects: 100% (3/3), done.
Delta compression using up to 12 threads
Compressing objects: 100% (2/2), done.
Writing objects: 100% (3/3), 308 bytes | 308.00 KiB/s, done.
Total 3 (delta 0), reused 0 (delta 0), pack-reused 0
To https://github.com/wiwa1978/flask-basic-cicd-heroku.git
 * [new branch]      main -> main
```

### Configure Heroku

Next, login to the Heroku platform with your account and create a new app just as I did in below screenshot.

![flask-basic-github-heroku](/images/2021-11-07-3.png)

Go to the `Deploy` tab and connect yout Github repository:

![flask-basic-github-heroku](/images/2021-11-07-4.png)

Take a good look at the 'Deployment Method'. You will see various options there. The `Heroku Git` option was covered in [this](https://blog.wimwauters.com/devops/2021-02-05-flaskbasic-heroku/) post while the `Github` option is what is covered in this post.

![flask-basic-github-heroku](/images/2021-11-07-5.png)

Also enable `Automatic Deploys`

![flask-basic-github-heroku](/images/2021-11-07-6.png)

### Commit our application

Next, we need to sync our local application to the remote Github repository.

```bash
~/Flask-Basic-CICD-Heroku ❯ git add .
~/Flask-Basic-CICD-Heroku ❯ git commit -m "Adding application"
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
~/Flask-Basic-CICD-Heroku ❯ git push origin main
Enumerating objects: 14, done.
Counting objects: 100% (14/14), done.
Delta compression using up to 12 threads
Compressing objects: 100% (9/9), done.
Writing objects: 100% (13/13), 2.11 KiB | 721.00 KiB/s, done.
Total 13 (delta 0), reused 0 (delta 0), pack-reused 0
To https://github.com/wiwa1978/flask-basic-cicd-heroku.git
   ef0f8e5..83c38b7  main -> main
```

You'll see the files in your Github repository.

![flask-basic-github-heroku](/images/2021-11-07-7.png)

### Check Heroku Deployment

Next, login to your Heroku account, In the Overview tab you will see that your app is being deployed. So I imagine that Git sends a Webhook to Heroku to trigger the deployment (remember we set it to auto-deploy from the main branch).

![flask-basic-github-heroku](/images/2021-11-07-8.png)

### Showing off the result

And here it is... Our app has been deployed automatically to Heroku.

![flask-basic-github-heroku](/images/2021-11-07-9.png)

### Updating our app

As we did in the previous blog posts, we will also update our application just to see how it works. Update the index.html file. I added `version 2` to the tagline under the title. Then, sync to your Github repository.

```bash
~/Flask-Basic-CICD-Heroku ❯ git add .
~/Flask-Basic-CICD-Heroku ❯ git commit -m "Update application"
[main 034cf0e] Update application
 1 file changed, 1 insertion(+), 1 deletion(-)
~/Flask-Basic-CICD-Heroku ❯ git push origin main
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

Also here (obviously), you will notice that a build is in progress

![flask-basic-github-heroku](/images/2021-11-07-10.png)

And if all went well you should see the updated application.

![flask-basic-github-heroku](/images/2021-11-07-11.png)

Hope you enjoyed this post. As always, code can be found [here](https://github.com/wiwa1978/blog-hugo-netlify-code/tree/main/Flask/Flask-Basic-CICD-Heroku)
