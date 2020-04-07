---
title: Create Cisco ACI network with Terraform and Gitlab CICD
date: 2019-11-29T14:39:50+01:00
draft: false
categories:
  - DevOps
  - Infrastructure As Code
  - CI / CD
  - Private Cloud
tags:
  - Terraform
  - ACI
  - Gitlab
---
### Introduction
As promised, we’ll configure an ACI network using a CI/CD pipeline. If you understood this post, you’ll easily grasp this one as well. In the end, whether you configure an ACI resource or an ACI resource with Gitlab, the same principles apply. So we’ll run fast over this one.

In this blog post, we will be using exactly the same Terraform files as we used in the previous post so I won’t spend time describing them here.

Note: I will use my own APIC in my lab. If you don't have one available, have a look [here](https://devnetsandbox.cisco.com/RM/Diagram/Index/5a229a7c-95d5-4cfd-a651-5ee9bc1b30e2?diagramType=Topology) from Cisco Devnet. This is an always-on sandbox environment which you can use if you want to follow along with this blog post.

### Gitlab configuration
In this post, we used the online version of Gitlab. For this post, we will be installing our own Gitlab server since I cannot access our internal lab from outside. Hence the need to install the Community Edition of Gitlab in my lab. I won’t go over the installation in this blog post. A pretty easy guide can be found here.

Anyway, after installation, continue to create a project. See the below screenshot:

![gitlab](/images/2019-11-29-1.png)

Once the project has been created, you will see a page with instructions how to add files to the repository. Just follow these instructions on your own laptop. 

So let’s go ahead and add the files to our repository. Just follow the commands below:
- git init
- git commit -m "Initial commit"
- git push -u origin master

```bash
WAUTERW-M-65P7:ACI__Terraform_Gitlab wauterw$ git init
Initialized empty Git repository in /Users/wauterw/Dropbox/Programming/blog-hugo-netlify-code/ACI__Terraform_Gitlab/.git/
WAUTERW-M-65P7:ACI__Terraform_Gitlab wauterw$ git remote add origin http://10.48.109.16/cisco/terraform-aci.git
WAUTERW-M-65P7:ACI__Terraform_Gitlab wauterw$ git add .
WAUTERW-M-65P7:ACI__Terraform_Gitlab wauterw$ git commit -m "Initial commit"
[master (root-commit) 9de6e78] Initial commit
 4 files changed, 53 insertions(+)
 create mode 100644 .gitignore
 create mode 100644 main_aci.tf
 create mode 100644 terraform.tf
 create mode 100644 variables_aci.tf
WAUTERW-M-65P7:ACI__Terraform_Gitlab wauterw$ git push -u origin master
Enumerating objects: 7, done.
Counting objects: 100% (7/7), done.
Delta compression using up to 12 threads
Compressing objects: 100% (6/6), done.
Writing objects: 100% (7/7), 1.09 KiB | 1.09 MiB/s, done.
Total 7 (delta 0), reused 0 (delta 0)
To http://10.48.109.16/cisco/terraform-aci.git
 * [new branch]      master -> master
Branch 'master' set up to track remote branch 'master' from 'origin'.
```
![gitlab](/images/2019-11-29-3.png)

### Gitlab CI/CD

As we learned in earlier posts, we need to create a ‘.gitlab-ci.yaml’ with the following content. It does define a number of stages, essentially to do a ‘terraform init’, ‘terraform plan’ and ‘terraform apply’.
```bash
image:
  name: hashicorp/terraform:light
  entrypoint:
    - '/usr/bin/env'
    - 'PATH=/usr/local/sbin:/usr/local/bin:/usr/sbin:/usr/bin:/sbin:/bin'

before_script:
  - rm -rf .terraform
  - terraform --version
  - export AWS_ACCESS_KEY_ID
  - export AWS_SECRET_ACCESS_KEY
  - terraform init

stages:
  - validate
  - plan
  - apply

validate:
  stage: validate
  script:
    - terraform validate

plan:
  stage: plan
  script:
    - terraform plan -out "planfile"
  dependencies:
    - validate
  artifacts:
    paths:
      - planfile

apply:
  stage: apply
  script:
    - terraform apply -input=false "planfile"
  dependencies:
    - plan
  when: manual
```
Save the .gitlab-ci.yaml into the same folder as the rest of your Terraform files and check this file in with Gitlab (just follow the 3 commands as mentioned earlier).

```bash
WAUTERW-M-65P7:ACI__Terraform_Gitlab wauterw$ git add .
WAUTERW-M-65P7:ACI__Terraform_Gitlab wauterw$ git commit -m "Adding CICD file"
[master 2d9fae2] Adding CICD file
 1 file changed, 27 insertions(+)
 create mode 100644 .gitlab-ci.yml
WAUTERW-M-65P7:ACI__Terraform_Gitlab wauterw$ git push origin master
Enumerating objects: 4, done.
Counting objects: 100% (4/4), done.
Delta compression using up to 12 threads
Compressing objects: 100% (3/3), done.
Writing objects: 100% (3/3), 521 bytes | 521.00 KiB/s, done.
Total 3 (delta 1), reused 0 (delta 0)
To http://10.48.109.16/cisco/terraform-aci.git
   5fc2d86..2d9fae2  master -> master
```

After this is done, your repo will look as follows:
![gitlab](/images/2019-11-29-4.png)

Once Gitlab detects that a .gitlab-ci.yml file is uploaded to your repo, it will automatically start the pipeline process.
![gitlab](/images/2019-11-29-5.png)

Note: before doing all this, note that in our .gitlab-ci.yml file we do export some AWS variables. Therefore you must declrare these in Gitlab. Go to Settings > CICD > Variables and insert your AWS values there.

![gitlab](/images/2019-11-29-11.png)

And as we put in the .gitlab-ci.yml file that we wanted to manually approve before applying the configuration, you will see that the pipeline stops after the two first steps of the pipeline. Waiting until you approve manually. Once approved you will see that all three steps are completed.
![gitlab](/images/2019-11-29-6.png)

And finally, you will also see the configuration on Cisco’s ACI platform.
![gitlab](/images/2019-11-29-7.png)

### Destroy ACI configuration
Let’s now use a pipeline to destroy the network constructs on ACI automatically.
```bash
image:
  name: hashicorp/terraform:light
  entrypoint:
    - '/usr/bin/env'
    - 'PATH=/usr/local/sbin:/usr/local/bin:/usr/sbin:/usr/bin:/sbin:/bin'

before_script:
  - rm -rf .terraform
  - terraform --version
  - export AWS_ACCESS_KEY_ID
  - export AWS_SECRET_ACCESS_KEY
  - terraform init

stages:
  - validate
  - destroy

validate:
  stage: validate
  script:
    - terraform validate

destroy:
  stage: destroy
  script:
    - terraform destroy -auto-approve
  when: manual
  ```
Add this file again to the Gitlab repo using the 3 GIT commands we used earlier. Note that Terraform remembers the state so it knows it has created a Tenant, VRF and BD. Hence, it will also remember what it needs to delete.

As you can see from below screenshot, we only have two steps now, the validate and destroy step (because this is what is mentioned in the .gitlab-ci file as such).
![gitlab](/images/2019-11-29-8.png)

Click on the trigger to destroy the network configuration on Cisco ACI. When all works well, you will see the below screen:
![gitlab](/images/2019-11-29-9.png)

And obviously, the tenant, VRF and BD will be removed from the ACI network.
![gitlab](/images/2019-11-29-10.png)

The files we used in this tutorial can be found on [Github](https://github.com/wiwa1978/blog-hugo-netlify-code/tree/master/ACI_Terraform_Gitlab).