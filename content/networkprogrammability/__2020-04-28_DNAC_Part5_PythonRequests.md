---
title: Cisco DNA Center - Discovery and Command runner
date: 2020-04-28T10:32:50+01:00
draft: false
categories:
  - Network Programming
  - Programming
  - All
tags:
  - DNAC
  - Python
---
### Introduction


### Device Discovery

###### Get Credentials

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