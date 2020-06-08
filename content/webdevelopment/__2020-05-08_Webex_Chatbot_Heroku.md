---
title: Webex Teams Chatbot with Python
date: 2020-05-08T17:19:50+01:00
draft: true
categories:
  - Web Development
  - Programming
tags:
  - Webex Teams
  - Python
  - All
---
### Introduction


### Install Heroku
```
wauterw@WAUTERW-M-65P7 Part3 % brew tap heroku/brew && brew install heroku
```

``` 
wauterw@WAUTERW-M-65P7 Part3 % heroku login
heroku: Press any key to open up the browser to login or q to exit: 
Opening browser to https://cli-auth.heroku.com/auth/cli/browser/fb003f16-3230-4cf2-b766-9e153d5aec4f
Logging in... done
Logged in as w********@gmail.com
```

```
wauterw@WAUTERW-M-65P7 Part3 % git init
Initialized existing Git repository in /Users/wauterw/Dropbox/Programming/ttg-miniproject/Part3/.git/
wauterw@WAUTERW-M-65P7 Part3 % heroku git:remote -a webexteams-messagebot
set git remote heroku to https://git.heroku.com/webexteams-messagebot.git
```

```
wauterw@WAUTERW-M-65P7 Part3 % git add .
wauterw@WAUTERW-M-65P7 Part3 % git commit -m "Initial commit"
wauterw@WAUTERW-M-65P7 Part3 % git push heroku master
Enumerating objects: 7, done.
Counting objects: 100% (7/7), done.
Delta compression using up to 12 threads
Compressing objects: 100% (6/6), done.
Writing objects: 100% (7/7), 1.36 KiB | 1.36 MiB/s, done.
Total 7 (delta 1), reused 0 (delta 0)
remote: Compressing source files... done.
remote: Building source:
remote: 
remote: -----> Python app detected
remote: -----> Installing python-3.6.10
remote: -----> Installing pip
***Truncated***
remote: -----> Discovering process types
remote:        Procfile declares types -> web
remote: 
remote: -----> Compressing...
remote:        Done: 50.7M
remote: -----> Launching...
remote:        Released v3
remote:        https://webexteams-messagebot.herokuapp.com/ deployed to Heroku
remote: 
remote: Verifying deploy... done.
To https://git.heroku.com/webexteams-messagebot.git
 * [new branch]      master -> master
```


Use `https://webexteams-messagebot.herokuapp.com/` in POSTMAN
