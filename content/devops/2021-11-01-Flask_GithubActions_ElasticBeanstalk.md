---
title: Deploy Flask App to Beanstalk using Github Actions
date: 2021-10-01T10:19:50+01:00
draft: false
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

In the previous [post](https://blog.wimwauters.com/devops/2021-10-29-flask_elasticbeanstalk/), we deployed a very basic Flask application to AWS Elastic Beanstalk using the CLI. This is likely not the way you would want to deploy applications. So I've been messing around with Github Actions a bit. Find below how to deploy the same application using a CICD pipeline.

### Running the application locally

First off, in case you did not follow along with the previous post, let's verify if our app works locally. Next, clone the code ([here](https://github.com/wiwa1978/blog-hugo-netlify-code/tree/main/Flask/Flask-Basic-Beanstalk)). Then create a Python virtual environment and install the requirements as follows:

```bash
~/Flask/Flask-Basic-CICD-GithubActions-Beanstalk ❯ python3 -m venv venv
~/Flask/Flask-Basic-CICD-GithubActions-Beanstalk ❯ source venv/bin/activate
```

Next, run the application using `gunicorn`.

```bash
(venv) ~/Flask/Flask-Basic-CICD-GithubActions-Beanstalk ❯ gunicorn --bind 0.0.0.0:5000 wsgi:application -w 1
[2021-10-27 16:19:19 +0200] [47651] [INFO] Starting gunicorn 20.0.4
[2021-10-27 16:19:19 +0200] [47651] [INFO] Listening at: http://0.0.0.0:5000 (47651)
[2021-10-27 16:19:19 +0200] [47651] [INFO] Using worker: sync
[2021-10-27 16:19:19 +0200] [47653] [INFO] Booting worker with pid: 47653
```

I hope you'll trust me that our app is running just fine locally.

### Uploading to Github

Next, let's dive into Github a bit. First action is to create a Github repository for our application. In my case, it's called `flask-basic-cicd-github-beanstalk`.

![flask-basic-beanstalk](/images/2021-11-01-1.png)

Once the repo has been created, you will receive some instructions to configure you're local git repository. In the directory where you cloned the application, issue the following commands (note: since you cloned there will already be a .git folder, so you might want to remove that one first by issuing a `rm -rfv .git` command).

![flask-basic-beanstalk](/images/2021-11-01-2.png)

```bash
(venv)  ~/Flask/Flask-Basic-CICD-GithubActions-Beanstalk ❯ git init

hint: Using 'master' as the name for the initial branch. This default branch name
hint: is subject to change. To configure the initial branch name to use in all
hint: of your new repositories, which will suppress this warning, call:
hint:
hint:   git config --global init.defaultBranch <name>
hint:
hint: Names commonly chosen instead of 'master' are 'main', 'trunk' and
hint: 'development'. The just-created branch can be renamed via this command:
hint:
hint:   git branch -m <name>
Initialized empty Git repository in ...Flask/Flask-Basic-CICD-GithubActions-Beanstalk/.git/
(venv) ~/Flask/Flask-Basic-CICD-GithubActions-Beanstalk ❯ git branch -m main
(venv) ~/Flask/Flask-Basic-CICD-GithubActions-Beanstalk ❯ git add .
(venv) ~/Flask/Flask-Basic-CICD-GithubActions-Beanstalk ❯ git commit -m "Initial commit"
[main (root-commit) cee8310] Initial commit
 6 files changed, 65 insertions(+)
 create mode 100644 .gitignore
 create mode 100644 application.py
 create mode 100644 requirements.txt
 create mode 100644 templates/index.html
 create mode 100644 test_application.py
 create mode 100644 wsgi.py
(venv) ~/Flask/Flask-Basic-CICD-GithubActions-Beanstalk ❯ git remote add origin https://github.com/wiwa1978/flask-basic-cicd-github-beanstalk.git
(venv) ~/Flask/Flask-Basic-CICD-GithubActions-Beanstalk ❯ git push origin main
Enumerating objects: 9, done.
Counting objects: 100% (9/9), done.
Delta compression using up to 12 threads
Compressing objects: 100% (8/8), done.
Writing objects: 100% (9/9), 1.47 KiB | 500.00 KiB/s, done.
Total 9 (delta 0), reused 0 (delta 0), pack-reused 0
To https://github.com/wiwa1978/flask-basic-cicd-github-beanstalk.git
 * [new branch]      main -> main
```

If all went well, you will see that your local application folder is now also available in the remote repo on Github

![flask-basic-beanstalk](/images/2021-11-01-3.png)

### Github Actions

As a next step, let's tackle the Github Actions part. You should see the screen below:

![flask-basic-beanstalk](/images/2021-11-01-4.png)

The idea is that Github Actions will look for a certain file (main.yml) in a particular folder (.github/workflows) in your repo. When it finds such a file, it will automatically go ahead and execute the actions that are described in that file. So how to get that `main.yml` file?

Click on setup a workflow yourself. Next you will see a proposal `main.yml` file from Github Actions. For now, it doesn't really matter what is in that file. Go ahead and commit that file (using the Github webinterface) to the main repository. You will see that this file is being put in a `.github/workflows` folder.

![flask-basic-beanstalk](/images/2021-11-01-5.png)

Right now, your remote git repository has a folder `.github/worklflows` that is not available in your local git repository. So as a next step, you need to pull the latest changes from Github to your local git repository. One way of doing this is as follows:

```bash
(venv) ~/Flask/Flask-Basic-CICD-GithubActions-Beanstalk ❯ git pull origin main
remote: Enumerating objects: 6, done.
remote: Counting objects: 100% (6/6), done.
remote: Compressing objects: 100% (3/3), done.
remote: Total 5 (delta 1), reused 0 (delta 0), pack-reused 0
Unpacking objects: 100% (5/5), 1.25 KiB | 213.00 KiB/s, done.
From https://github.com/wiwa1978/flask-basic-cicd-github-beanstalk
 * branch            main       -> FETCH_HEAD
   cee8310..0d2e0a9  main       -> origin/main
Updating cee8310..0d2e0a9
Fast-forward
 .github/workflows/main.yml | 36 ++++++++++++++++++++++++++++++++++++
 1 file changed, 36 insertions(+)
 create mode 100644 .github/workflows/main.yml
```

Now the file is available locally and we can edit it.

### Changing the Github actions workflow

As this was just a generic `main.yml` file proposed by Github, we need to tweak it to do exactly that what we want it to do (which is deploying the application to Beanstalk of course).
To start off with, we will checkout the application in a certain workspace. Then we install the dependencies and test the application. You'll surely notice that after each `RUN` command, we just provide a list of commands we want to execute.

```yml
# This is a basic workflow to help you get started with Actions

name: CI

# Controls when the workflow will run
on:
  # Triggers the workflow on push or pull request events but only for the main branch
  push:
    branches: [main]

  # Allows you to run this workflow manually from the Actions tab
  workflow_dispatch:

# A workflow run is made up of one or more jobs that can run sequentially or in parallel
jobs:
  # This workflow contains a single job called "build"
  build:
    # The type of runner that the job will run on
    runs-on: ubuntu-latest

    # Steps represent a sequence of tasks that will be executed as part of the job
    steps:
      # Checks-out your repository under $GITHUB_WORKSPACE, so your job can access it
      - uses: actions/checkout@v2

        # Set up Python 3.6 environment
      - name: Set up Python 3.9
        uses: actions/setup-python@v1
        with:
          python-version: "3.9"

      # Install dependencies
      - name: Install dependencies
        run: |
          python -m pip install --upgrade pip
          pip install -r requirements.txt
      # Run our unit tests using the test application
      - name: Run unit tests
        run: |
          python test_application.py
```

Next, as this file is now only available in our local repository, we need to upload it to Github:

```bash
(venv) ~/Flask/Flask-Basic-CICD-GithubActions-Beanstalk ❯ git add .
(venv) ~/Flask/Flask-Basic-CICD-GithubActions-Beanstalk ❯  git commit -m "Changing workflow file"
[main 8528605] Changing workflow file
 3 files changed, 14 insertions(+), 10 deletions(-)
 create mode 100644 __pycache__/application.cpython-39.pyc
 create mode 100644 __pycache__/wsgi.cpython-39.pyc
(venv) ~/Flask/Flask-Basic-CICD-GithubActions-Beanstalk ❯  git push origin main
Enumerating objects: 12, done.
Counting objects: 100% (12/12), done.
Delta compression using up to 12 threads
Compressing objects: 100% (6/6), done.
Writing objects: 100% (8/8), 1.33 KiB | 1.33 MiB/s, done.
Total 8 (delta 2), reused 0 (delta 0), pack-reused 0
remote: Resolving deltas: 100% (2/2), completed with 2 local objects.
To https://github.com/wiwa1978/flask-basic-cicd-github-beanstalk.git
   0d2e0a9..8528605  main -> main
```

As we mentioned before, once Github detects this file, it will start automatically the Github workflow. After a while, if all went well, you will see that the workflow got executed successfully.

![flask-basic-beanstalk](/images/2021-11-01-6.png)

If you click on the workflow you will be able to see some details, exactly the steps that we defined in the workflow file.

![flask-basic-beanstalk](/images/2021-11-01-7.png)

### Adding AWS secrets to Github

It's about time now to focus on the deployment of the application to Elastic Beanstalk. We need to supply Github with our AWS credentials. Obviously it's a bad idea to hardcode them into our application or into the Github workflow file. The proper way to achieve this is to create secret variables in Github itself. Go to settings and secrets and create the variables (see screenshot for the variable names).

![flask-basic-beanstalk](/images/2021-11-01-8.png)

### Deploy the application

Next, we will deploy the application. To do so, we can modify the `main.yml` file by providing the commands that we want to execute. In below file you'll notice we start off with an ubuntu image (running as a container). Obviously that image does not have the Elastic Beanstalk CLI installed by default so we start off with the installation of the CLI.

Next, we read out the AWS environment variables so they are available within our container. And then finally, we issue the EBS commands to deploy the application.

```yml
deploy:
  # Only run this job if "build" has ended successfully
  needs:
    - build

  runs-on: ubuntu-latest

  steps:
    # Checks-out your repository under $GITHUB_WORKSPACE
    - uses: actions/checkout@v2

    # Set up Python 3.6 environment
    - name: Set up Python 3.9
      uses: actions/setup-python@v1
      with:
        python-version: "3.9"

    # Elastic Beanstalk CLI version
    - name: Get EB CLI version
      run: |
        python -m pip install --upgrade pip
        pip install awsebcli --upgrade
        eb --version

        # Configure AWS Credentials

    # Configure AWS Credentials
    - name: Configure AWS Credentials
      uses: aws-actions/configure-aws-credentials@v1
      with:
        aws-access-key-id: ${{ secrets.AWS_ACCESS_KEY_ID }}
        aws-secret-access-key: ${{ secrets.AWS_SECRET_ACCESS_KEY }}
        aws-region: eu-central-1

    # Create the Elastic Beanstalk application
    - name: Create EBS application
      run: |
        eb init -p python-3.8 Flask_Basic --region eu-central-1

    # Deploy to (or Create) the Elastic Beanstalk environment
    - name: Create test environment & deploy
      run: |
        (eb use test-environment && eb status test-environment && eb deploy) || eb create test-environment
```

Once our workflow file is ready, add it to the Github repository as follows:

```bash
(venv) ~/Flask/Flask-Basic-CICD-GithubActions-Beanstalk ❯ git add .
(venv) ~/Flask/Flask-Basic-CICD-GithubActions-Beanstalk ❯ git commit -m "Adding deploy step"
[main 2e8921c] Adding deploy step
 1 file changed, 44 insertions(+), 1 deletion(-)
(venv) ~/Flask/Flask-Basic-CICD-GithubActions-Beanstalk ❯ git push origin main
Enumerating objects: 9, done.
Counting objects: 100% (9/9), done.
Delta compression using up to 12 threads
Compressing objects: 100% (3/3), done.
Writing objects: 100% (5/5), 1.27 KiB | 1.27 MiB/s, done.
Total 5 (delta 1), reused 0 (delta 0), pack-reused 0
remote: Resolving deltas: 100% (1/1), completed with 1 local object.
To https://github.com/wiwa1978/flask-basic-cicd-github-beanstalk.git
   8528605..2e8921c  main -> main
```

Again, you will see that Github automatically started executing the workflow:

![flask-basic-beanstalk](/images/2021-11-01-9.png)

Click on the workflow run to see more details:

![flask-basic-beanstalk](/images/2021-11-01-10.png)

If all goes well, after some time you will see a green indication that everything went fine:

![flask-basic-beanstalk](/images/2021-11-01-11.png)

Our application is now deployed onto Elastic Beanstalk. But where to find it, what URL has been created for us?

You will find that in the details of the `Create test environment & deploy` step. In my case, it appears to be: http://test-environment.eba-mg8ypbdj.eu-central-1.elasticbeanstalk.com/.
See also the screenshot below. BTW: clicking on this URL will not work as I deleted the Beanstalk environment after creating this blog post).

