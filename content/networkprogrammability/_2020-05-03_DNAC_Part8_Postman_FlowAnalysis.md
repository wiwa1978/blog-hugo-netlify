---
title: Cisco DNA Center - Flow-Analysis (POSTMAN)
date: 2020-05-03T06:32:50+01:00
draft: false
categories:
  - Network Programming
  - Programming
  - All
tags:
  - DNAC
  - POSTMAN
---
### DNAC Series

This is part of a DNAC series:

- Part 1: [Getting started](https://blog.wimwauters.com/networkprogrammability/2020-04-22_dnac_part1_gettingstarted/)
- Part 2: [Cisco DNA Center - Devices](https://blog.wimwauters.com/networkprogrammability/2020-04-24_dnac_part2_pythonrequests/)
- Part 3: [Cisco DNA Center - Assurance](https://blog.wimwauters.com/networkprogrammability/2020-04-25_dnac_part3_pythonrequests/)
- Part 4: [Cisco DNA Center - Sites](https://blog.wimwauters.com/networkprogrammability/2020-04-27_dnac_part4_pythonrequests/)
- Part 5: [Cisco DNA Center - Discovery (POSTMAN)](https://blog.wimwauters.com/networkprogrammability/2020-04-29_dnac_part5_postman_networkdiscovery/)
- Part 6: [Cisco DNA Center - Discovery (Python)](https://blog.wimwauters.com/networkprogrammability/2020-05-01_dnac_part6_pythonrequests/)
- Part 7: [Cisco DNA Center - CommandRunner (Python)](https://blog.wimwauters.com/networkprogrammability/2020-05-02_dnac_part7_pythonrequests/)
- Part 8 (this post): [Cisco DNA Center - Flow-Analysis (POSTMAN)](https://blog.wimwauters.com/networkprogrammability/2020-05-03_dnac_part8_postman_flowanalysis/)
- Part 9: [Cisco DNA Center - Flow-Analysis (Python)](https://blog.wimwauters.com/networkprogrammability/2020-05-04_dnac_part9_pythonrequests_flowanalysis/)

### Introduction
 In [this](https://blog.wimwauters.com/networkprogrammability/2020-04-29_dnac_part4_postman_networkdiscovery/) we have been showing some POSTMAN samples for running a Network Discovery and Command Runner. In the next sections, we will explore how we can achieve exactly the same using Python Requests. I recommend you to first go through the POSTMAN post before attempting this one.

### Note about equipment

In this post, I'm using my own DNAC in my lab. However, if you want to follow along, you could also use a Cisco sandbox environment delivered by [Cisco Devnet](https://developer.cisco.com). To get a list of all sandboxes, check out [this](https://devnetsandbox.cisco.com/) link. For this tutorial, you could use [this](https://devnetsandbox.cisco.com/RM/Diagram/Index/b8d7aa34-aa8f-4bf2-9c42-302aaa2daafb?diagramType=Topology) one. Note that this is a reservable instance as the always-on is restricted in functionality.

### Flow Analysis - Start trace

DNAC allows you to perform flow-analysis through the `dna/intent/api/v1/flow-analysis` API. In below screenshot, pay attention to the JSON body. Here we will perform a trace from 192.168.30.60 to 192.168.30.61

![DNAC](/images/2020-05-03-1.png)

You'll see we get back a response containing a `flowAnalysisId`. We can use that ID in the `dna/intent/api/v1/flow-analysis/{flowAnalysisId}` API call. The response contains the details of the path trace.

![DNAC](/images/2020-05-03-2.png)

That's it in terms of getting familiar with Flow Analysis. In the next post, we will implement this example using Python. Stay tuned!