---
title: Cisco Webex Teams - Deploy chatbot to Heroku with Flask
date: 2020-10-01T04:32:50+01:00
draft: true
categories:
  - Network Programming
  - Programming
  - All
tags:
  - Webex Teams
  - Python

---

### Introduction

### Running our code on Heroku

```
appdirs==1.4.3
certifi==2019.11.28
chardet==3.0.4
Click==7.0
filelock==3.0.12
Flask==1.1.1
idna==2.8
itsdangerous==1.1.0
Jinja2==2.11.1
MarkupSafe==1.1.1
requests==2.22.0
six==1.14.0
urllib3==1.25.8
virtualenv==20.0.0
Werkzeug==1.0.0
gunicorn==19.7.1
```


Procfile
```
web: gunicorn chatbot_aci:app
```

Login to Heroku
```bash
~/blog-hugo-netlify-code/WebexT/p/deckofcards_bot master !4 ?1 ❯ heroku login                                                                                      
heroku: Press any key to open up the browser to login or q to exit: 
Opening browser to https://cli-auth.heroku.com/auth/cli/browser/00ced68a-7f07-417b-9354-f516eec262ef?requestor=SFMyNTY.g3QAAAACZAAEZGF0YW0AAAAOOTQuMTA0LjExNC4xMjJkAAZzaWduZWRuBgCcbKDzdAE.Ih4xA9_RoNXtcLasJ6iYfspn5ehi1IdcZM0
Logging in... done
Logged in as wauters1978@gmail.com
```

Create Heroku app

```
~blog-hugo-netlify-code/WebexT/p/deckofcards_bot master !5 ?1 ❯ heroku create webexteams-deckofcards                                                              
Creating ⬢ webexteams-deckofcards... done
https://webexteams-deckofcards.herokuapp.com/ | https://git.heroku.com/webexteams-deckofcards.git
```

![webex](/images/2020-10-07-4.png)


Upload git

```
~blog-hugo-netlify-code/WebexT/p/deckofcards_bot master ❯ heroku git:remote -a webexteams-deckofcards                                                             14:42:54
set git remote heroku to https://git.heroku.com/webexteams-deckofcards.git
```
This is configure the remote. Let's check it:
```
~blog-hugo-netlify-code/WebexT/p/deckofcards_bot master ❯ git remote -v                                                                                           14:44:09
heroku  https://git.heroku.com/webexteams-deckofcards.git (fetch)
heroku  https://git.heroku.com/webexteams-deckofcards.git (push)
```


```bash
~blog-hugo-netlify-code/WebexT/p/deckofcards_bot master ❯ git push heroku master                                                                                  14:44:27
Enumerating objects: 10, done.
Counting objects: 100% (10/10), done.
Delta compression using up to 12 threads
Compressing objects: 100% (8/8), done.
Writing objects: 100% (10/10), 3.42 KiB | 3.42 MiB/s, done.
Total 10 (delta 0), reused 0 (delta 0)
remote: Compressing source files... done.
remote: Building source:
remote: 
remote: -----> Python app detected
<TRUNCATED>
remote: Verifying deploy... done.
To https://git.heroku.com/webexteams-deckofcards.git
 * [new branch]      master -> master

```

Create the webhook

![webex](/images/2020-10-07-1.png)

Test the application

https://webexteams-deckofcards.herokuapp.com/

![webex](/images/2020-10-07-2.png)


[Code](https://github.com/wiwa1978/blog-hugo-netlify-code/tree/master/WebexTeams_Chatbots/python_deckofcards/deckofcards_bot)