---
title: Cisco DNA Center - Getting Started
date: 2020-04-22T08:32:50+01:00
draft: false
categories:
  - Network Programming
  - Programming
tags:
  - DNAC
---
### What is Cisco DNA

Cisco DNA is Cisco’s architecture for enterprise networks – across the campus, branch, WAN, and extended enterprise. It provides an open, extensible, and software-driven approach that makes the network simpler to manage and more agile and responsive to business needs. Cisco DNA is an intelligent system that encompasses policy, automation, analytics, and open platform capabilities to deliver on all required aspects of an intent-based network.

That's a mouth full of marketing terms but in my own words it boils down to: Cisco DNA allows you to provision and configure all your network devices in minutes while it uses advanced analytics to proactively monitor and optimize your network. Read more info on DNA [here](https://www.cisco.com/c/en/us/solutions/collateral/enterprise-networks/cisco-digital-network-architecture/nb-06-cisco-dna-soln-ovw-cte-en.html) and [this](https://www.cisco.com/c/dam/global/ru_kz/solutions/enterprise-networks/digital-network-architecture/pdf/white-paper-c11-736842.pdf) whitepaper.

As this is a blog about (network programmabily) more than anything else we'll focus more on its API.

### DNA API

Cisco DNA architecture comes with a controller, which is called DNAC (Digital Network Architecture Center). Not only does it allow us to design and provision our network, it also allows for monitoring and analytics. And above all, it is an open and extensible platform to allow integration via its API (documentation [here](https://developer.cisco.com/docs/dna-center/api/1-3-3-x/)).

### Overview of the API

Cisco DNAC comes with an intend API, which is a Northbound API that exposes specific capabilities of the DNA Center platform. Reading the API can be a bit overwhelming so have a look at the below categories to understand the several subsections of the API ([source](https://developer.cisco.com/docs/dna-center/#!cisco-dna-center-platform-overview/intent-api-northbound)). Each category below corresponds to a particular section in the [API](https://developer.cisco.com/docs/dna-center/api/1-3-3-x/))
 
**Domain**
- Sites 
- Topology
- Devices
- Client
- Users
- Issues

**Site Management**
- Site Design
- Network Settings
- Software Image Management
- Configuration Templates

**Connectivity**
- Fabric Wired
- Non-Fabric Wireless

**Operational APIs**
- Command Runner
- Network Discovery => seperate blog post (Part 2)
- Path Trace
- File
- Task
- Tags

**Policy**
- Application Policy

**Event Management**
- Event Management

### Getting started example (POSTMAN)

Enough intro...let's dive into some basic programming. Let's try this with POSTMAN first. 

> Note about equipment: For all the examples that involve retrieving information, we will use a Cisco sandbox environment delivered by Cisco Devnet. Go check out Devnet, really brilliant. In this tutorial, I'm using the DNAC Always On sandbox. For later tutorials, we will use a different DNAC as the always-on variant does not allow us to create/update information on the DNAC.

##### A) Preparing POSTMAN

Let's first configure our POSTMAN client a bit. As we are using the DEVNET DNAC Always-On sandbox, we need to set the credentials in POSTMAN. This allows us to use placeholders in our calls, so we can write things like `https://{dnac}` instead of `https://sandboxdnac2.cisco.com` which makes it much more flexible to work with other DNAC instances later on.

![DNAC](/images/2020-04-22-5.png)

##### B) Authentication

As a first step, we need to authenticate to the DNAC. Below is a screenshot from the API documentation. We need to issue a REST call towards the endpoint `/dna/system/api/v1/auth/token` and we will get back a `Token`.

![DNAC](/images/2020-04-22-1.png)

In the below screenshot, you'll see the endpoint set to `/dna/system/api/v1/auth/token`. For the Authorization, we need to use Basic Auth Base64 encoding according the the documentation. This is easy with POSTMAN, as we set the `Type` to Basic Auth and it'll work.

![DNAC](/images/2020-04-22-2.png)

As headers, simply use:

![DNAC](/images/2020-04-22-3.png)

Let's run this call, and you'll see we get a `Token` returned which we need to use in subsequent queries.

![DNAC](/images/2020-04-22-4.png)

As an extra, let's capture the token via some Javascript call and store it in the `api_token` environment variable, so we don't always need to manually update the token once it expires. Store the following code in the `Tests` tab in POSTMAN.

![DNAC](/images/2020-04-22-6.png)

##### C) Get Devices
Now, we have the token, we can easily use it for any subsequent REST call. Let's try it out through a simple call. For this example, we'll use the `Get Device list` API call. The documentation says to use `/dna/intent/api/v1/network-device`. Take a look at below screenshot.

![DNAC](/images/2020-04-22-7.png)

### Getting started example (Python)

>Disclaimer: the code in this post is not production-grade code obviously. One should never store the username and password in the clear, not in the source code itself. The examples in the post are merely conceptual and for informational purposes.

Let's now do exactly the same but through Python. It's straigthforward and boils down to understanding the Python requests linrary. We'll have to use the `POST verb` and pass in the username/password (the auth function is documented [here](https://requests.readthedocs.io/en/master/user/authentication/)).

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
Hope you found this a bit useful. In next posts, I'll dive a bit deeper into DNAC API and provide some simple code for some DNA use cases. Stay tuned!