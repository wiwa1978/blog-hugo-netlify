---
title: Cisco DNA Center - Discovery (Python)
date: 2020-05-01T10:32:50+01:00
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

- Part 1: [Getting started](https://blog.wimwauters.com/networkprogrammability/2020-04-22_dnac_part1_gettingstarted/)
- Part 2: [Cisco DNA Center - Devices](https://blog.wimwauters.com/networkprogrammability/2020-04-24_dnac_part2_pythonrequests/)
- Part 3: [Cisco DNA Center - Assurance](https://blog.wimwauters.com/networkprogrammability/2020-04-25_dnac_part3_pythonrequests/)
- Part 4: [Cisco DNA Center - Sites](https://blog.wimwauters.com/networkprogrammability/2020-04-27_dnac_part3_pythonrequests/)
- Part 5 (this post): [Cisco DNA Center - Discovery (POSTMAN)](https://blog.wimwauters.com/networkprogrammability/2020-04-29_dnac_part5_postman_networkdiscovery/)
- Part 6: [Cisco DNA Center - Discovery (Python)](https://blog.wimwauters.com/networkprogrammability/2020-05-01_dnac_part6_pythonrequests/)
- Part 7: [Cisco DNA Center - CommandRunner (Python)](https://blog.wimwauters.com/networkprogrammability/2020-05-02_dnac_part7_pythonrequests/)
- Part 8: [Cisco DNA Center - Flow-Analysis (POSTMAN)](https://blog.wimwauters.com/networkprogrammability/2020-05-03_dnac_part8_postman_flowanalysis/)
- Part 9: [Cisco DNA Center - Flow-Analysis (Python)](https://blog.wimwauters.com/networkprogrammability/2020-05-04_dnac_part9_pythonrequests_flowanalysis/)

>Disclaimer: the code in this post is not production-grade code obviously.  The examples in the post are merely conceptual and for informational purposes.

### Introduction
 In [this](https://blog.wimwauters.com/networkprogrammability/2020-04-29_dnac_part4_postman_networkdiscovery/) we have been showing some POSTMAN samples for running a Network Discovery and Command Runner. In the next sections, we will explore how we can achieve exactly the same using Python Requests. I recommend you to first go through the POSTMAN post before attempting this one.

### Note about equipment

In this post, I'm using my own DNAC in my lab. However, if you want to follow along, you could also use a Cisco sandbox environment delivered by [Cisco Devnet](https://developer.cisco.com). To get a list of all sandboxes, check out [this](https://devnetsandbox.cisco.com/) link. For this tutorial, you could use [this](https://devnetsandbox.cisco.com/RM/Diagram/Index/b8d7aa34-aa8f-4bf2-9c42-302aaa2daafb?diagramType=Topology) one. Note that this is a reservable instance as the always-on is restricted in functionality.

### Network Discovery
DNAC allows you to discover devices in your network by scanning the network based on a given IP range or through CDP. Let’s see how the IP range based discovery works.

###### Get Credentials
The following Python snippet will call the `/dna/intent/api/v1/global-credential` API endpoint. You will see that we add a query parameter called `credentialSubType: CLI`. Once we issue the request, we will parse the response and add the credentials (id) to a Python list.

```python
import requests
from authenticate import get_token
import json, time
from jinja2 import Environment
from jinja2 import FileSystemLoader

dnac = "10.48.82.183"
token = get_token(dnac)
url = f"https://{dnac}"

headers = {
      "Content-Type": "application/json",
      "Accept": "application/json",
      "X-auth-Token": token 
   }

def main():
   cred_url = "/api/v1/global-credential"
   credtype = "CLI"
   
   params = {
      "credentialSubType": {credtype}
   }

   discoveryname = "newDiscovery_v1" #needs to be updated every run

   # Part 1: Get Credentials to run the discovery
   cred_list = []
   response =  requests.get(url + cred_url, params=params, headers=headers, verify=False ).json()
   cred_list.append(response["response"][0]["id"])
```
Note: the discovery name needs to be different every run. I did not implement code to check if the name was already used.

###### Start IP based discovery
Next, we will create the following Jinja2 template. This contains the sample JSON body for our API request.

```json
{
   "name": "{{ name }}",
   "discoveryType": "Range",
   "ipAddressList": "192.168.30.0-192.168.30.100",
   "timeOut": 1,
   "protocolOrder": "ssh,telnet",
   "preferredMgmtIPMethod": "None"
}
```
In the following Python snippet, we will get the Jinja2 template and pass in the `discoveryName` variable dynamically. Note that the Jinja2 template does not contain a `globalCredentialIdList` while the documentation requires this. Given that this list of credentials can change I decided not to hardcode them into the Jinja2 file but rather set them in Python by adding the key `globalCredentialIdList` and have it point to the list of credentials we populated in the previous section.

```python
# Part 2: read in the discovery template
   jinja_templates = Environment(loader=FileSystemLoader('templates'), trim_blocks=True)
   template = jinja_templates.get_template("discovery.j2.json")
   payload = template.render(name=discoveryname)
   discovery = json.loads(payload)
   discovery["globalCredentialIdList"] = cred_list
```

To actually start the discovery, we will call the `dna/intent/v1/discovery` API. A quick note, in the lab DNAC I'm using I need to use `api/v1/discovery` instead of `dna/intent/v1/discovery`. Not really sure why, I think it's a bug. In your code, you should be able to use `dna/intent/v1/discovery`. 

```python
   # Part 3: Run the discovery
   discover_url = f"https://{dnac}/api/v1/discovery"
   # discovery_url = f""https://{dnac}/dna/intent/v1/discovery"
   response_discovery =  requests.post(discover_url, data=json.dumps(discovery), headers=headers, verify=False ).json()
   task_url = response_discovery['response']['url']
```

######  Check Task
As we know from [previous](http://blog.wimwauters.com/networkprogrammability/2020-04-29_dnac_part4_postman_networkdiscovery/) blog, the discovery endpoint will return a task id which we will have to pull frequently until we get back the response. 

```python
task = waitTask(url, task_url )
```

Therefore, I implemented a `waitTask` function that essentually checks every second the `tasks` API and checks the response. If there is an endTime in the response then we can finish the polling.

```python
def waitTask(url, task_url):
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
You will see we get back a number in the progress attribute. This essentially is the ID of the discovery job itself. We store this in a variable called `discoverId`.

```python
discoverId = task['response']['progress']
```

######  Get Discovery status

Next, using the discoverId, we can query the `/dna/intent/api/v1/discovery/{id}` API endpoint. Again, in the lab DNAC I'm using I need to use `api/v1/discovery` instead of `dna/intent/v1/discovery`. Not really sure why, I think it's a bug. In your code, you should be able to use `dna/intent/v1/discovery`. 

In any case, in below snippet we will check the `discoveryCondition` parameter and once it's `Complete`, we will print out the number of devices.

```python
# Part 5: Get Discovery Status
   discover_url = f"https://{dnac}/api/v1/discovery/{discoverId}"
   # discover_url =  f"https://{dnac}/dna/intent/api/v1/discovery/{id}

   while True:
      response =  requests.get(discover_url, headers=headers, verify=False ).json()
      
      if response['response']['discoveryCondition'] == "Complete":
         print(f"Discovery with id {discoverId} completed successfully")
         print(f"Discovery found {response['response']['numDevices']} devices") 
```

######  Get Discovered devices 
The point of this exercise is of course to retrieve the list of discovered devices so we will issue a request to `api/v1/discovery/{discoverId}/network-device`. We will loop over the devices in the response and print out the hostname and the management IP address.

```python
# Part 6: Get Discovered Devices
         devices_url = f"https://{dnac}/api/v1/discovery/{discoverId}/network-device"
         # devices_url =  f"https://{dnac}/dna/intent/api/v1/discovery/{discoverId}/network-device

         response_devices =  requests.get(devices_url, headers=headers, verify=False ).json()
        
         for device in response_devices['response']:
            print(f"Device name: {device['hostname']} with IP address {device['managementIpAddress']}")

         break
```
Run the script and you will see indeed the 4 devices with their associated IP addresses.

```bash
wauterw@WAUTERW-M-65P7 NetworkDiscovery % python3 run_discovery.py
Discovery with id 1081 completed successfully
Discovery found 4 devices
Device name: csrv-GB.sda.lab with IP address 192.168.30.11
Device name: 3850-1B1.sda.lab with IP address 192.168.30.60
Device name: 3850-1B2.sda.lab with IP address 192.168.30.61
Device name: c2901-peg3-2.sda.lab with IP address 192.168.30.49
```

I shared the code in bits and pieces, if you want to see the final Python script, check out the Github repo [here](https://github.com/wiwa1978/blog-hugo-netlify-code/tree/master/DNAC_PythonRequests/NetworkDiscovery).

