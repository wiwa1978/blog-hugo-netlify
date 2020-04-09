---
title: Scraplu Introduction
date: 2020-04-15T12:32:50+01:00
draft: True
categories:
  - Network Programming
  - Programming
tags:
  - Python
  - Scrapli
---

### Introduction

scrapli -- scrap(e c)li -- is a python library focused on connecting to devices, specifically network devices (routers/switches/firewalls/etc.) via SSH or Telnet. The name scrapli -- is just "scrape cli" (as in screen scrape) squished together! scrapli's goal is to be as fast and flexible, while providing a well typed, well documented, simple API.

### Virtual Environment

```bash
WAUTERW-M-65P7:Scrapli Introduction wauterw$ python3 -m venv venv
WAUTERW-M-65P7:Scrapli Introduction wauterw$ source venv/bin/activate
```

### Installation

```bash
(venv) WAUTERW-M-65P7:Scrapli Introduction wauterw$ pip3 install scrapli
```

### Note about equipment

For all the examples, we will use a Cisco sandbox environment delivered by [Cisco Devnet](https://developer.cisco.com). Go check out Devnet, really brilliant. To get a list of all sandboxes, check out [this](https://devnetsandbox.cisco.com/) link. For this tutorial, I'm using the IOS XE sandbox (see [here](https://devnetsandbox.cisco.com/RM/Diagram/Index/38ded1f0-16ce-43f2-8df5-43a40ebf752e?diagramType=Topology)) and the IOS XR sandbox (see [here](https://devnetsandbox.cisco.com/RM/Diagram/Index/e83cfd31-ade3-4e15-91d6-3118b867a0dd?diagramType=Topology)). 

### Show version and interfaces for IOSXE device

```python
from scrapli.driver.core import IOSXEDriver

device = {
    "host": "ios-xe-mgmt-latest.cisco.com",
    "auth_username": "developer",
    "auth_password": "C1sco12345",
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

```bash
(venv) WAUTERW-M-65P7:Scrapli Introduction wauterw$ python3 example1.py 
RESPONSE
****************************************************************************************************
Cisco IOS XE Software, Version 16.11.01a
Cisco IOS Software [Gibraltar], Virtual XE Software (X86_64_LINUX_IOSD-UNIVERSALK9-M), Version 16.11.1a, RELEASE SOFTWARE (fc1)
Technical Support: http://www.cisco.com/techsupport
Copyright (c) 1986-2019 by Cisco Systems, Inc.
Compiled Thu 11-Apr-19 23:59 by mcpre


Cisco IOS-XE software, Copyright (c) 2005-2019 by cisco Systems, Inc.
All rights reserved.  Certain components of Cisco IOS-XE software are
licensed under the GNU General Public License ("GPL") Version 2.0.  The
software code licensed under GPL Version 2.0 is free software that comes
with ABSOLUTELY NO WARRANTY.  You can redistribute and/or modify such
GPL code under the terms of GPL Version 2.0.  For more details, see the
documentation or "License Notice" file accompanying the IOS-XE software,
or the applicable URL provided on the flyer accompanying the IOS-XE
software.


ROM: IOS-XE ROMMON

csr1000v-1 uptime is 2 days, 28 minutes
Uptime for this control processor is 2 days, 29 minutes
System returned to ROM by reload
System image file is "bootflash:packages.conf"
Last reload reason: reload



This product contains cryptographic features and is subject to United
States and local country laws governing import, export, transfer and
use. Delivery of Cisco cryptographic products does not imply
third-party authority to import, export, distribute or use encryption.
Importers, exporters, distributors and users are responsible for
compliance with U.S. and local country laws. By using this product you
agree to comply with applicable laws and regulations. If you are unable
to comply with U.S. and local laws, return this product immediately.

A summary of U.S. laws governing Cisco cryptographic products may be found at:
http://www.cisco.com/wwl/export/crypto/tool/stqrg.html

If you require further assistance please contact us by sending email to
export@cisco.com.

License Level: ax
License Type: N/A(Smart License Enabled)
Next reload license Level: ax


Smart Licensing Status: UNREGISTERED/No Licenses in Use

cisco CSR1000V (VXE) processor (revision VXE) with 2378575K/3075K bytes of memory.
Processor board ID 9RYMNIFHO62
3 Gigabit Ethernet interfaces
32768K bytes of non-volatile configuration memory.
8112832K bytes of physical memory.
7774207K bytes of virtual hard disk at bootflash:.
0K bytes of WebUI ODM Files at webui:.

Configuration register is 0x2102
RESPONSE
****************************************************************************************************
Interface              IP-Address      OK? Method Status                Protocol
GigabitEthernet1       10.10.20.48     YES NVRAM  up                    up
GigabitEthernet2       unassigned      YES NVRAM  up                    up
GigabitEthernet2.4     192.168.1.1     YES manual up                    up
GigabitEthernet3       unassigned      YES NVRAM  administratively down down
GigabitEthernet3.4     192.168.2.1     YES manual administratively down down
Loopback70             7.7.7.225       YES manual up                    up
Loopback71             7.7.7.193       YES manual up                    up
Loopback72             7.7.7.161       YES manual up                    up
Loopback73             7.7.7.129       YES manual up                    up
Loopback76             unassigned      YES unset  up                    up
Loopback100            172.16.100.1    YES other  up                    up
Loopback101            101.101.101.101 YES other  up                    up
Loopback102            102.102.102.102 YES other  up                    up
Loopback103            103.103.103.103 YES manual up                    up
Loopback133            1.1.1.1         YES manual up                    up
Loopback155            1.5.1.5         YES manual up                    up
Loopback700            1.1.1.1         YES manual up                    up
Port-channel1          unassigned      YES unset  down                  down
VirtualPortGroup0      192.168.1.1     YES manual up                    up
```

### Show version and interfaces for IOSXE and IOSXR device

```python3
from scrapli.driver.core import IOSXEDriver
from scrapli.driver.core import IOSXRDriver

devices = [{
    "host": "ios-xe-mgmt-latest.cisco.com",
    "auth_username": "developer",
    "auth_password": "C1sco12345",
    "port": 8181,
    "auth_strict_key": False,
}, {
    "host": "sbx-iosxr-mgmt.cisco.com",
    "auth_username": "admin",
    "auth_password": "C1sco12345",
    "port": 8181,
    "auth_strict_key": False,
}]

conn_XE = IOSXEDriver(**devices[0])
conn_XE.open()
responses = conn_XE.send_commands(["show version", "show ip int brief"])

for response in responses:
   print("RESPONSE")
   print("*" * 100)
   print(response.result)
conn_XE.close()


conn_XR = IOSXRDriver(**devices[1])
conn_XR.open()
responses = conn_XR.send_commands(["show version", "show ip int brief"])

for response in responses:
   print("RESPONSE")
   print("*" * 100)
   print(response.result)
conn_XR.close()
```



```bash
(venv) WAUTERW-M-65P7:Scrapli Introduction wauterw$ python3 example2.py 
RESPONSE
****************************************************************************************************
Cisco IOS XE Software, Version 16.11.01a
Cisco IOS Software [Gibraltar], Virtual XE Software (X86_64_LINUX_IOSD-UNIVERSALK9-M), Version 16.11.1a, RELEASE SOFTWARE (fc1)
Technical Support: http://www.cisco.com/techsupport
Copyright (c) 1986-2019 by Cisco Systems, Inc.
Compiled Thu 11-Apr-19 23:59 by mcpre


Cisco IOS-XE software, Copyright (c) 2005-2019 by cisco Systems, Inc.
All rights reserved.  Certain components of Cisco IOS-XE software are
licensed under the GNU General Public License ("GPL") Version 2.0.  The
software code licensed under GPL Version 2.0 is free software that comes
with ABSOLUTELY NO WARRANTY.  You can redistribute and/or modify such
GPL code under the terms of GPL Version 2.0.  For more details, see the
documentation or "License Notice" file accompanying the IOS-XE software,
or the applicable URL provided on the flyer accompanying the IOS-XE
software.


ROM: IOS-XE ROMMON

csr1000v-1 uptime is 2 days, 1 hour, 2 minutes
Uptime for this control processor is 2 days, 1 hour, 3 minutes
System returned to ROM by reload
System image file is "bootflash:packages.conf"
Last reload reason: reload



This product contains cryptographic features and is subject to United
States and local country laws governing import, export, transfer and
use. Delivery of Cisco cryptographic products does not imply
third-party authority to import, export, distribute or use encryption.
Importers, exporters, distributors and users are responsible for
compliance with U.S. and local country laws. By using this product you
agree to comply with applicable laws and regulations. If you are unable
to comply with U.S. and local laws, return this product immediately.

A summary of U.S. laws governing Cisco cryptographic products may be found at:
http://www.cisco.com/wwl/export/crypto/tool/stqrg.html

If you require further assistance please contact us by sending email to
export@cisco.com.

License Level: ax
License Type: N/A(Smart License Enabled)
Next reload license Level: ax


Smart Licensing Status: UNREGISTERED/No Licenses in Use

cisco CSR1000V (VXE) processor (revision VXE) with 2378575K/3075K bytes of memory.
Processor board ID 9RYMNIFHO62
3 Gigabit Ethernet interfaces
32768K bytes of non-volatile configuration memory.
8112832K bytes of physical memory.
7774207K bytes of virtual hard disk at bootflash:.
0K bytes of WebUI ODM Files at webui:.

Configuration register is 0x2102
RESPONSE
****************************************************************************************************
Interface              IP-Address      OK? Method Status                Protocol
GigabitEthernet1       10.10.20.48     YES NVRAM  up                    up
GigabitEthernet2       unassigned      YES NVRAM  up                    up
GigabitEthernet2.4     192.168.1.1     YES manual up                    up
GigabitEthernet3       unassigned      YES NVRAM  administratively down down
GigabitEthernet3.4     192.168.2.1     YES manual administratively down down
Loopback70             7.7.7.225       YES manual up                    up
Loopback71             7.7.7.193       YES manual up                    up
Loopback72             7.7.7.161       YES manual up                    up
Loopback73             7.7.7.129       YES manual up                    up
Loopback76             unassigned      YES unset  up                    up
Loopback100            172.16.100.1    YES other  up                    up
Loopback101            101.101.101.101 YES other  up                    up
Loopback102            102.102.102.102 YES other  up                    up
Loopback103            103.103.103.103 YES manual up                    up
Loopback133            1.1.1.1         YES manual up                    up
Loopback155            1.5.1.5         YES manual up                    up
Loopback700            1.1.1.1         YES manual up                    up
Port-channel1          unassigned      YES unset  down                  down
VirtualPortGroup0      192.168.1.1     YES manual up                    up
RESPONSE
****************************************************************************************************
Tue Apr  7 19:50:31.426 UTC
Cisco IOS XR Software, Version 6.5.3
Copyright (c) 2013-2019 by Cisco Systems, Inc.

Build Information:
 Built By     : ahoang
 Built On     : Tue Mar 26 06:52:25 PDT 2019
 Built Host   : iox-ucs-019
 Workspace    : /auto/srcarchive13/prod/6.5.3/xrv9k/ws
 Version      : 6.5.3
 Location     : /opt/cisco/XR/packages/

cisco IOS-XRv 9000 () processor
System uptime is 4 days 2 hours 45 minutes
RESPONSE
****************************************************************************************************
Tue Apr  7 19:50:31.827 UTC

Interface                      IP-Address      Status          Protocol Vrf-Name
Loopback100                    1.1.1.100       Up              Up       default
Loopback200                    1.1.1.200       Up              Up       default
MgmtEth0/RP0/CPU0/0            10.10.20.175    Up              Up       default
GigabitEthernet0/0/0/0         unassigned      Up              Up       default
GigabitEthernet0/0/0/0.144     10.10.20.1      Up              Down     default
GigabitEthernet0/0/0/1         unassigned      Shutdown        Down     default
GigabitEthernet0/0/0/2         unassigned      Shutdown        Down     default
GigabitEthernet0/0/0/3         unassigned      Shutdown        Down     default
GigabitEthernet0/0/0/4         unassigned      Shutdown        Down     default
GigabitEthernet0/0/0/5         unassigned      Shutdown        Down     default
GigabitEthernet0/0/0/6         unassigned      Shutdown        Down     default
```

### Set interface description for IOSXE device

```python
from scrapli.driver.core import IOSXEDriver

device = {
    "host": "ios-xe-mgmt-latest.cisco.com",
    "auth_username": "developer",
    "auth_password": "C1sco12345",
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


```
(venv) WAUTERW-M-65P7:Scrapli Introduction wauterw$ python3 example3.py 
****************************************************************************************************
Interface                      Status         Protocol Description
Gi3                            admin down     down     Configured by Netmiko
****************************************************************************************************
Interface                      Status         Protocol Description
Gi3                            admin down     down     Configured by Scrapli
```