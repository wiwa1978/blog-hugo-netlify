---
title: Gitlab
date: 2020-10-01T14:39:50+01:00
draft: True
categories:
  - DevOps
  - All
tags:
  - Terraform
  - AWS
---

### Introduction

### Installation Gitlab
```
cisco@wauterw-gitlab:~$ sudo apt update
cisco@wauterw-gitlab:~$ sudo apt-get install -y curl openssh-server ca-certificates tzdata
cisco@wauterw-gitlab:~$ cd /tmp
cisco@wauterw-gitlab:~$ curl -LO https://packages.gitlab.com/install/repositories/gitlab/gitlab-ce/script.deb.sh
cisco@wauterw-gitlab:~$ sudo bash /tmp/script.deb.sh
cisco@wauterw-gitlab:~$ sudo apt install gitlab-ce
```

Change EXTERNAL_URL
```
cisco@wauterw-gitlab:~$ sudo nano /etc/gitlab/gitlab.rb
```
Change the external URL and save the file
```
cisco@wauterw-gitlab:~$ sudo gitlab-ctl reconfigure
Starting Chef Infra Client, version 15.12.22
resolving cookbooks for run list: ["gitlab"]
<TRUNCATED>

```

Provide your root password

![webex](/images/gitlab-0.png)

![webex](/images/gitlab-0-a.png)

![webex](/images/gitlab-0-b.png)

### Configure Gitlab runners

![webex](/images/gitlab-runner-0.png)

![webex](/images/gitlab-runner-1.png)


##### Install Gitlab runner

```
cisco@wauterw-gitlab:~$ curl -LO https://packages.gitlab.com/install/repositories/runner/gitlab-runner/script.deb.sh
  % Total    % Received % Xferd  Average Speed   Time    Time     Time  Current
                                 Dload  Upload   Total   Spent    Left  Speed
100  5945  100  5945    0     0   8517      0 --:--:-- --:--:-- --:--:--  8505
cisco@wauterw-gitlab:~$ ls
script.deb.sh
cisco@wauterw-gitlab:~$ sudo bash script.deb.sh
<TRUNCATED>
The repository is setup! You can now install packages.
cisco@wauterw-gitlab:~$ sudo apt-get install gitlab-runner=13.4.1
```

##### Register Gitlab runner

```
cisco@wauterw-gitlab-runner:~$ sudo gitlab-runner register
Runtime platform                                    arch=amd64 os=linux pid=15208 revision=e95f89a0 version=13.4.1
Running in system-mode.

Please enter the gitlab-ci coordinator URL (e.g. https://gitlab.com/):
http://10.61.124.189/
Please enter the gitlab-ci token for this runner:
5zRFzmL-Sd2CEFV14osd
Please enter the gitlab-ci description for this runner:
[wauterw-gitlab-runner]:
Please enter the gitlab-ci tags for this runner (comma separated):

Registering runner... succeeded                     runner=5zRFzmL-
Please enter the executor: docker-ssh, parallels, ssh, virtualbox, docker+machine, kubernetes, custom, shell, docker-ssh+machine, docker:
shell
Runner registered successfully. Feel free to start it, but if it's running already the config should be automatically reloaded!
```



### Installation Jenkins

```
root@wauterw-jenkins:/home/cisco# sudo apt install default-jre
root@wauterw-jenkins:/home/cisco# wget -qO - https://pkg.jenkins.io/debian-stable/jenkins.io.key | apt-key add -
OK
cisco@wauterw-jenkins:~$ sudo sh -c 'echo deb https://pkg.jenkins.io/debian-stable binary/ > /etc/apt/sources.list.d/jenkins.list'
cisco@wauterw-jenkins:~$ sudo apt update
cisco@wauterw-jenkins:~$ sudo apt install jenkins
```

###### Installing Ansible

```
cisco@wauterw-jenkins:~$ sudo apt-add-repository ppa:ansible/ansible
cisco@wauterw-jenkins:~$ sudo apt update
cisco@wauterw-jenkins:~$ sudo apt install ansible
```

###### Installing Terraform

```
cisco@wauterw-jenkins:~$ sudo apt-get install zip -y
cisco@wauterw-jenkins:~$ wget https://releases.hashicorp.com/terraform/0.12.24/terraform_0.12.24_linux_amd64.zip
cisco@wauterw-jenkins:~$ unzip terraform*.zip
cisco@wauterw-jenkins:~$ sudo mv terraform /usr/local/bin
```

###### Install Docker
```
cisco@wauterw-jenkins:~$ sudo apt install apt-transport-https ca-certificates curl software-properties-common
cisco@wauterw-jenkins:~$ curl -fsSL https://download.docker.com/linux/ubuntu/gpg | sudo apt-key add -
cisco@wauterw-jenkins:~$ sudo add-apt-repository "deb [arch=amd64] https://download.docker.com/linux/ubuntu bionic stable"
cisco@wauterw-jenkins:~$ sudo apt update
cisco@wauterw-jenkins:~$ sudo apt install docker-ce
cisco@wauterw-jenkins:~$ sudo usermod -aG docker ${USER}
```

