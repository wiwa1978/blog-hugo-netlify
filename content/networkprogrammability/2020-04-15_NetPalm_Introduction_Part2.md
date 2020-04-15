---
title: NetPalm Introduction - Part 2
date: 2020-04-15T09:32:50+01:00
draft: false
categories:
  - Network Programming
  - Programming
tags:
  - Netpalm
---
### Introduction

This post is a follow up on [part 1](https://blog.wimwauters.com/networkprogrammability/2020-04-14_netpalm_introduction/) where we focused on retrieving information from our devices using NetPalm. In part 2, we will focus on changing/setting information in our device.

### Use Case 1: change configuration via Netmiko
So far we have only been reading configuration data from our device. What about changing some configuration data? Works easy with Netpalm.

First, let's get an overview of the existing interfaces. We will be using the method we learned in use case 1 from [part 1](https://blog.wimwauters.com/networkprogrammability/2020-04-15_netpalm_introduction_part1/) above.

![netpalm](/images/2020-04-14-10.png)

Let's execute the task and you will see we get an overview of all interface descriptions, just as we requested.

![netpalm](/images/2020-04-14-11.png)

Next, we will change the description for interface GigabitEthernet3. We can do this via the `setconfig` method as documented [here](https://documenter.getpostman.com/view/2391814/SzYbxcQx?version=latest#c6c4ca08-6ba5-4272-b9cb-457e1a986d57).

![netpalm](/images/2020-04-14-12.png)

In the response, we get back a view on the changes we performed.

![netpalm](/images/2020-04-14-13.png)

Let's have a look again at the interface description via the API we used in the first step.

![netpalm](/images/2020-04-14-14.png)

You will notice indeed that the interface description got changed to 'Set via Netpalm', just as we requested. That's how easy it is in fact to change some configuration on our device using Netpalm.

![netpalm](/images/2020-04-14-15.png)


### Use Case 2: add interface via RESTCONF

RESTCONF in general is a bit of a tougher protocol. Let's give it a try to add an interface to our device. In the below example, we will be adding a Loopback interface.

Let's first check what interfaces we currently have on our device. For this, we will be using what we learned in use case 4 from [part 1](https://blog.wimwauters.com/networkprogrammability/2020-04-15_netpalm_introduction_part1/).

First we will call the getconfig (via RESTCONF):

![netpalm](/images/2020-04-14-16.png)

You'll see an overview of all interfaces in below screenshot.

![netpalm](/images/2020-04-14-17.png)

It's a bit hard to read it from the response message, so I've logged in to the device via SSH to retrieve the list.

```bash
csr1000v-1#show ip interface brief
Interface              IP-Address      OK? Method Status                Protocol
GigabitEthernet1       10.10.20.48     YES other  up                    up
GigabitEthernet2       unassigned      YES NVRAM  administratively down down
GigabitEthernet3       unassigned      YES NVRAM  administratively down down
Loopback76             unassigned      YES unset  up                    up
Loopback100            unassigned      YES unset  up                    up
Loopback1000           unassigned      YES unset  up                    up
```
Let's now add an additional loopback interface. NetPalm has an API for doing this, check it out [here](https://documenter.getpostman.com/view/2391814/SzYbxcQx?version=latest#68da9960-7c95-4045-8a28-15becdb2b104).

In below example, you see that we call the `setconfig` API and pass in the JSON body quite some information:
- URI: corresponds to the URI to deal with the interfaces on our IOS XE device (see [this](https://blog.wimwauters.com/networkprogrammability/2020-03-30-netconf_python_part1/) and [this](https://blog.wimwauters.com/networkprogrammability/2020-03-31_netconf_python_part2/) post to learn more on why we use that URI exactly).
- payload: the payload required to add an interface to the device

![netpalm](/images/2020-04-14-18.png)

In the below screenshot, you see we basically get back a status code 204, which means 'No Content'.

![netpalm](/images/2020-04-14-19.png)

Next, let's login to the device. You can see in below snippet that the interface `Loopback2000` got added to the device successfully.

```bash
csr1000v-1#show ip interface brief
Interface              IP-Address      OK? Method Status                Protocol
GigabitEthernet1       10.10.20.48     YES other  up                    up
GigabitEthernet2       unassigned      YES NVRAM  administratively down down
GigabitEthernet3       unassigned      YES NVRAM  administratively down down
Loopback76             unassigned      YES unset  up                    up
Loopback100            unassigned      YES unset  up                    up
Loopback1000           unassigned      YES unset  up                    up
Loopback2000           10.0.0.40       YES other  up                    up1
csr1000v-1#show interface description
Interface                      Status         Protocol Description
Gi1                            up             up       MANAGEMENT INTERFACE - DON'T TOUCH ME
Gi2                            admin down     down     Network Interface
Gi3                            admin down     down     Set via Netpalm
Lo76                           up             up       scripted with Go
Lo100                          up             up       "test it"
Lo1000                         up             up
Lo2000                         up             up       Added via Netpalm
```

### Use Case 3: remove interface via RESTCONF

Let's now also remove the interface we just added. We will be using [this](https://documenter.getpostman.com/view/2391814/SzYbxcQx?version=latest#f7c75846-aace-4bc4-ab58-2ec34054b4e6) API to achieve that.

Instead of using SSH to verify the current set of interfaces, let's use the NAPALM getconfig method (see use case 2 in [part 1](https://blog.wimwauters.com/networkprogrammability/2020-04-15_netpalm_introduction_part1/)). You will get back a list of the current interfaces. Verify that Loopback 2000 is mentioned.

![netpalm](/images/2020-04-14-20.png)

Next, let's continue with deleting the interface. In below screenshot, you will notice that I point the URI to the exact interface we want to see deleted and we add the 'delete' method.

![netpalm](/images/2020-04-14-21.png)

In the response, you will get back 204 HTTP message and a task_status mentioning the action has been finished.

![netpalm](/images/2020-04-14-22.png)

Next, we will issue again the `getconfig` NAPALM method to verify our list of interfaces.

![netpalm](/images/2020-04-14-23.png)

And as we expect, the interface `Loopback2000` is deleted from our device and hence is not showing up in the interface list.

![netpalm](/images/2020-04-14-24.png)


### Use Case 4: changing interface via RESTCONF

Here I'm specifying the exact interface `"uri":"/restconf/data/ietf-interfaces:interfaces/interface=Loopback2000"`
![netpalm](/images/2020-04-14-25.png)

But the description is not changed

![netpalm](/images/2020-04-14-26.png)

Here I'm not specifying the exact interface `"uri":"/restconf/data/ietf-interfaces:interfaces"`
![netpalm](/images/2020-04-14-27.png)

But the description is not changed

![netpalm](/images/2020-04-14-28.png)



