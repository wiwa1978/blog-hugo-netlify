### Introduction

### Installation

##### Install JAVA

First we need to install Java:
```bash
cisco@wauterw-jenkins:~$ sudo apt install default-jre
```
Next, check the java version:
```
cisco@wauterw-jenkins:~$ java -version
openjdk version "11.0.8" 2020-07-14
OpenJDK Runtime Environment (build 11.0.8+10-post-Ubuntu-0ubuntu118.04.1)
OpenJDK 64-Bit Server VM (build 11.0.8+10-post-Ubuntu-0ubuntu118.04.1, mixed mode, sharing)
```

##### Install Jenkins

```
cisco@wauterw-jenkins:~$ wget -q -O - https://pkg.jenkins.io/debian-stable/jenkins.io.key | sudo apt-key add -
[sudo] password for cisco:
OK
cisco@wauterw-jenkins:~$ sudo sh -c 'echo deb https://pkg.jenkins.io/debian binary/ > \
>     /etc/apt/sources.list.d/jenkins.list'
cisco@wauterw-jenkins:~$ sudo apt update
cisco@wauterw-jenkins:~$ sudo apt install jenkins
```