![webex](/images/jenkins-0.png)

![webex](/images/jenkins-0-a.png)

![webex](/images/jenkins-0-b.png)

![webex](/images/jenkins-0-c.png)

![webex](/images/jenkins-0-d.png)

![webex](/images/jenkins-0-e.png)

###### Manage Jenkins plugins

Install the Gitlab plugin

![webex](/images/jenkins-0-f.png)

Configure the Gitlab plugin

You will need to generate a Gitlab token (Jenkins is asking for that). To do so, login to the /admin and go to Settings > Access Tokens

![webex](/images/gitlab-0-c.png)

You need to give also api and read_api access
![webex](/images/gitlab-0-d.png)

![webex](/images/jenkins-0-g.png)


![webex](/images/jenkins-0-h.png)
### Configure Gitlab installation

![webex](/images/gitlab-1.png)

Login
![webex](/images/gitlab-2.png)

Create  project

![webex](/images/gitlab-3.png)

Create new Project

![webex](/images/gitlab-4.png)

![webex](/images/gitlab-5.png)

```bash
~/S/P/C/CICD_OneServer_Jenkins_Ansible_A/ThreePipelines/ACIPipeline_AMS ❯ git init  
Initialized empty Git repository in /Users/wauterw/SynologyDrive/Programming/CICD/CICD_OneServer_Jenkins_Ansible_Amsterdam/ThreePipelines/ACIPipeline_AMS/.git/
~/S/P/C/CICD_OneServer_Jenkins_Ansible_A/T/ACIPipeline_AMS master ?7 ❯ git remote add origin http://10.61.124.189/cisco/pipeline-aci-team.git
~/S/P/C/CICD_OneServer_Jenkins_Ansible_A/T/ACIPipeline_AMS master ?7 ❯ git add . 
~/S/P/C/CICD_OneServer_Jenkins_Ansible_A/T/ACIPipeline_AMS master +8 ❯ git commit -m "Initial commit" 

[master (root-commit) 449b11f] Initial commit
 8 files changed, 668 insertions(+)
 create mode 100755 Jenkinsfile
 create mode 100755 Tests/aci.robot
 create mode 100755 Tests/aci_tests_testbed.yaml
 create mode 100755 hosts_aci.yml
 create mode 100755 parse.py
 create mode 100755 playbook_aci_create.yml
 create mode 100755 playbook_aci_delete.yml
 create mode 100755 vars/variables_aci.yml
~/S/P/C/CICD_OneServer_Jenkins_Ansible_A/T/ACIPipeline_AMS master ❯ git push -u origin master 
Enumerating objects: 12, done.
Counting objects: 100% (12/12), done.
Delta compression using up to 12 threads
Compressing objects: 100% (11/11), done.
Writing objects: 100% (12/12), 6.14 KiB | 3.07 MiB/s, done.
Total 12 (delta 0), reused 0 (delta 0)
To http://10.61.124.189/cisco/pipeline-aci-team.git
 * [new branch]      master -> master
Branch 'master' set up to track remote branch 'master' from 'origin'.
```
![webex](/images/gitlab-6.png)


### Configure Jenkins installation
![webex](/images/jenkins-1.png)


![webex](/images/jenkins-2.png)


### Integrate Jenkins and Gitlab

![webex](/images/jenkins-3.png)



Test
![webex](/images/jenkins-4.png)

![webex](/images/jenkins-5.png)

Configure on Gitlab side

![webex](/images/gitlab-8.png)

![webex](/images/gitlab-9.png)

Note: in case you receive `Url is blocked: Requests to the local network are not allowed`, then go to /admin/application_settings/network (login with root account) and do the following
![webex](/images/gitlab-9-a.png)

You will see that we get an error:

![webex](/images/jenkins-6.png)

We need to ensure that Jenkins can read our repository. Therefore we need to create a token in Gitlab

![webex](/images/gitlab-10.png)

![webex](/images/gitlab-11.png)


### Specific Jenkins library
![webex](/images/jenkins-7.png)

![webex](/images/jenkins-8.png)

![webex](/images/jenkins-9.png)





rob@ubuntu:~$ cat jenkins.sh
sudo docker run --detach \
    --publish 8080:8080 \
    --publish 50000:50000 \
    --name jenkins \
    --restart always \
    --volume jenkins_home:/var/jenkins_home \
    jenkins/jenkins:lts


  3daa2239ac2d        jenkins/jenkins:lts           "/sbin/tini -- /usr/…"   10 months ago       Up 2 months             0.0.0.0:8080->8080/tcp, 0.0.0.0:50000->50000/tcp               jenkins