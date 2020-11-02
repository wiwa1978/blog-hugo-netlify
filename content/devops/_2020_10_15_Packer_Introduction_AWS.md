---
title: Packer Introduction with AWS
date: 2020-10-09T14:39:50+01:00
draft: True
categories:
  - DevOps
  - All
tags:
  - Packer
  - AWS
---
### Introduction

### Simple
```
~/SynologyDrive/Programming/Packer/ brew tap hashicorp/tap
~/SynologyDrive/Programming/Packer/ brew install hashicorp/tap/packer
~/SynologyDrive/Programming/Packer/ packer
                                                                                                                                                                        
Usage: packer [--version] [--help] <command> [<args>]

Available commands are:
    build           build image(s) from template
    console         creates a console for testing variable interpolation
    fix             fixes templates from old versions of packer
    hcl2_upgrade    transform a JSON template into a HCL2 configuration
    inspect         see components of a template
    validate        check that a template is valid
    version         Prints the Packer version
```


```
{
    "variables": {
      "aws_access_key": "",
      "aws_secret_key": ""
    },
    "builders": [
      {
        "type": "amazon-ebs",
        "access_key": "{{user `aws_access_key`}}",
        "secret_key": "{{user `aws_secret_key`}}",
        "region": "eu-west-1",
        "source_ami_filter": {
          "filters": {
            "virtualization-type": "hvm",
            "name": "ubuntu/images/*ubuntu-xenial-16.04-amd64-server-*",
            "root-device-type": "ebs"
          },
          "owners": ["099720109477"],
          "most_recent": true
        },
        "instance_type": "t2.micro",
        "ssh_username": "ubuntu",
        "ami_name": "packer-example {{timestamp}}",
        "subnet_id": "subnet-f9b317a3"
      }
    ]
  }

```


```
~/SynologyDrive/Programming/Packer/AWS ❯ packer build \                                                                                                               
-var 'aws_access_key=AKI*****5MQ' \
-var 'aws_secret_key=lO/*****IVZ' \
aws.json

amazon-ebs: output will be in this color.

==> amazon-ebs: Prevalidating any provided VPC information
==> amazon-ebs: Prevalidating AMI Name: packer-example 1602591781
    amazon-ebs: Found Image ID: ami-08f3064d8481f3782
==> amazon-ebs: Creating temporary keypair: packer_5f859c25-bd93-59da-a5cd-bd3c8da97c7a
==> amazon-ebs: Creating temporary security group for this instance: packer_5f859c2d-99a7-7362-064b-6643b7e3d45e
==> amazon-ebs: Authorizing access to port 22 from [0.0.0.0/0] in the temporary security groups...
==> amazon-ebs: Launching a source AWS instance...
==> amazon-ebs: Adding tags to source instance
    amazon-ebs: Adding tag: "Name": "Packer Builder"
    amazon-ebs: Instance ID: i-0f6b8a3701a419d52
==> amazon-ebs: Waiting for instance (i-0f6b8a3701a419d52) to become ready...
==> amazon-ebs: Using ssh communicator to connect: 34.251.239.201
==> amazon-ebs: Waiting for SSH to become available...
==> amazon-ebs: Connected to SSH!
==> amazon-ebs: Stopping the source instance...
    amazon-ebs: Stopping instance
==> amazon-ebs: Waiting for the instance to stop...
==> amazon-ebs: Creating AMI packer-example 1602591781 from instance i-0f6b8a3701a419d52
    amazon-ebs: AMI: ami-0e57a48e91e049cf4
==> amazon-ebs: Waiting for AMI to become ready...
==> amazon-ebs: Terminating the source AWS instance...
==> amazon-ebs: Cleaning up any extra volumes...
==> amazon-ebs: No volumes to clean up, skipping
==> amazon-ebs: Deleting temporary security group...
==> amazon-ebs: Deleting temporary keypair...
Build 'amazon-ebs' finished after 3 minutes 459 milliseconds.

==> Wait completed after 3 minutes 459 milliseconds

==> Builds finished. The artifacts of successful builds are:
--> amazon-ebs: AMIs were created:
eu-west-1: ami-0e57a48e91e049cf4
```

