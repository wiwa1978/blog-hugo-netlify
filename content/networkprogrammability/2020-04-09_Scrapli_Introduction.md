---
title: Scrapli Introduction
date: 2020-04-09T12:32:50+01:00
draft: false
categories:
  - Network Programming
  - Programming
  - All
tags:
  - Python
  - Scrapli
---

### Introduction

 I first came across Scrapli on Twitter. 

![Scrapli](/images/2020-04-09-1.png)

It spiked my interest as these days I'm going through the major network automation tools. I read some good reviews on Twitter about it and hence I put it on my todo list to try out. Today the day has come to experiment a bit with it. 

Note: the Youtube video being mentioned in this tweet can be found [here](https://www.youtube.com/watch?v=twO0jYGQdH8&feature=emb_logo). Definitely worth listening to.

### What is Scrapli

Scrapli is a Python screenscraping library for network devices developed by Carl Montanari. It can be used for anything that you can interact with over Telnet or SSH. In his words:

> Scrapli -- scrap(e c)li -- is a python library focused on connecting to devices, specifically network devices (routers/switches/firewalls/etc.) via SSH or Telnet. The name scrapli -- is just "scrape cli" (as in screen scrape) squished together! scrapli's goal is to be as fast and flexible, while providing a well typed, well documented, simple API.

### Virtual Environment
As usual, we prefer to work within a virtual environment. Let's start with setting this up.
```bash
WAUTERW-M-65P7:Scrapli Introduction wauterw$ python3 -m venv venv
WAUTERW-M-65P7:Scrapli Introduction wauterw$ source venv/bin/activate
```

>Disclaimer: the code in this post is not production-grade code obviously. One should never store the username and password in the clear, not in the source code itself. The examples in the post are merely conceptual and for informational purposes.

### Installing Scrapli
Installation is dead simple. It's just installed through PIP.
```bash
(venv) WAUTERW-M-65P7:Scrapli Introduction wauterw$ pip3 install scrapli
```

### Note about equipment

For all the examples, we will use a Cisco sandbox environment delivered by [Cisco Devnet](https://developer.cisco.com). Go check out Devnet, really brilliant. To get a list of all sandboxes, check out [this](https://devnetsandbox.cisco.com/) link. For this tutorial, I'm using the IOS XE sandbox (see [here](https://devnetsandbox.cisco.com/RM/Diagram/Index/38ded1f0-16ce-43f2-8df5-43a40ebf752e?diagramType=Topology)) and the IOS XR sandbox (see [here](https://devnetsandbox.cisco.com/RM/Diagram/Index/e83cfd31-ade3-4e15-91d6-3118b867a0dd?diagramType=Topology)). 

### Device support 
Scrapli uses so called `drivers` to connect to various devices. Scrapli supports the following devices (see [here](https://carlmontanari.github.io/scrapli/#picking-the-right-driver)).

|  Platform/OS 	| Scrapli Driver 	            |
|:------------:	|:----------------:           |
| Cisco IOS-XE 	| IOSXEDriver    	            | 
| Cisco NX-OS  	| NXOSDriver     	            |
| Cisco IOS-XR 	| IOSXRDriver    	            |
| Arista EOS 	  | EOSDriver                   |
| Juniper JunOS	| IOSXRJunosDriverDriver    	|

### Use Case: Show version and interfaces for IOSXE device
In this first simple use case, we will use Scrapli to retrieve some information from our network device. 
We will be using an IOS XE device, so we'll start off with importing the correct driver `IOSXEDriver` from the `scrapli.driver.core` package. Next, we will create a device object that contains all the required arguments (full list [here](https://carlmontanari.github.io/scrapli/#basic-driver-arguments)) for Scrapli to be able to make a connection to our device. 

Next, explicitly open the connection using the `open()` method. Scrapli will not do this for us. Opening the connection is required in order to sending commands to your device.

The most 'important' part in the below script is the `send_commands` method. Speaks for itself but it allows us to send some commands to our device. In the below snippet, we are sending a `show version` and `show ip int brief` command and we capture the output in the responses object.


```python
from scrapli.driver.core import IOSXEDriver

device = {
    "host": "ios-xe-mgmt-latest.cisco.com",
    "auth_username": "***",
    "auth_password": "***",
    "port": 8181,
    "auth_strict_key": False,
}

conn = IOSXEDriver(**device)
conn.open()
responses = conn.send_commands(["show version", "show ip int brief"])

for response in responses:
   print(response.result)
conn.close()
```
In below output you can verify indeed that the script returned the output for the `show version` and the `show ip int brief` respectively.
```bash
(venv) WAUTERW-M-65P7:Scrapli Introduction wauterw$ python3 example1.py 
(venv) WAUTERW-M-65P7:Scrapli Introduction wauterw$ python3 example1.py 
RESPONSE
****************************************************************************************************
Cisco IOS XE Software, Version 16.11.01a
***Truncated***
csr1000v-1 uptime is 5 days, 2 minutes
Uptime for this control processor is 5 days, 4 minutes
***Truncated***
Configuration register is 0x2102
RESPONSE
****************************************************************************************************
Interface              IP-Address      OK? Method Status                Protocol
GigabitEthernet1       10.10.20.48     YES NVRAM  up                    up
GigabitEthernet2       10.20.221.100   YES other  up                    up
***Truncated***
```

### Use Case: Show interfaces for IOSXE and IOSXR device
In case you wonder how good Scrapli works with network devices running different OS's, check out the below script. It's very similar to the one we discussed above, but it handles two different network devices. Note: we are only sending a single command to our device, so we have opted for the `send_command` method (not the `send_commands` method as we did in previous example).
```python
from scrapli.driver.core import IOSXEDriver
from scrapli.driver.core import IOSXRDriver

devices = [{
    "host": "ios-xe-mgmt-latest.cisco.com",
    "auth_username": "***",
    "auth_password": "***",
    "port": 8181,
    "auth_strict_key": False,
}, {
    "host": "sbx-iosxr-mgmt.cisco.com",
    "auth_username": "***",
    "auth_password": "***",
    "port": 8181,
    "auth_strict_key": False,
}]

conn_XE = IOSXEDriver(**devices[0])
conn_XE.open()
response = conn_XE.send_command("show ip int brief")
print("RESPONSE")
print("*" * 100)
print(response.result)
conn_XE.close()


conn_XR = IOSXRDriver(**devices[1])
conn_XR.open()
response = conn_XR.send_command("show ip int brief")

print("RESPONSE")
print("*" * 100)
print(response.result)
conn_XR.close()
```
In the output below, you can verify indeed that we get back a list of interfaces for both the IOSXE and IOSXR device, all with the same `send_commands` method.
```bash
(venv) WAUTERW-M-65P7:Scrapli Introduction wauterw$ python3 example2.py 
RESPONSE
****************************************************************************************************
Interface              IP-Address      OK? Method Status                Protocol
GigabitEthernet1       10.10.20.48     YES NVRAM  up                    up
GigabitEthernet2       10.20.221.100   YES other  up                    up
GigabitEthernet2.4     unassigned      YES manual deleted               down
***Truncated***
RESPONSE
****************************************************************************************************
Fri Apr 09 18:57:07.499 UTC

Interface                      IP-Address      Status          Protocol Vrf-Name
Loopback100                    4.4.4.101       Up              Up       default
Loopback200                    1.1.1.200       Shutdown        Down     default
MgmtEth0/RP0/CPU0/0            10.10.20.175    Up              Up       default
***Truncated***
```

### Use Case: Set interface description for IOSXE device
In the previous two use cases, we have essentially retrieved information from a variety of network devices. How about making configuration changes?

Also this is pretty straightforward using Scrapli. Let's check it out.

In below snippet, we first use the `send_command` method to retrieve the description for one of our devices' interfaces. Next, we will use the `send_configs` method to change the interface description and finally we will check if everything worked nicely.

```python
from scrapli.driver.core import IOSXEDriver

device = {
    "host": "ios-xe-mgmt-latest.cisco.com",
    "auth_username": "***",
    "auth_password": "***",
    "port": 8181,
    "auth_strict_key": False,
}

conn = IOSXEDriver(**device)
conn.open()
response = conn.send_command("show interface Gi3 description")
print("*"* 100)
print(response.result)

conn.send_configs(["interface GigabitEthernet3", "description Configured by Scrapli"])

response = conn.send_command("show interface Gi3 description")
print("*"* 100)
print(response.result)
```
Below you'll find the output. And indeed, as you can witness, the interface description was set correctly.

```
(venv) WAUTERW-M-65P7:Scrapli Introduction wauterw$ python3 example3.py 
****************************************************************************************************
Interface                      Status         Protocol Description
Gi3                            up             up       Devnet noob is getting at it
****************************************************************************************************
Interface                      Status         Protocol Description
Gi3                            up             up       Configured by Scrapli
```

That's it for this introductory post on Scrapli. Code can be found [here](https://github.com/wiwa1978/blog-hugo-netlify-code/tree/master/Scrapli%20Introduction).

 I have been reading of Scrapli integrations with Nornir, Scrapli integration with Genie etc so I'll keep this for one of my next posts. Stay tuned!