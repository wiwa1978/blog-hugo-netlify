---
title: VMware tools on ESXi Raspberry Pi
date: 2020-11-01T20:19:50+01:00
draft: false
categories:
  - Private Cloud
  - All
tags:
  - Raspberry Pi
  - vSphere
---

### Introduction

In [this](https://blog.wimwauters.com/devops/2020-10-20_esxi_raspberry/) post, I showed how to go about installing ESXi onto a Raspberry Pi 4 device. As I was launching multiple Ubuntu virtual servers, I found issues with installing VMWare tools onto them. I found [this](https://www.virten.net/2020/10/vmware-tools-for-ubuntu-20-04-lts-arm64-on-esxi-arm/) post and applied it to my setup. 

### Commands for Ubuntu 20.04 ARM

Here are the exact commands I have been using:

First update and upgrade your Ubuntu OS:
```
apt update -y && apt upgrade -y
```
Next, install the pre-requisites for compiling the Open VM tools. This took a while in my case.
```
apt install -y git automake make gobjc++ libtool pkg-config libmspack-dev libglib2.0-dev libpam0g-dev libssl-dev libxml2-dev libxmlsec1-dev libx11-dev libxext-dev libxinerama-dev libxi-dev libxrender-dev libxrandr-dev libxtst-dev libgdk-pixbuf2.0-dev libgtk-3-dev libgtkmm-3.0-dev
```
Next, clone the open-vm-tools Github [repository](https://github.com/vmware/open-vm-tools):

```
git clone https://github.com/vmware/open-vm-tools.git
cd open-vm-tools/open-vm-tools/
```
Next, we need to compile and install the tools:
```
autoreconf -i
./configure --disable-dependency-tracking
make
make install
ldconfig
```
Now we need to create a service file which is required to run vmtoolsd as service with systemd:
```
cat > /etc/systemd/system/vmtoolsd.service << EOF
[Unit]
Description=Service for virtual machines hosted on VMware
Documentation=http://github.com/vmware/open-vm-tools
After=network-online.target

[Service]
ExecStart=/usr/local/bin/vmtoolsd
Restart=always
TimeoutStopSec=5

[Install]
WantedBy=multi-user.target
EOF
```
And finally, we meed to enable that service we just created
```
systemctl enable vmtoolsd.service
systemctl start vmtoolsd.service
systemctl status vmtoolsd.service
```
If all went well, you should be seeing something along the lines of below output
```
● vmtoolsd.service - Service for virtual machines hosted on VMware
     Loaded: loaded (/etc/systemd/system/vmtoolsd.service; enabled; vendor preset: enabled)
     Active: active (running) since Wed 2020-11-11 20:56:01 UTC; 47ms ago
       Docs: http://github.com/vmware/open-vm-tools
   Main PID: 102012 (vmtoolsd)
      Tasks: 1 (limit: 2233)
     Memory: 680.0K
     CGroup: /system.slice/vmtoolsd.service
             └─102012 /usr/local/bin/vmtoolsd

Nov 01 20:56:01 ubuntu-microk8s-1 systemd[1]: Started Service for virtual machines hosted on VMware.
```
I created a Shell script that does exactly these steps but in an automated way. Saves you some time. Please find it [here](https://github.com/wiwa1978/blog-hugo-netlify-code/blob/master/ESXi_ARM_VMTools/install_openvm_tools.sh).