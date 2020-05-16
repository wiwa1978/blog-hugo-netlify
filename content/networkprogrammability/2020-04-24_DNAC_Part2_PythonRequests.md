---
title: Cisco DNA Center - Devices
date: 2020-04-24T12:32:50+01:00
draft: false
categories:
  - Network Programming
  - Programming
  - All
tags:
  - DNAC
  - Python
---

### DNAC Series

This is part of a DNAC series:

- Part 1: [Getting started](https://blog.wimwauters.com/networkprogrammability/2020-04-22_dnac_gettingstarted/)
- Part 2 (this post): [Cisco DNA Center - Devices](https://blog.wimwauters.com/networkprogrammability/2020-04-24_dnac_part1_pythonrequests/)
- Part 3: [Cisco DNA Center - Assurance](https://blog.wimwauters.com/networkprogrammability/2020-04-25_dnac_part2_pythonrequests/)
- Part 4: [Cisco DNA Center - Sites](https://blog.wimwauters.com/networkprogrammability/2020-04-27_dnac_part3_pythonrequests/)
- Part 5: [Cisco DNA Center - Discovery (POSTMAN)](https://blog.wimwauters.com/networkprogrammability/2020-04-29_dnac_part4_postman_networkdiscovery/)
- Part 6: [Cisco DNA Center - Discovery (Python)](https://blog.wimwauters.com/networkprogrammability/2020-05-01_dnac_part5_pythonrequests/)
- Part 7: [Cisco DNA Center - CommandRunner (Python)](https://blog.wimwauters.com/networkprogrammability/2020-05-02_dnac_part6_pythonrequests/)

### Introduction

In this [post](https://blog.wimwauters.com/networkprogrammability/2020-04-22_dnac_gettingstarted/), we introduced DNAC at a fairly high level and we have shown some POSTMAN samples to get a basic understanding of some of the APIs like authentication and devices). In this post, we will continue that journey as we will show some  Python samples.

>Disclaimer: the code in this post is not production-grade code obviously.  The examples in the post are merely conceptual and for informational purposes.

### Authentication through Python

```python
import requests
import json

def get_token(dnac="sandboxdnac2.cisco.com"):  
   url = f"https://{dnac}/dna/system/api/v1/auth/token"

   username = "***"
   password = "***"

   headers = {
      "Content-Type": "application/json",
   }

   requests.packages.urllib3.disable_warnings()
   response =  requests.post(url, auth=(username, password), verify=False).json()

   return response["Token"]

if __name__ == "__main__":
   token = get_token()
   print(token)
```
Executing this script, results in following output:
```
WAUTERW-M-65P7:Devices wauterw$ python authenticate.py 
eyJ0eXAiOiJKV1QiLCJhbGciOiJSUzI1NiJ9.eyJzdWIiOiI1ZTVhN2JmNDc1MTYxMjAwY2M0YWUwNjQiLCJhdXRoU291cmNlIjoiaW50ZXJuYWwiLCJ0ZW5hbnROYW1lIjoiVE5UMCIsInJvbGVzIjpbIjVlNWE0MzI2NzUxNjEyMDBjYzRhYzk2MyJdLCJ0ZW5hbnRJZCI6IjVlNWE0MzI1NzUxNjEyMDBjYzRhYzk1YyIsImV4cCI6MTU4NzU0OTY0MywiaWF0IjoxNTg3NTQ2MDQzLCJqdGkiOiJjNDUyOTI3OS01ZTkwLTRhZmMtYjYxYS1hNzJiNWM0NmMzOTMiLCJ1c2VybmFtZSI6ImRldm5ldHVzZXIifQ.Kq6LM60eRXg2ubmiXjh8Vy5fmLN3M1VkmTj0CLLMYYvU6G8-s3EU7MVvFCbh8MW4RjJUGrZW4SHm33WitVRZAG2kqOi7o7C44gHkT7jiTb8a4D28mbG5G7JXRPTi6_77wK6l09plualxLYmT3jYGTaRvhpIHqg_q7Y8e2GYUsjOROkqW7mz8NrvHvVuK1nGQ1JrP-kOn0C30iq2ybzFMihJHUxVLwDrRDzXVo4n793Rb70Tmuu1KmX7pHwZxNQPyg_-cIit49Dkms1vIPli9bODOwaxdxqozQ_vKjG91g5SIyOjGgI4efRqfOclo0wacnHfV0-QBsUn-Lc9as0uOcw
```

### Retrieve all devices using Python

Next, let's create a script that implements the `network-device` REST call. We will call the `get_token` function to retrieve the token which we will then use in the headers section (as per the DNAC API documentation). What follows is merely a call to the required API (in this case `/dna/intent/api/v1/network-device`) and store the response into a list which we can loop through.

```python
import requests
from authenticate import get_token
from pprint import pprint


def main():
   dnac = "sandboxdnac2.cisco.com"

   token = get_token(dnac)

   url = f"https://{dnac}/dna/intent/api/v1/network-device"

   headers = {
      "Content-Type": "application/json",
      "Accept": "application/json",
      "X-auth-Token": token 
   }

   response =  requests.get(url, headers=headers, verify=False ).json()
   devices = response["response"]

   for device in devices:
      if device['type'] is not None:
         print(f"{device['type']} with serial number {device['serialNumber']}")

if __name__ == "__main__":
   main()
```
This will result in the following output.
```
WAUTERW-M-65P7:Devices wauterw$ python get_devicelist.py 
Cisco 3504 Wireless LAN Controller with serial number FCW2218M0B1
Cisco Catalyst 9300 Switch with serial number FCW2214L0VK
Cisco Catalyst 9300 Switch with serial number FCW2214L0UZ
Cisco Catalyst38xx stack-able ethernet switch with serial number FCW2212D05S
Cisco 1140 Unified Access Point with serial number 1140K0001
Cisco 1140 Unified Access Point with serial number 1140K0010
Cisco 1140 Unified Access Point with serial number 1140K0002
Cisco 1140 Unified Access Point with serial number 1140K0003
Cisco 1140 Unified Access Point with serial number 1140K0004
Cisco 1140 Unified Access Point with serial number 1140K0005
Cisco 1140 Unified Access Point with serial number 1140K0006
Cisco 1140 Unified Access Point with serial number 1140K0007
Cisco 1140 Unified Access Point with serial number 1140K0008
Cisco 1140 Unified Access Point with serial number 1140K0009
```

### Retrieve interfaces from device using Python

In the above output, you see we get a list of both wired as well as wireless devices Next, we'll write a script to retrieve the interfaces from a specific device category. This requires us to work with two different APIs. Let's look into it.

In the `Devices`, you will see the following API. As you look into it, you'll see we can use it to retrieve a list of interfaces if we pass the device ID.

![DNAC](/images/2020-04-24-1.png)

In the below Python script, you notice we do the following actions:
- Query the network-device API with a family query set to "Switches and Hubs". Look at the API to verify this, you'll see the possibility to pass some query parameters, of which `family` is one example. We will get back a list of wired devices but not the access points. 
- Next, we iterate over that list of devices and store their ID into a list.
- Next, we loop through the list of device id's and for each one, we call the `interface/network-device/{device-id}` API (see screenshot above).
- The result of the previous call is a list of interfaces belonging to that device(-id)
- We just loop over the interfaces and print out the name and the IP address.

```python
import requests
from authenticate import get_token
from pprint import pprint

def main():
   token = get_token()

   dnac = "sandboxdnac2.cisco.com"
   url = f"https://{dnac}/dna/intent/api/v1/"
   family = "Switches and Hubs"

   headers = {
      "Content-Type": "application/json",
      "Accept": "application/json",
      "X-auth-Token": token 
   }

   device_url = url + "network-device?family=" +  family
   response =  requests.get(device_url, headers=headers, verify=False ).json()
   devices = response["response"]

   device_list = []

   for device in devices:
      print(f"{device['type']} with ID {device['id']}")
      device_list.append(device['id'])
 
   for device_id in device_list:
      print("Investigating device: " + device_id)
      new_interface_url = url + "interface/network-device/" + device_id
      response =  requests.get(new_interface_url, headers=headers, verify=False ).json()
      interfaces = response["response"]
      for interface in interfaces:
         if interface['ipv4Address'] is not None:
            print(f"    {interface['portName']} with IP address {interface['ipv4Address']}")
      
if __name__ == "__main__":
   main() 
```
Below you can see the output of this script. As you expect, we simply get back a list of devices with its related interfaces.

```
wauterw@WAUTERW-M-65P7 Devices % python3 get_interfaces.py
Investigating device: cda508ff-bca4-4b96-8e17-9d3c603cb628
    Loopback0 with IP address 10.2.2.3
    Vlan823 with IP address 10.10.20.81
Investigating device: 5bc5b967-3f83-4195-891c-788f3e9048f3
    Loopback0 with IP address 10.2.2.4
    Vlan823 with IP address 10.10.20.82
Investigating device: 2f0b7d3b-c9e1-491e-a584-f272b5403719
    Loopback0 with IP address 10.2.2.2
    Loopback1 with IP address 11.11.11.11
    Vlan823 with IP address 10.10.20.80
```

The code can be found [here](https://github.com/wiwa1978/blog-hugo-netlify-code/tree/master/DNAC_PythonRequests/Devices).