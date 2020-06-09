---
title: Exploring Guest Shell and Python On-box on IOS-XE
date: 2020-06-05T10:19:50+01:00
draft: false
categories:
  - Network Programming
  - Programming
  - All
tags:
  - Python
  - Guestshell
---
### What is Guest Shell?

Cisco IOS-XE devices (as of release 16.6.1) come with a guest shell, which is essentlially a virtualized Linux (CentOS) environment. It allows to run Linux applications on the network device. It comes prepopulated with common tools like net-tools, iproute, tcpdump and OpenSSH but it allows you also to install and update Linux applications (think of Puppet agents, Chef agents, Splunk agents...).  Also Python (at the moment version 2.7) is integrated in the Linux environment so the guest shell allows you to run Python scripts as well. The guest shell is intended for tools, Linux utilities and manageability rather than networking functions.

The guest shell container is managed through IOx, which is Cisco's Application Hosting Infrastructure for Cisco IOS XE devices. It provides application hosting capabilities for different application types. The guest shell (container deployment) is actually just one such application. The IOx facilitates the lifecycle management of applications, think of distribution, deployment, hosting, starting and stopping, monitoring applications and data.

### Note about equipment

For all the examples in this post, we will use a Cisco sandbox environment delivered by [Cisco Devnet](https://developer.cisco.com). Go check out Devnet, really brilliant. To get a list of all sandboxes, check out [this](https://devnetsandbox.cisco.com/) link. For this tutorial, I'm using the IOS XE sandbox (see [here](https://devnetsandbox.cisco.com/RM/Diagram/Index/27d9747a-db48-4565-8d44-df318fce37ad?diagramType=Topology). In case you prefer to reserve an instance, check out [this](https://devnetsandbox.cisco.com/RM/Diagram/Index/3fd70d1b-0432-4ff8-98ff-1a5d8ec01a72) one.

### Enable IOx
In order to work with the guest shell, we need to enable first the IOx. I won't cover it here as part of this blog post. Though it's documented really well in the 'How to Enable Guest Shell' of this [guide](https://www.cisco.com/c/en/us/td/docs/ios-xml/ios/prog/configuration/166/b_166_programmability_cg/guest_shell.html).

### Enable guest shell
We need to enable connectivity between the guest shell, which is a container and the IOS XE (or outside network). Therefore we need to configure some network settings:

- Network settings of the host: create a Virtual Port Group
- Network settings of the container: define the guest shall as an application through `app-hosting appid guestshell`
- NAT configuration: create NAT entry on the IOS XE device to allow the container to access the outside world
- Enable guest shell: use the `guestshell enable` command to launch the guest shell container (takes several minutes)

Note: for the previous items, check out [this](https://www.ciscolive.com/c/dam/r/ciscolive/us/docs/2018/pdf/DEVNET-1695.pdf) Cisco Live.presentation. Slides 15-19 give a step-by-step approach of the above steps.

### Applied to the reserved sandbox

Step 1: Enable iox
```bash
csr1000v-1#conf t
Enter configuration commands, one per line.  End with CNTL/Z.
csr1000v-1(config)#iox
csr1000v-1(config)#end
```
Step 2: Network settings of the host
```
csr1000v-1#conf t
Enter configuration commands, one per line.  End with CNTL/Z.
csr1000v-1(config)#interface VirtualPortGroup0
csr1000v-1(config-if)#ip address 192.168.1.1 255.255.255.0
csr1000v-1(config-if)#end
csr1000v-1#conf t
Enter configuration commands, one per line.  End with CNTL/Z.
csr1000v-1(config)#app-hosting appid guestshell
csr1000v-1(config-app-hosting)#vnic gateway1 virtualportgroup 0 guest-interface 0 guest-ipaddress 192.168.1.2 netmask 255.255.255.0 gateway 192.168.1.1 name-server 8.8.8.8
csr1000v-1(config-app-hosting)#end
```
Step 3: NAT configuration
```bash
csr1000v-1#conf t
Enter configuration commands, one per line.  End with CNTL/Z.
csr1000v-1(config)#interface VirtualPortGroup0
csr1000v-1(config-if)#ip nat inside
csr1000v-1(config-if)#exit
csr1000v-1(config)#interface Gi
csr1000v-1(config)#interface GigabitEthernet1
csr1000v-1(config-if)#ip nat outside
csr1000v-1(config-if)#exit
csr1000v-1(config)#ip access-list extended NAT-ACL
csr1000v-1(config-ext-nacl)#permit ip 192.168.1.0 0.0.0.255 any
csr1000v-1(config-ext-nacl)#exit
csr1000v-1(config)#ip nat inside source list NAT-ACL interface GigabitEthernet1 overload
csr1000v-1(config)#end
```
Step 4: Enable guestshell
```bash
csr1000v-1#guestshell enable
Interface will be selected if configured in app-hosting
Please wait for completion
guestshell installed successfully
Current state is: DEPLOYED
guestshell activated successfully
Current state is: ACTIVATED
guestshell started successfully
Current state is: RUNNING
Guestshell enabled successfully
csr1000v-1#guestshell
[guestshell@guestshell ~]$
```

######  Execute commands within Guest Shell 
After enabling the guestshell, you can access it as follows:

```bash
csr1000v# guestshell
[guestshell@guestshell ~]$
```
Next, you can execute Linux commands inside the guestshell (which is in fact a Centos linux environment). See below some examples:

```bash
csr1000v#guestshell
[guestshell@guestshell ~]$ whoami
guestshell
[guestshell@guestshell ~]$ pwd
/home/guestshell
[guestshell@guestshell ~]$ cat /etc/centos-release
CentOS Linux release 7.8.2003 (Core)
```
The Guest Shell container is isolated from the rest of operating system - it can't consume more resources than were dedicated to it and it does not impact the host operating system.

######  Execute Guest Shell commands from IOS prompt  
In the above example, we ran all commands from within the guestshell. What if we want to run these commands from the IOS XE prompt. That's perfectly possible through the `guestshell run` command. See below:

```bash
csr1000v#guestshell run whoami
guestshell

csr1000v#guestshell run pwd
/home/guestshell

csr1000v#guestshell run cat /etc/centos-release
CentOS Linux release 7.8.2003 (Core)
```

###### Execute IOS-XE commands from Guest Shell prompt
From within the guestshell it is also possible to execute IOS XE CLI commands using dohost binary

```bash
csr1000v#guestshell
[guestshell@guestshell ~]$ dohost "show memory statistics"

Tracekey : 1#9ae4e6ddd6c2aa2a21997806924e75ee
                Head    Total(b)     Used(b)     Free(b)   Lowest(b)  Largest(b)
Processor  7F9125F80010   2449862632   355170016   2094692616   2092880960   1479733148
 lsmpi_io  7F91257351A8     3149400     3148576         824         824         412

[guestshell@guestshell ~]$
[guestshell@guestshell ~]$
[guestshell@guestshell ~]$ dohost "show ip interface brief"

Interface              IP-Address      OK? Method Status                Protocol
GigabitEthernet1       10.10.20.48     YES other  up                    up
GigabitEthernet2       10.255.255.1    YES other  up                    up
GigabitEthernet3       192.168.122.205 YES other  up                    up
Loopback100            172.16.100.1    YES other  up                    up
Loopback103            172.16.102.1    YES other  up                    up
Loopback321            10.3.2.1        YES other  up                    up
Loopback420            4.20.20.20      YES other  up                    up
Loopback666            12.13.14.1      YES other  up                    up
Loopback901            unassigned      YES unset  up                    up
Loopback1000           unassigned      YES unset  up                    up
Loopback1230           1.2.3.0         YES other  up                    up
Loopback1233           1.2.3.3         YES other  up                    up
Loopback1234           1.2.3.4         YES other  up                    up
Loopback1235           1.2.3.5         YES TFTP   up                    up
Loopback1919           unassigned      YES unset  up                    up
Loopback3040           10.30.40.1      YES other  up                    up
Loopback4040           10.40.40.1      YES other  up                    up
Loopback5040           10.50.40.1      YES other  up                    up
Loopback5555           5.5.5.5         YES other  up                    up
Loopback6040           10.60.40.1      YES other  up                    up
Loopback6692           12.12.12.2      YES other  up                    up
Loopback6694           12.12.12.4      YES other  up                    up
Loopback6789           6.7.8.9         YES other  up                    up
VirtualPortGroup0      192.168.1.1     YES other  up                    up
```

###### Installing packages in Guest Shell
```bash
[guestshell@guestshell ~]$ sudo yum update -y
Loaded plugins: fastestmirror
Loading mirror speeds from cached hostfile
 * base: mirror.sitbv.nl
 * extras: mirrors.powernet.com.ru
 * updates: mirror.cherryservers.com
<truncated output>
...
Complete!
```s

### What is Python On-box
On-Box Python is Python interpreter installed inside the Guest Shell that allows to run Python scripts on IOS XE router or switch. It is tied with Embedded Event Manager (EEM) and has direct access to IOS XE CLI commands using custom module called `cli`. It allows to react to different events directly on the device (Edge Computing), this includes additional reporting, post-configuration checks, complex troubleshooting capabilities, automation of repetitive tasks.

###### Run Python in Guest Shell

```bash
csr1000v-1#guestshell run python
Python 2.7.5 (default, Jun 17 2014, 18:11:42)
[GCC 4.8.2 20140120 (Red Hat 4.8.2-16)] on linux2
Type "help", "copyright", "credits" or "license" for more information.
>>>
```
###### Commands overview

A) cli.cli(commands)

```bash
>>> import cli
>>> output = cli.cli("show ip interface brief")
>>> print (output)

Interface              IP-Address      OK? Method Status                Protocol
GigabitEthernet1       10.10.20.48     YES NVRAM  up                    up
GigabitEthernet2       unassigned      YES NVRAM  administratively down down
GigabitEthernet3       unassigned      YES NVRAM  administratively down down
VirtualPortGroup0      192.168.1.1     YES manual up                    up

>>> cli.cli("show ip interface brief")
'\nInterface              IP-Address      OK? Method Status                Protocol\nGigabitEthernet1       10.10.20.48     YES NVRAM  up                    up      \nGigabitEthernet2       unassigned      YES NVRAM  administratively down down    \nGigabitEthernet3       unassigned      YES NVRAM  administratively down down    \nVirtualPortGroup0      192.168.1.1     YES manual up                    up      \n'
```
You will have noticed that cli.cli returns the result to a variable that you can then print. You could also directly print it to screen, but then it would essentially return everything as a single string. In that case, it's better to use the `cli.clip ` command.

B) cli.clip(commands)

```bash
>>> cli.clip("show ip interface brief")

Interface              IP-Address      OK? Method Status                Protocol
GigabitEthernet1       10.10.20.48     YES NVRAM  up                    up
GigabitEthernet2       unassigned      YES NVRAM  administratively down down
GigabitEthernet3       unassigned      YES NVRAM  administratively down down
VirtualPortGroup0      192.168.1.1     YES manual up                    up
```

In case you want to run two or more commands, seperate these with the `;`. Pay also attention to the quotes. It should be:
`"show ip interface brief; show interface description"` and not `"show ip interface brief"; "show interface description"`. See below for an example:

```bash
>>> cli.clip("show ip interface brief; show interface description")

Interface              IP-Address      OK? Method Status                Protocol
GigabitEthernet1       10.10.20.48     YES NVRAM  up                    up
GigabitEthernet2       unassigned      YES NVRAM  administratively down down
GigabitEthernet3       unassigned      YES NVRAM  administratively down down
VirtualPortGroup0      192.168.1.1     YES manual up                    up
Interface                      Status         Protocol Description
Gi1                            up             up       MANAGEMENT INTERFACE - DON'T TOUCH ME
Gi2                            admin down     down     Network Interface
Gi3                            admin down     down     Network Interface
```
C) cli.execute(command)

