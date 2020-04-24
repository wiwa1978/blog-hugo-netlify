---
title: DNA Center Introduction - Python
date: 2020-04-24T12:32:50+01:00
draft: True
categories:
  - Network Programming
  - Programming
tags:
  - DNAC
---
### Introduction
In this [post](https://blog.wimwauters.com/networkprogrammability/2020-04-22_dnac_gettingstarted/), we introduced DNAC at a fairly high level and we have shown some POSTMAN samples to get a basic understanding of some of the APIs like authentication and devices). In this post, we will continue that journey as we will show some  Python samples.

>Disclaimer: the code in this post is not production-grade code obviously. One should never store the username and password in the clear, not in the source code itself. The examples in the post are merely conceptual and for informational purposes.


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

