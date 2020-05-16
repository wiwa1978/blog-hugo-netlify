---
title: Cisco DNA Center - Webhooks (POSTMAN)
date: 2020-05-05T04:32:50+01:00
draft: true
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
- Part 4: [Cisco DNA Center - Sites](https://blog.wimwauters.com/networkprogrammability/2020-04-27_dnac_part3_pythonrequests/)
- Part 5 (this post): [Cisco DNA Center - Discovery (POSTMAN)](https://blog.wimwauters.com/networkprogrammability/2020-04-29_dnac_part5_postman_networkdiscovery/)
- Part 6: [Cisco DNA Center - Discovery (Python)](https://blog.wimwauters.com/networkprogrammability/2020-05-01_dnac_part6_pythonrequests/)
- Part 7: [Cisco DNA Center - CommandRunner (Python)](https://blog.wimwauters.com/networkprogrammability/2020-05-02_dnac_part7_pythonrequests/)
- Part 8: [Cisco DNA Center - Flow-Analysis (POSTMAN)](https://blog.wimwauters.com/networkprogrammability/2020-05-03_dnac_part8_postman_flowanalysis/)
- Part 9: [Cisco DNA Center - Flow-Analysis (Python)](https://blog.wimwauters.com/networkprogrammability/2020-05-04_dnac_part9_pythonrequests_flowanalysis/)

In this tutorial, we will look into (and experiment) with some more advanced features of DNAC, like Network Discovery and Command Runner. To start with, we will focus on some POSTMAN interactions to get familiar with the flow of these features. As a reminder, the API documentation for DNAC can be found [here](https://developer.cisco.com/docs/dna-center/api/1-3-3-x/). Refer to the `Network Decovery` and the `Command Runner` section respectively.

### Note about equipment

In this post, I'm using my own DNAC in my lab. However, if you want to follow along, you could also use a Cisco sandbox environment delivered by [Cisco Devnet](https://developer.cisco.com). To get a list of all sandboxes, check out [this](https://devnetsandbox.cisco.com/) link. For this tutorial, you could use [this](https://devnetsandbox.cisco.com/RM/Diagram/Index/b8d7aa34-aa8f-4bf2-9c42-302aaa2daafb?diagramType=Topology) one. Note that this is a reservable instance as the always-on is restricted in functionality.

### Create webhooks

###### Get Assurance related events

![webhook](/images/2020-05-05-1.png)

###### Create webhook (event) subscription

![webhook](/images/2020-05-05-2.png)

###### Check webhook (event) status

![webhook](/images/2020-05-05-3.png)

###### Get webhook (event) subscriptions

![webhook](/images/2020-05-05-4.png)


