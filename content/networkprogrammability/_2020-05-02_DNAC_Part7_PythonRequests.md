---
title: Cisco DNA Center - Command Runner (Python)
date: 2020-05-02T10:32:50+01:00
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
- Part 4: [Cisco DNA Center - Sites](https://blog.wimwauters.com/networkprogrammability/2020-04-27_dnac_part4_pythonrequests/)
- Part 5: [Cisco DNA Center - Discovery (POSTMAN)](https://blog.wimwauters.com/networkprogrammability/2020-04-29_dnac_part5_postman_networkdiscovery/)
- Part 6: [Cisco DNA Center - Discovery (Python)](https://blog.wimwauters.com/networkprogrammability/2020-05-01_dnac_part6_pythonrequests/)
- Part 7 (this post): [Cisco DNA Center - CommandRunner (Python)](https://blog.wimwauters.com/networkprogrammability/2020-05-02_dnac_part7_pythonrequests/)
- Part 8: [Cisco DNA Center - Flow-Analysis (POSTMAN)](https://blog.wimwauters.com/networkprogrammability/2020-05-03_dnac_part8_postman_flowanalysis/)
- Part 9: [Cisco DNA Center - Flow-Analysis (Python)](https://blog.wimwauters.com/networkprogrammability/2020-05-04_dnac_part9_pythonrequests_flowanalysis/)

>Disclaimer: the code in this post is not production-grade code obviously.  The examples in the post are merely conceptual and for informational purposes.

### Introduction
 In [this](https://blog.wimwauters.com/networkprogrammability/2020-04-29_dnac_part5_postman_networkdiscovery/) blog post, we have been showing some POSTMAN samples for running a Network Discovery and Command Runner. In the next sections, we will explore how we can implement the CommandRunner functionality using Python Requests. I recommend you to first go through the POSTMAN post before attempting this one.

### Note about equipment

In this post, I'm using my own DNAC in my lab. However, if you want to follow along, you could also use a Cisco sandbox environment delivered by [Cisco Devnet](https://developer.cisco.com). To get a list of all sandboxes, check out [this](https://devnetsandbox.cisco.com/) link. For this tutorial, you could use [this](https://devnetsandbox.cisco.com/RM/Diagram/Index/b8d7aa34-aa8f-4bf2-9c42-302aaa2daafb?diagramType=Topology) one. Note that this is a reservable instance as the always-on is restricted in functionality.

### CommandRunner

See also the previous [post](https://blog.wimwauters.com/networkprogrammability/2020-04-29_dnac_part5_postman_networkdiscovery/) towards the bottom to see the same flow using Postman screenshots.

###### Get list of devices
First, we will retrieve the list of devices onto which we would like to run our command. So we will call the `/api/v1/network-device` endpoint but we'll pass in a `family` parameter as we only want to execute the command on devices in the 'switches' family. We will store all device id's in a list as we will need to pass that list in the JSON body of the Command Runner API. The command we will run against the devices in our list, will be `show ip interface brief`.

A quick note, in the lab DNAC I’m using I need to use /api/v1/network-device instead of /dna/intent/api/v1/network-device. Not really sure why, I think it’s a bug. In your code, you should be able to use /dna/intent/api/v1/network-device without any issue.

```python
def main():
    family = "Switches and Hubs"

    device_url = url + "/api/v1/network-device?family=" +  family
    response =  requests.get(device_url, headers=headers, verify=False ).json()
    devices = response["response"]

    device_list = []

    # Store all device ids in a list
    for device in devices:
        device_list.append(device['id'])

    # Pass the device list into the payload
    payload = {
        "commands": [
            "show ip interface brief"
        ],
        "deviceUuids": device_list
    }
```
###### Run command

Next, let's actually run the command. We'll need to do a POST request to the `/api/v1/network-device-poller/cli/read-request` endpoint and pass along the JSON body we created earlier. This JSON body contained the command to execute as well as the list of device (id's) to execute the command against.

A quick note, in the lab DNAC I’m using I need to use /api/v1/network-device-poller/cli/read-request instead of /dna/intent/v1/network-device-poller/cli/read-request. Not really sure why, I think it’s a bug. In your code, you should be able to use /dna/intent/v1/network-device-poller/cli/read-request without any issue.

We will get back a (task) url as part of the response:

```python
    command_url = url + "/api/v1/network-device-poller/cli/read-request"
    response = requests.post(command_url, headers=headers, data=json.dumps(payload), verify=False ).json()
```

###### Check task
Also this API is asynchonous, so we will get back a response containing the `taskId` and the `url` which we can use for further processing.

```python
    task_url = response['response']['url']
    task = waitTask(url, task_url )
    fileId = json.loads(task['response']['progress'])
```
Therefore, I implemented a `waitTask` function that essentually checks every second the `tasks` API and checks the response. If there is an endTime in the response then we can finish the polling.

```python
def waitTask(url, task_url):
   for i in range(10):
      time.sleep(1)
      response_task =  requests.get(url + task_url, headers=headers, verify=False ).json()
      if response_task['response']['isError']:
         print("Error")
      if "endTime" in response_task['response']:
         return response_task
```

###### Check result
The tasks API will provide a response with a fileId. This fileId points to a file containing the output of the command. 

```python
processFile(url, fileId['fileId'])
```
I have created a `processFile` method that calls the `/dna/intent/api/v1/file/{fileId}` endpoint.
```python
def processFile(url, fileid):
    file_url = url + f"/api/v1/file/{fileid}"
    print(f"FileURL: {file_url}")
    response = requests.get(file_url, headers=headers, verify=False ).json()
    print(response[0]['commandResponses']['SUCCESS']['show ip interface brief'])
```
This API will show the results of our command. See below for an example:

```bash
wauterw@WAUTERW-M-65P7 CommandRunner % python3 commandrunner.py
The file id is 9f6b1061-c8d4-47cb-804a-082415f3495e
FileURL: https://10.48.82.183/dna/intent/api/v1/file/9f6b1061-c8d4-47cb-804a-082415f3495e
show ip interface brief
Interface              IP-Address      OK? Method Status                Protocol
Vlan1                  unassigned      YES manual up                    up      
Vlan23                 192.168.23.60   YES NVRAM  up                    up      
Vlan3011               192.168.11.25   YES manual up                    up      
Vlan3012               192.168.11.29   YES manual up                    up      
Vlan3013               192.168.11.33   YES manual up                    up      
GigabitEthernet0/0     10.48.172.60    YES NVRAM  up                    up      
GigabitEthernet1/0/1   192.168.13.66   YES manual up                    up      
GigabitEthernet1/0/2   unassigned      YES unset  administratively down down    
GigabitEthernet1/0/3   unassigned      YES unset  down                  down    
GigabitEthernet1/0/4   unassigned      YES unset  down                  down    
GigabitEthernet1/0/5   unassigned      YES unset  down                  down    
GigabitEthernet1/0/6   unassigned      YES unset  down                  down    
GigabitEthernet1/0/7   unassigned      YES unset  down                  down    
GigabitEthernet1/0/8   unassigned      YES unset  down                  down    
GigabitEthernet1/0/9   unassigned      YES unset  down                  down    
GigabitEthernet1/0/10  unassigned      YES unset  down                  down    
GigabitEthernet1/0/11  unassigned      YES unset  down                  down    
GigabitEthernet1/0/12  unassigned      YES unset  down                  down    
GigabitEthernet1/0/13  unassigned      YES unset  down                  down    
GigabitEthernet1/0/14  unassigned      YES unset  down                  down    
GigabitEthernet1/0/15  unassigned      YES unset  down                  down    
GigabitEthernet1/0/16  unassigned      YES unset  down                  down    
GigabitEthernet1/0/17  unassigned      YES unset  down                  down    
GigabitEthernet1/0/18  unassigned      YES unset  down                  down    
GigabitEthernet1/0/19  unassigned      YES unset  down                  down    
GigabitEthernet1/0/20  unassigned      YES unset  down                  down    
GigabitEthernet1/0/21  unassigned      YES unset  down                  down    
GigabitEthernet1/0/22  unassigned      YES unset  up                    up      
GigabitEthernet1/0/23  unassigned      YES unset  up                    up      
GigabitEthernet1/0/24  unassigned      YES unset  up                    up      
GigabitEthernet1/1/1   unassigned      YES unset  up                    up      
GigabitEthernet1/1/2   unassigned      YES unset  down                  down    
GigabitEthernet1/1/3   unassigned      YES unset  down                  down    
GigabitEthernet1/1/4   unassigned      YES unset  up                    up      
Te1/1/1                unassigned      YES unset  down                  down    
Te1/1/2                unassigned      YES unset  down                  down    
Te1/1/3                unassigned      YES unset  down                  down    
Te1/1/4                unassigned      YES unset  down                  down    
LISP0                  unassigned      YES unset  up                    up      
LISP0.4097             192.168.30.60   YES unset  up                    up      
LISP0.4099             172.16.100.1    YES unset  up                    up      
LISP0.4100             192.168.11.29   YES unset  up                    up      
LISP0.4101             unassigned      YES unset  deleted               down    
Loopback0              192.168.30.60   YES NVRAM  up                    up      
Loopback1042           172.16.100.1    YES manual up                    up      
Loopback1044           172.16.96.1     YES manual up                    up      
Loopback1045           172.16.99.1     YES manual up                    up      
Loopback2045           192.168.18.1    YES manual up                    up      
```
I shared the code in bits and pieces, if you want to see the final Python script, check out the Github repo [here](https://github.com/wiwa1978/blog-hugo-netlify-code/tree/master/DNAC_PythonRequests/CommandRunner).