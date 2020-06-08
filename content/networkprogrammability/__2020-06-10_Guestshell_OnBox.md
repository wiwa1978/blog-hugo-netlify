---
title: Exploring Guest Shell and Python On-box on IOS-XE
date: 2020-06-10T10:19:50+01:00
draft: false
categories:
  - Network Programming
  - Programming
  - All
tags:
  - Python
---
### What is Guest Shell?

Cisco IOS-XE devices (as of release 16.6.1) come with a guest shell, which is essentlially a virtualized Linux (CentOS) environment. It allows to run Linux applications on the network device. It comes prepopulated with common tools like net-tools, iproute, tcpdump and OpenSSH but it allows you also to install and update Linux applications (think of Puppet agents, Chef agents, Splunk agents...).  Also Python (at the moment version 2.7) is integrated in the Linux environment so the guest shell allows you to run Python scripts as well. The guest shell is intended for tools, Linux utilities and manageability rather than networking functions.

The guest shell container is managed through IOx, which is Cisco's Application Hosting Infrastructure for Cisco IOS XE devices. It provides application hosting capabilities for different application types. The guest shell (container deployment) is actually just one such application. The IOx facilitates the lifecycle management of applications, think of distribution, deployment, hosting, starting and stopping, monitoring applications and data.

### Enable IOx
In order to work with the guest shell, we need to enable first the IOx. I won't cover it here as part of this blog post. Though it's documented really well in the 'How to Enable Guest Shell' of this [guide](https://www.cisco.com/c/en/us/td/docs/ios-xml/ios/prog/configuration/166/b_166_programmability_cg/guest_shell.html).

### Enable guest shell
We need to enable connectivity between the guest shell, which is a container and the IOS XE (or outside network). Therefore we need to configure some network settings:

- Network settings of the host: create a Virtual Port Group
- Network settings of the container: define the guest shall as an application through `app-hosting appid guestshell`
- NAT configuration: create NAT entry on the IOS XE device to allow the container to access the outside world
- Enable guest shell: use the `guestshell enable` command to launch the guest shell container (takes several minutes)

Note: for the previous items, check out [this](https://www.ciscolive.com/c/dam/r/ciscolive/us/docs/2018/pdf/DEVNET-1695.pdf) Cisco Live.presentation. Slides 15-19 give a step-by-step approach of the above steps.

######  Guest Shell verification

######  Execute commands within Guest Shell 
After enabling the guestshell, you can access it as follows:

```bash
csr1000v# guestshell
[guestshell@guestshell ~]$
```
Next, you can execute Linux commands inside the guestshell (which is in fact a Centos linux environment). See below some examples:

```bash
[guestshell@guestshell ~]$


MORE EXAMPELS
```
######  Execute Guest Shell commands from IOS prompt  
In the above example, we ran all commands from within the guestshell. What if we want to run these commands from the IOS XE prompt. That's perfectly possible through the `guestshell run` command. See below:

```bash
csr1000v# guestshell run 


MORE EXAMPELS
```

###### Execute IOS-XE commands from Guest Shell prompt

```bash
[guestshell@guestshell ~]$ dohost "show memory statistics"

```

###### Installing packages in Guest Shell
```bash
[guestshell@guestshell ~]$ sudo yum update -y
<output omitted>
...
Complete!
```


### What is Python On-box


###### Run Python in Guest Shell


###### Commands overview

A) cli.cli(commands)


B) cli.clip(commands)


C) cli.execute(command)


D) cli.executep(command)


E) cli.configure(commands)


F) cli.configurep(commands)


###### Execute Python script

Install requests and do a simple request example



###### Perform backup to git

https://ondemandelearning.cisco.com/cisco-cte/enaui10/sections/5/pages/4

takes a backup of the switch configuration and commits it to a local Git repository and send notification to WebexTeams

###### EEM applet

Each time there is a config change, we will trigger the backup script, so essentially all configuration changes will be uploaded to Git.