This one is similar to cli.cli but only runs 1 single command

```bash
>>> output = cli.execute("show ip interface brief")
>>> print(output)
Interface              IP-Address      OK? Method Status                Protocol
GigabitEthernet1       10.10.20.48     YES NVRAM  up                    up
GigabitEthernet2       unassigned      YES NVRAM  administratively down down
GigabitEthernet3       unassigned      YES NVRAM  administratively down down
VirtualPortGroup0      192.168.1.1     YES manual up                    up
>>> cli.execute("show ip interface brief")
'Interface              IP-Address      OK? Method Status                Protocol\nGigabitEthernet1       10.10.20.48     YES NVRAM  up                    up      \nGigabitEthernet2       unassigned      YES NVRAM  administratively down down    \nGigabitEthernet3       unassigned      YES NVRAM  administratively down down    \nVirtualPortGroup0      192.168.1.1     YES manual up                    up
```
Also here, you will notice that cli.execute returns the result to a variable that you can then print. You could also directly print it to screen, but then it would essentially return everything as a single string. In that case, it's better to use the `cli.executep ` command.

D) cli.executep(command)
Similarly, the cli.executep executes a single command but does print out the output directly to screen.

```bash
>>> cli.executep("show ip interface brief")
Interface              IP-Address      OK? Method Status                Protocol
GigabitEthernet1       10.10.20.48     YES NVRAM  up                    up
GigabitEthernet2       unassigned      YES NVRAM  administratively down down
GigabitEthernet3       unassigned      YES NVRAM  administratively down down
VirtualPortGroup0      192.168.1.1     YES manual up                    up
```
E) cli.configure(commands)

