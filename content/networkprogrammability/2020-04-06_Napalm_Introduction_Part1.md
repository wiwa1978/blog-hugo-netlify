---
title: NAPALM Introduction - Part 1
date: 2020-04-06T12:32:50+01:00
draft: false
categories:
  - Network Programming
  - Programming
  - All
tags:
  - Python
  - Napalm
---
### Introduction
Napalm stands for 'Network Automation and Programmability Abstraction Layer with Multivendor support (NAPALM)' and is a Python library that can be used to automate and interavt with networking devices and OSs using a unified API. So if you have multiple vendors in your network and don't want to write a dedicated automation script per vendor device, then NAPALM is exactly what you are looking for.

>Disclaimer: the code in this post is not production-grade code obviously. One should never store the username and password in the clear, not in the source code itself. The examples in the post are merely conceptual and for informational purposes.

### Installing the library
Installing NAPALM is very straightforward and is described very well in the online [documentation](https://napalm.readthedocs.io/en/latest/installation/). 
```bash
WAUTERW-M-65P7:Napalm_intro wauterw$ python3 -m venv venv
WAUTERW-M-65P7:Napalm_Introduction wauterw$ source venv/bin/activate
(venv) WAUTERW-M-65P7:Napalm_Introduction wauterw$ pip3 install napalm
Collecting napalm
**Truncated**
Successfully installed ciscoconfparse-1.5.1 colorama-0.4.3 dnspython-1.16.0 junos-eznc-2.2.1 napalm-2.5.0 netaddr-0.7.19 netmiko-2.4.2 nxapi-plumbing-0.5.2 passlib-1.7.2 pyIOSXR-0.53 pyYAML-5.3.1 pyeapi-0.8.3
```


### Getting to know NAPALM
In this first use case, we will give some examples for you to build an understanding on how NAPALM works. First of course, we will load the NAPALM library, more in particular the `get_network_driver` method. This immmediately triggers the question what network devices (or drivers) are supported. This can be found back in the documentation, see here the [support matrix](https://napalm.readthedocs.io/en/latest/support/index.html).

I'll be testing here with an IOS XR device from Cisco DevNet. We load the correct driver using the `get_network_driver` method. Here you essentially tell NAPALM that it needs to connect to an 'IOSXR' device. As you can see in the support matrix, under the hood NAPALM used the pyIOSXR Python library.

After making a connection to the device using the `driver()` method (the result from the get_network_driver method), we just open the connection using the `open()` connection. 

As we want to show what methods are supported by this connection, we are using the `dir()` function to print these.

```python
from napalm import get_network_driver
import json

driver_xr = get_network_driver("iosxr")

device = {
    "device_type": "cisco_xr",
    "ip": "sbx-iosxr-mgmt.cisco.com",
    "username": "***",
    "password": "***",
    "port": "8181",
}

device_xr = driver_xr(hostname=device['ip'], username=device['username'], password=device['password'], optional_args={'port':device['port']})
device_xr.open()
get_method = dir(device_xr)
print(json.dumps(get_method, sort_keys=True, indent=4))
```
This results in the following:

```bash
(venv) WAUTERW-M-65P7:Napalm_Introduction wauterw$ python3 start.py 
[
    "__class__",
    "__del__",
     ***Truncated***
    "compare_config",
    "compliance_report",
    "connection_tests",
    "device",
    "hostname",
    ***Truncated***
    "traceroute",
    "username"
]
```
The output represents a list of methods that can be called on this connection. Let's do this by adding following lines to our script:

```python
get_hostname = device_xr.hostname
print(f"Hostname is {get_hostname}")
```
This will result in:
```bash
venv) WAUTERW-M-65P7:Napalm_Introduction wauterw$ python3 start.py 
Hostname is sbx-iosxr-mgmt.cisco.com
```
Another method (refer to the output before), is the `get_facts` method. Let's call this by adding the following lines to our script:
```python
(venv) WAUTERW-M-65P7:Napalm_Introduction wauterw$ python3 start.py 
{
    "fqdn": "iosxr1",
    "hostname": "iosxr1",
    "interface_list": [
        "GigabitEthernet0/0/0/0",
        "GigabitEthernet0/0/0/0.144",
        "GigabitEthernet0/0/0/1",
        "GigabitEthernet0/0/0/2",
        "GigabitEthernet0/0/0/3",
        "GigabitEthernet0/0/0/4",
        "GigabitEthernet0/0/0/5",
        "GigabitEthernet0/0/0/6",
        "Loopback100",
        "Loopback200",
        "MgmtEth0/RP0/CPU0/0",
        "Null0"
    ],
    "model": "R-IOSXRV9000-CC",
    "os_version": "6.5.3",
    "serial_number": "05EFB4E4D3D",
    "uptime": 78892,
    "vendor": "Cisco"
}
```
As you can witness, this will give an overview of some important information, e.g hostname, interface overview, serial number and more. 

Let's pretend we want to know more about a given interface, let's say the IP configuration or the interface counters. Let's do this next.

```python
get_interfaces_counters = device_xr.get_interfaces_counters()
print(json.dumps(get_interfaces_counters, sort_keys=True, indent=4))

get_interfaces_ip = device_xr.get_interfaces_ip()
print(json.dumps(get_interfaces_ip, sort_keys=True, indent=4))
```
The output of above print statements is the provided in the below snippet. Note that I truncated the output quite a bit but I'm sure you get the picture.

```bash
(venv) WAUTERW-M-65P7:Napalm_Introduction wauterw$ python3 start.py 
{
    "GigabitEthernet0/0/0/0": {
        "rx_broadcast_packets": 0,
        "rx_discards": 0,
        "rx_errors": 0,
        "rx_multicast_packets": 0,
        "rx_octets": 0,
        "rx_unicast_packets": 0,
        "tx_broadcast_packets": 0,
        "tx_discards": 0,
        "tx_errors": 0,
        "tx_multicast_packets": 0,
        "tx_octets": 0,
        "tx_unicast_packets": 0
    },
    ***Truncated***
    "MgmtEth0/RP0/CPU0/0": {
        "ipv4": {
            "10.10.20.175": {
                "prefix_length": 24
            }
        }
    }
}
```
The code we have been using in this blog post can be found [here](https://github.com/wiwa1978/blog-hugo-netlify-code/tree/master/Napalm_Introduction). The start.py file covers exactly what we discussed in this post.