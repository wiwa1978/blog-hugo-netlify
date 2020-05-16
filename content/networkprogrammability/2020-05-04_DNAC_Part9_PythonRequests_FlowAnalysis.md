---
title: Cisco DNA Center - Flow-Analysis (Python)
date: 2020-05-04T06:32:50+01:00
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
- Part 2: [Cisco DNA Center - Devices](https://blog.wimwauters.com/networkprogrammability/2020-04-24_dnac_part1_pythonrequests/)
- Part 3: [Cisco DNA Center - Assurance](https://blog.wimwauters.com/networkprogrammability/2020-04-25_dnac_part2_pythonrequests/)
- Part 4: [Cisco DNA Center - Sites](https://blog.wimwauters.com/networkprogrammability/2020-04-27_dnac_part3_pythonrequests/)
- Part 5: [Cisco DNA Center - Discovery (POSTMAN)](https://blog.wimwauters.com/networkprogrammability/2020-04-29_dnac_part4_postman_networkdiscovery/)
- Part 6 (this post): [Cisco DNA Center - Discovery (Python)](https://blog.wimwauters.com/networkprogrammability/2020-05-01_dnac_part5_pythonrequests/)
- Part 7: [Cisco DNA Center - CommandRunner (Python)](https://blog.wimwauters.com/networkprogrammability/2020-05-02_dnac_part6_pythonrequests/)

>Disclaimer: the code in this post is not production-grade code obviously.  The examples in the post are merely conceptual and for informational purposes.

### Introduction
 In [this](https://blog.wimwauters.com/networkprogrammability/2020-05-03_DNAC_Part8_Postman_FlowAnalysis) we have been showing some POSTMAN samples for running a Flow Analysis. In the next sections, we will explore how we can achieve exactly the same using Python Requests. I recommend you to first go through the POSTMAN post before attempting this one.

### Note about equipment

In this post, I'm using my own DNAC in my lab. However, if you want to follow along, you could also use a Cisco sandbox environment delivered by [Cisco Devnet](https://developer.cisco.com). To get a list of all sandboxes, check out [this](https://devnetsandbox.cisco.com/) link. For this tutorial, you could use [this](https://devnetsandbox.cisco.com/RM/Diagram/Index/b8d7aa34-aa8f-4bf2-9c42-302aaa2daafb?diagramType=Topology) one. Note that this is a reservable instance as the always-on is restricted in functionality.

### Flow Analysis

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

def flowanalysis():
    payload = {
        "sourceIP": "192.168.30.60",
        "destIP": "192.168.13.64",
        "inclusions": [
            "INTERFACE-STATS",
            "DEVICE-STATS",
            "QOS-STATS"
        ],
        "controlPath": False,
        "periodicRefresh": False
    }
      
    flow_url = f"https://{dnac}/api/v1/flow-analysis"
    response_flow =  requests.post(flow_url, data=json.dumps(payload), headers=headers, verify=False ).json()
    analysis_url = response_flow['response']['url']
    
    response =  requests.get(url + analysis_url, headers=headers, verify=False ).json()
    print(f"{response['response']['request']['status']} => Reason: {response['response']['request']['failureReason']}")

if __name__ == "__main__":
   flowanalysis()

```