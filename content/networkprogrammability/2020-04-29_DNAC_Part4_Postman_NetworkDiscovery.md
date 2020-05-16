---
title: Cisco DNA Center - Discovery & Command Runner (POSTMAN)
date: 2020-04-29T06:32:50+01:00
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
- Part 5 (this post): [Cisco DNA Center - Discovery (POSTMAN)](https://blog.wimwauters.com/networkprogrammability/2020-04-29_dnac_part4_postman_networkdiscovery/)
- Part 6: [Cisco DNA Center - Discovery (Python)](https://blog.wimwauters.com/networkprogrammability/2020-05-01_dnac_part5_pythonrequests/)
- Part 7: [Cisco DNA Center - CommandRunner (Python)](https://blog.wimwauters.com/networkprogrammability/2020-05-02_dnac_part6_pythonrequests/)

In this tutorial, we will look into (and experiment) with some more advanced features of DNAC, like Network Discovery and Command Runner. To start with, we will focus on some POSTMAN interactions to get familiar with the flow of these features. As a reminder, the API documentation for DNAC can be found [here](https://developer.cisco.com/docs/dna-center/api/1-3-3-x/). Refer to the `Network Decovery` and the `Command Runner` section respectively.

### Note about equipment

In this post, I'm using my own DNAC in my lab. However, if you want to follow along, you could also use a Cisco sandbox environment delivered by [Cisco Devnet](https://developer.cisco.com). To get a list of all sandboxes, check out [this](https://devnetsandbox.cisco.com/) link. For this tutorial, you could use [this](https://devnetsandbox.cisco.com/RM/Diagram/Index/b8d7aa34-aa8f-4bf2-9c42-302aaa2daafb?diagramType=Topology) one. Note that this is a reservable instance as the always-on is restricted in functionality.


### Network Discovery

DNAC allows you to discover devices in your network by scanning the network based on a given IP range or through CDP. Let's see how the IP range based discovery works.

###### Get Credentials

As a first step we need to retrieve the credentials. The credentials endpoint looks as follows: `/dna/intent/api/v1/global-credential`. You are supposed to provide a query parameter as well. This can be:
- CLI
- SNMPV2_READ_COMMUNITY 
- SNMPV2WRITE_COMMUNITY 
- SNMPV3
- HTTP_WRITE
- HTTP_READ
- NETCONF

Below is an example for the credential type `CLI`.

![DNAC](/images/2020-04-29-1.png)

In the next step, we will use the credentials ID.

###### Start IP based discovery
In order to start the IP based discovery, we need to call the `dna/intent/api/v1/discovery` endpoint and pass a JSON body with the required IP range to search in. Refer to below screenshot for an example.

![DNAC](/images/2020-04-29-2.png)

###### Check Task

As you can see, we get back from DNAC a response containing a `taskID` and a `URL`. The network discovery is done in an asynchronous way so the idea is that we check on a regular base the `tasks` API. Below screenshot shows an example:

![DNAC](/images/2020-04-29-3.png)

###### Get Discovery status

Querying the task API, gives you a response that allows you to check the status of the discovery process. You will see we get back a `progress` number. This essentially is the ID of the discovery job itself. We will use this ID in the discovery API `/dna/intent/api/v1/discovery/{id}`. See below for the corresponding screenshot.

![DNAC](/images/2020-04-29-4.png)

###### Get Discovered devices 
In previous screenshot, you could see in the response the line item `discoveryCondition: "complete"`. This essentially means the discovery process was completed successfully. So as a next step, we can actually look at what devices were discovered. You can do this by using the following API `/dna/intent/api/v1/discovery/{id}/network-device`

![DNAC](/images/2020-04-29-5.png)

You'll see we get back a complete list of discovered devices (in the specified IP range) with the corresponding information.

### Command Runner

DNAC also allows you to execute commands on a list of devices. It's something we have already done in many `network programmability` blog posts before (e.g. using [Ansible](https://blog.wimwauters.com/networkprogrammability/2020-04-29_ansible_iosxe_iosmodules/), using [NAPALM](https://blog.wimwauters.com/networkprogrammability/2020-04-07_napalm_introduction_part2/), using [Netmiko](https://blog.wimwauters.com/networkprogrammability/2020-03-25-netmiko_introduction/), ... ).  DNAC also allows you to run commands against our devices. To do so, we will perform a number of steps. 

###### Get list of devices
Firstly, we need to specify the list of devices we want to run the commands against. We will use the API we have seen [here](http://blog.wimwauters.com/networkprogrammability/2020-04-24_dnac_part1_pythonrequests#retrieve-all-devices-using-python) already. In that example, we did not specify a query parameter. In the below example we will specify a `family` and a `type` parameter to retrieve only the device types belonging to a specific family/category.

See below screenshot for an example how this is achieved.

![DNAC](/images/2020-04-29-6.png)

###### Run command

In above screenshot, you will see we get back a list of devices belonging to a specific type and a specific family, e.g. all Catalyst 9300 switches from the 'Switches and Hubs' family. This is the list of devices we will run the commands against in the next step. See the screenshot below for an example. Here we run the `show version` command on these devices (note: we will pass the device ID's in the JSON body).

![DNAC](/images/2020-04-29-7.png)

###### Check Task

From previous screenshot, you see we get back again a task response. These commands are run asynchronously and we are supposed to regularly poll the `tasks` API to check the response. Below is an example screenshot on how we are doing this.

![DNAC](/images/2020-04-29-8.png)

###### Check result

In previous screenshot, you can see we got back a fileid as part of the `process` attribute. We can now use that fileid to retrieve the file content. We will use the 

![DNAC](/images/2020-04-29-9.png)

If all works well, you will see the output of the command you wanted to run.

That's it in terms of getting familiar with network discovery process and running some commands through Command Runner. In the next post, we will implement all of this using Python. Stay tuned!