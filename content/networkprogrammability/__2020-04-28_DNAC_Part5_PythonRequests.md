---
title: Cisco DNA Center - Discovery and Command runner
date: 2020-04-28T10:32:50+01:00
draft: true
categories:
  - Network Programming
  - Programming
  - All
tags:
  - DNAC
  - Python
---
### Introduction
 In [this](https://blog.wimwauters.com/networkprogrammability/2020-04-29_dnac_part4_postman_networkdiscovery/) we have been showing some POSTMAN samples for running a Network Discovery and Command Runner. In the next sections, we will explore how we can achieve exactly the same using Python Requests. I recommend you to first go through the POSTMAN post before attempting this one.

### Note about equipment

In this post, I'm using my own DNAC in my lab. However, if you want to follow along, you could also use a Cisco sandbox environment delivered by [Cisco Devnet](https://developer.cisco.com). To get a list of all sandboxes, check out [this](https://devnetsandbox.cisco.com/) link. For this tutorial, you could use [this](https://devnetsandbox.cisco.com/RM/Diagram/Index/b8d7aa34-aa8f-4bf2-9c42-302aaa2daafb?diagramType=Topology) one. Note that this is a reservable instance as the always-on is restricted in functionality.

### Device Discovery
DNAC allows you to discover devices in your network by scanning the network based on a given IP range or through CDP. Let’s see how the IP range based discovery works.

###### Get Credentials
The following Python snippet

```python
import requests
from authenticate import get_token
from pprint import pprint
import json, time
from jinja2 import Environment
from jinja2 import FileSystemLoader

def main():
   dnac = "10.48.82.183"
   token = get_token(dnac)

   url = f"https://{dnac}"

   cred_url = "/api/v1/global-credential"
   credtype = "CLI"
   
   params = {
      "credentialSubType": {credtype}
   }

   headers = {
      "Content-Type": "application/json",
      "Accept": "application/json",
      "X-auth-Token": token 
   }

   discoveryname = "newDiscovery_v1"

   # Part 1: Get Credentials to run the discovery
   cred_list = []
   response =  requests.get(url + cred_url, params=params, headers=headers, verify=False ).json()
   cred_list.append(response["response"][0]["id"])
```


###### Start IP based discovery

```python
# Part 2: read in the discovery template
   jinja_templates = Environment(loader=FileSystemLoader('templates'), trim_blocks=True)
   template = jinja_templates.get_template("discovery.j2.json")
   payload = template.render(name=discoveryname)
   discovery = json.loads(payload)
   discovery["globalCredentialIdList"] = cred_list
```


```python
   # Part 3: Run the discovery
   discover_url = f"https://{dnac}/api/v1/discovery"
   response_discovery =  requests.post(discover_url, data=json.dumps(discovery), headers=headers, verify=False ).json()
   task_url = response_discovery['response']['url']
   #print(f"taskUrl => {task_url}")
   #print(f"taskId => {response_discovery['response']['taskId']}")
```

######  Check Task

```python
task = waitTask(url, task_url )
discoverId = task['response']['progress']
```


```python
def waitTask(url, task_url):
   dnac = "10.48.82.183"
   token = get_token(dnac)
   headers = {
      "Content-Type": "application/json",
      "Accept": "application/json",
      "X-auth-Token": token 
   }
   for i in range(10):
      time.sleep(1)
      response_task =  requests.get(url + task_url, headers=headers, verify=False ).json()
      print(response_task)
      if response_task['response']['isError']:
         print("Error")
      if "endTime" in response_task['response']:
         print(response_task['response']['progress'])
         return response_task
```

######  Get Discovery status

```python
# Part 5: Get Discovery Status
   discover_url = f"https://{dnac}/api/v1/discovery/{discoverId}"

   while True:
      response =  requests.get(discover_url, headers=headers, verify=False ).json()
      
      if response['response']['discoveryCondition'] == "Complete":
         print(f"Discovery with id {discoverId} completed successfully")
         print(f"Discovery found {response['response']['numDevices']} devices") 
```

######  Get Discovered devices 

```python
# Part 6: Get Discovered Devices
         devices_url = f"https://{dnac}/api/v1/discovery/{discoverId}/network-device"
         response_devices =  requests.get(devices_url, headers=headers, verify=False ).json()
        
         for device in response_devices['response']:
            print(f"Device name: {device['hostname']} with IP address {device['managementIpAddress']}")

         break
```

```bash
wauterw@WAUTERW-M-65P7 NetworkDiscovery % python3 run_discovery.py
Discovery with id 1081 completed successfully
Discovery found 4 devices
Device name: csrv-GB.sda.lab with IP address 192.168.30.11
Device name: 3850-1B1.sda.lab with IP address 192.168.30.60
Device name: 3850-1B2.sda.lab with IP address 192.168.30.61
Device name: c2901-peg3-2.sda.lab with IP address 192.168.30.49
```

[here](https://github.com/wiwa1978/blog-hugo-netlify-code/tree/master/DNAC_PythonRequests/NetworkDiscovery)

### Command Runner

Code needs to be updated to retrieve the file id and parse that one.


[here](https://github.com/wiwa1978/blog-hugo-netlify-code/tree/master/DNAC_PythonRequests/CommandRunner)