```bash
>>> output = cli.configure(["interface Loopback5000", "description Set through cli.configure"])
>>> print(output)
[ConfigResult(success=True, command='interface Loopback5000', line=1, output='', notes=None), ConfigResult(success=True, command='description Set through cli.configure', line=2, output='', notes=None)]
```
Let's see if it worked...We can use one of the previous commands for this:
```bash
>>> cli.clip("show ip interface brief; show interface description")

Interface              IP-Address      OK? Method Status                Protocol
GigabitEthernet1       10.10.20.48     YES NVRAM  up                    up
GigabitEthernet2       unassigned      YES NVRAM  administratively down down
GigabitEthernet3       unassigned      YES NVRAM  administratively down down
Loopback5000           unassigned      YES unset  up                    up
VirtualPortGroup0      192.168.1.1     YES manual up                    up
Interface                      Status         Protocol Description
Gi1                            up             up       MANAGEMENT INTERFACE - DON'T TOUCH ME
Gi2                            admin down     down     Network Interface
Gi3                            admin down     down     Network Interface
Lo5000                         up             up       Set through cli.configure
Vi0                            up             up
```
You can see the loopback interface was added successfully and the description was set properly. 

Note: unfortunately in this case, with `cli.configure` we need to use a `,` to seperate multiple commands. 

F) cli.configurep(commands)
Let's now use the `cli.configurep` command to remove the Loopback interface that we added previously.
```bash
>>> cli.configurep("no interface Loopback5000")
Line 1 SUCCESS: no interface Loopback5000
>>> cli.executep("show ip interface brief")
Interface              IP-Address      OK? Method Status                Protocol
GigabitEthernet1       10.10.20.48     YES NVRAM  up                    up
GigabitEthernet2       unassigned      YES NVRAM  administratively down down
GigabitEthernet3       unassigned      YES NVRAM  administratively down down
VirtualPortGroup0      192.168.1.1     YES manual up                    up
>>>
```