![Packer](/images/2020-10-15-1.png)

![Packer](/images/2020-10-15-2.png)


![Packer](/images/2020-10-15-3.png)



### With provisioners

```
~/SynologyDrive/Programming/Packer/AWS/WithProvisioner ❯ cat helloworld.sh                                                                                           
#!/bin/bash
echo "Hello World"%   
```

```
~/SynologyDrive/Programming/Packer/AWS/WithProvisioner ❯ cat helloworld.txt                                                                                           
Hello World%  
```

```
~/SynologyDrive/Programming/Packer/AWS ❯ export AWS_ACCESS_KEY_ID=AKI***5MQ                                                                                16:23:58
~/SynologyDrive/Programming/Packer/AWS ❯ export AWS_SECRET_ACCESS_KEY=lO/***fXDIVZ    
```

```
{
    "variables": {
      "aws_access_key": "{{env `AWS_ACCESS_KEY_ID`}}",
      "aws_secret_key": "{{env `AWS_SECRET_ACCESS_KEY`}}",
    },
    "builders": [
      {
        "type": "amazon-ebs",
        "access_key": "{{user `aws_access_key`}}",
        "secret_key": "{{user `aws_secret_key`}}",
        "region": "eu-west-1",
        "source_ami_filter": {
          "filters": {
            "virtualization-type": "hvm",
            "name": "ubuntu/images/*ubuntu-xenial-16.04-amd64-server-*",
            "root-device-type": "ebs"
          },
          "owners": ["099720109477"],
          "most_recent": true
        },
        "instance_type": "t2.micro",
        "ssh_username": "ubuntu",
        "ami_name": "packer-provisioner-{{timestamp}}",
        "subnet_id": "subnet-f9b317a3"
      }
    ],
    "provisioners": [
        {
          "type": "file",
          "source": "./helloworld.txt",
          "destination": "/home/ubuntu/"
        },
        {
          "type": "shell",
          "inline": ["ls -al /home/ubuntu", "cat /home/ubuntu/helloworld.txt"]
        },
        {
          "type": "shell",
          "script": "./helloworld.sh"
        }
      ]
  }


```