![flask-basic-beanstalk](/images/2021-11-01-12.png)

Visiting that link results in our application being shown:

![flask-basic-beanstalk](/images/2021-11-01-13.png)

### Updating the application

As we did in previous posts, we will update our application. In my case, I updated the index.html file by adding the words `version 2` to the tagline text under the title. As we made a change to our application, we need to sync it to our remote Git repo.

```bash
(venv) ~/Flask/Flask-Basic-CICD-GithubActions-Beanstalk ❯ git add .
(venv) ~/Flask/Flask-Basic-CICD-GithubActions-Beanstalk ❯ git commit -m "Changing index.html"
[main 1f96b16] Changing index.html
 1 file changed, 1 insertion(+), 1 deletion(-)
(venv) ~/Flask/Flask-Basic-CICD-GithubActions-Beanstalk ❯ git push origin main
Enumerating objects: 7, done.
Counting objects: 100% (7/7), done.
Delta compression using up to 12 threads
Compressing objects: 100% (3/3), done.
Writing objects: 100% (4/4), 351 bytes | 351.00 KiB/s, done.
Total 4 (delta 2), reused 0 (delta 0), pack-reused 0
remote: Resolving deltas: 100% (2/2), completed with 2 local objects.
To https://github.com/wiwa1978/flask-basic-cicd-github-beanstalk.git
   2e8921c..1f96b16  main -> main
```

Again, Github will immediately kick of a workflow run. After a while it should be successful.

![flask-basic-beanstalk](/images/2021-11-01-14.png)

And as expected, the application is updated:

![flask-basic-beanstalk](/images/2021-11-01-15.png)

Code can be found [here](https://github.com/wiwa1978/blog-hugo-netlify-code/tree/main/Flask/Flask-Basic-CICD-GithubActions-Beanstalk)
