---
title: Packer Introduction with DigitalOcean
date: 2020-10-10T14:39:50+01:00
draft: True
categories:
  - DevOps
  - All
tags:
  - Packer
  - AWS
---
### Introduction

###

```
#!/bin/bash
apt-get update -y
apt-get upgrade -y

apt-get install -y nginx

systemctl enable nginx
systemctl start nginx
```



```
{
    "variables": {
        "do_token": "{{env `do_token`}}"
    },
    "builders": [
      {
        "type": "digitalocean",
        "api_token": "{{user `do_token`}}",
        "image": "ubuntu-16-04-x64",
        "ssh_username": "root",
        "region": "ams3",
        "size": "512mb"
      }
    ],
    "provisioners": [
      {
        "type": "file",
        "source": "do.json",
        "destination": "/tmp"
      },
      {
        "type": "shell",
        "scripts": [
            "scripts/nginx.sh"
        ]
      }]
  }
```


![Packer](/images/2020-10-16-1.png)

![Packer](/images/2020-10-16-2.png)

![Packer](/images/2020-10-16-3.png)


```
~/SynologyDrive/Programming/Packer/DigitalOcean ❯ packer build do.json                                                                                                   17:51:34
digitalocean: output will be in this color.

==> digitalocean: Creating temporary ssh key for droplet...
==> digitalocean: Creating droplet...
==> digitalocean: Waiting for droplet to become active...
==> digitalocean: Using ssh communicator to connect: 188.166.77.118
==> digitalocean: Waiting for SSH to become available...
==> digitalocean: Connected to SSH!
==> digitalocean: Uploading do.json => /tmp/do_bu.json
    digitalocean: do.json 545 B / 545 B [==============================================================================================================================] 100.00% 0s
==> digitalocean: Provisioning with shell script: scripts/nginx.sh
    digitalocean: Get:1 http://security.ubuntu.com/ubuntu xenial-security InRelease [109 kB]
    digitalocean: Get:2 http://mirrors.digitalocean.com/ubuntu xenial InRelease [247 kB]
    digitalocean: Get:3 http://security.ubuntu.com/ubuntu xenial-security/main amd64 Packages [1,458 kB]
    digitalocean: Get:4 http://security.ubuntu.com/ubuntu xenial-security/main Translation-en [347 kB]

    <TRUNCATED>

==> digitalocean: Gracefully shutting down droplet...
==> digitalocean: Creating snapshot: packer-1602604297
==> digitalocean: Waiting for snapshot to complete...
==> digitalocean: Destroying droplet...
==> digitalocean: Deleting temporary ssh key...
Build 'digitalocean' finished after 2 minutes 53 seconds.

==> Wait completed after 2 minutes 53 seconds

==> Builds finished. The artifacts of successful builds are:
--> digitalocean: A snapshot was created: 'packer-1602604297' (ID: 71679548) in regions 'ams3'
```


![Packer](/images/2020-10-16-4.png)


![Packer](/images/2020-10-16-5.png)


![Packer](/images/2020-10-16-6.png)