```
~/SynologyDrive/Programming/Packer/AWS/WithProvisioner ❯ packer build aws.json                                                                                        16:32:41
amazon-ebs: output will be in this color.

==> amazon-ebs: Prevalidating any provided VPC information
==> amazon-ebs: Prevalidating AMI Name: packer-provisioner-1602599590
    amazon-ebs: Found Image ID: ami-08f3064d8481f3782
==> amazon-ebs: Creating temporary keypair: packer_5f85baa7-efc0-1aff-836a-8df85c93606d
==> amazon-ebs: Creating temporary security group for this instance: packer_5f85baaa-a3d5-4574-f7b5-f1e77afc025f
==> amazon-ebs: Authorizing access to port 22 from [0.0.0.0/0] in the temporary security groups...
==> amazon-ebs: Launching a source AWS instance...
==> amazon-ebs: Adding tags to source instance
    amazon-ebs: Adding tag: "Name": "Packer Builder"
    amazon-ebs: Instance ID: i-04a3f9f14adbc2276
==> amazon-ebs: Waiting for instance (i-04a3f9f14adbc2276) to become ready...
==> amazon-ebs: Using ssh communicator to connect: 34.249.118.210
==> amazon-ebs: Waiting for SSH to become available...
==> amazon-ebs: Connected to SSH!
==> amazon-ebs: Uploading ./helloworld.txt => /home/ubuntu/
    amazon-ebs: helloworld.txt 11 B / 11 B [=================================================================================================================] 100.00% 0s
==> amazon-ebs: Provisioning with shell script: /var/folders/27/hzk_ld5j6gb516mtp077gv600000gq/T/packer-shell646929318
    amazon-ebs: total 32
    amazon-ebs: drwxr-xr-x 4 ubuntu ubuntu 4096 Oct 13 14:33 .
    amazon-ebs: drwxr-xr-x 3 root   root   4096 Oct 13 14:33 ..
    amazon-ebs: -rw-r--r-- 1 ubuntu ubuntu  220 Aug 31  2015 .bash_logout
    amazon-ebs: -rw-r--r-- 1 ubuntu ubuntu 3771 Aug 31  2015 .bashrc
    amazon-ebs: drwx------ 2 ubuntu ubuntu 4096 Oct 13 14:33 .cache
    amazon-ebs: -rw-r--r-- 1 ubuntu ubuntu  655 Jul 12  2019 .profile
    amazon-ebs: drwx------ 2 ubuntu ubuntu 4096 Oct 13 14:33 .ssh
    amazon-ebs: -rw-r--r-- 1 ubuntu ubuntu   11 Oct 13 14:33 helloworld.txt
    amazon-ebs: Hello World
==> amazon-ebs: Provisioning with shell script: ./helloworld.sh
    amazon-ebs: Hello World
==> amazon-ebs: Stopping the source instance...
    amazon-ebs: Stopping instance
==> amazon-ebs: Waiting for the instance to stop...
==> amazon-ebs: Creating AMI packer-provisioner-1602599590 from instance i-04a3f9f14adbc2276
    amazon-ebs: AMI: ami-040c8f277953c9bba
==> amazon-ebs: Waiting for AMI to become ready...
==> amazon-ebs: Terminating the source AWS instance...
==> amazon-ebs: Cleaning up any extra volumes...
==> amazon-ebs: No volumes to clean up, skipping
==> amazon-ebs: Deleting temporary security group...
==> amazon-ebs: Deleting temporary keypair...
Build 'amazon-ebs' finished after 2 minutes 23 seconds.

==> Wait completed after 2 minutes 23 seconds

==> Builds finished. The artifacts of successful builds are:
--> amazon-ebs: AMIs were created:
eu-west-1: ami-040c8f277953c9bba                                                                              
```

![Packer](/images/2020-10-15-4.png)

![Packer](/images/2020-10-15-5.png)


![Packer](/images/2020-10-15-6.png)

### Provision machine using our AMI

```bash
~/SynologyDrive/Programming/Packer/AWS/WithProvisioner ❯ ssh -i ~/.ssh/AWS-Cisco.pem ubuntu@52.211.233.168 
Welcome to Ubuntu 16.04.7 LTS (GNU/Linux 4.4.0-1114-aws x86_64)

 * Documentation:  https://help.ubuntu.com
 * Management:     https://landscape.canonical.com
 * Support:        https://ubuntu.com/advantage

0 packages can be updated.
0 updates are security updates.

New release '18.04.5 LTS' available.
Run 'do-release-upgrade' to upgrade to it.


Last login: Tue Oct 13 14:41:20 2020 from 173.38.220.58
To run a command as administrator (user "root"), use "sudo <command>".
See "man sudo_root" for details.

ubuntu@ip-172-31-39-229:~$ ls -al
total 36
drwxr-xr-x 4 ubuntu ubuntu 4096 Oct 13 14:41 .
drwxr-xr-x 3 root   root   4096 Oct 13 14:33 ..
-rw------- 1 ubuntu ubuntu   28 Oct 13 14:41 .bash_history
-rw-r--r-- 1 ubuntu ubuntu  220 Aug 31  2015 .bash_logout
-rw-r--r-- 1 ubuntu ubuntu 3771 Aug 31  2015 .bashrc
drwx------ 2 ubuntu ubuntu 4096 Oct 13 14:33 .cache
-rw-r--r-- 1 ubuntu ubuntu   11 Oct 13 14:33 helloworld.txt
-rw-r--r-- 1 ubuntu ubuntu  655 Jul 12  2019 .profile
drwx------ 2 ubuntu ubuntu 4096 Oct 13 14:33 .ssh   
```

![Packer](/images/2020-10-15-7.png)

![Packer](/images/2020-10-15-8.png)

![Packer](/images/2020-10-15-9.png)