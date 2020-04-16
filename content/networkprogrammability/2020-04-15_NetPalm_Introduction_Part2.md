---
title: NetPalm Introduction - Part 2
date: 2020-04-15T09:32:50+01:00
draft: false
categories:
  - Network Programming
  - Programming
tags:
  - NetPalm
---
### Introduction

This post is a follow up on [part 1](http://blog.wimwauters.com/networkprogrammability/2020-04-14_netpalm_introduction_part1/) where we focused on retrieving information from our devices using NetPalm. In part 2, we will focus on changing/setting information in our device.

### Use Case 1: change configuration via NetPalm - Netmiko
So far we have only been retrieving configuration data from our device. What about changing some configuration data? Works easy with NetPalm.

First, let's get an overview of the existing interfaces. We will be using the method we learned in use case 1 from [part 1](https://blog.wimwauters.com/networkprogrammability/2020-04-14_netpalm_introduction_part1/) above.

![netpalm](/images/2020-04-15-1.png)

Let's execute the task and you will see we get an overview of all interface descriptions, just as we requested.

![netpalm](/images/2020-04-15-2.png)

Next, we will change the description for interface GigabitEthernet3. We can do this via the `setconfig` method as documented [here](https://documenter.getpostman.com/view/2391814/SzYbxcQx?version=latest#c6c4ca08-6ba5-4272-b9cb-457e1a986d57).

![netpalm](/images/2020-04-15-3.png)

In the response, we get back a view on the changes we performed.

![netpalm](/images/2020-04-15-4.png)

Let's have a look again at the interface description via the API we used in the first step.

![netpalm](/images/2020-04-15-5.png)

You will notice indeed that the interface description got changed to 'Set via Netpalm', just as we requested. That's how easy it is in fact to change some configuration on our device using Netpalm.

![netpalm](/images/2020-04-15-6.png)

### Use Case 2: add interface via NetPalm - RESTCONF

RESTCONF in general is a bit of a tougher protocol. Let's give it a try to add an interface to our device. In the below example, we will be adding a Loopback interface.

Let's first check what interfaces we currently have on our device. For this, we will be using what we learned in use case 4 from [part 1](https://blog.wimwauters.com/networkprogrammability/2020-04-14_netpalm_introduction_part1/). We will use the `getconfig netmiko` method to retrieve the current list of configured interfaces.

![netpalm](/images/2020-04-15-7.png)

Using the `task` method, we can see the output of the `show interface description` and the `show ip interface brief` commands.

![netpalm](/images/2020-04-15-8.png)


Alternatively, we can also verify the current list of interfaces using the good old CLI. I've logged in to the device via SSH to retrieve the list.

```bash
csr1000v-1#show ip interface brief
Interface              IP-Address      OK? Method Status                Protocol
GigabitEthernet1       10.10.20.48     YES other  up                    up
GigabitEthernet2       10.255.255.2    YES other  administratively down down
GigabitEthernet3       10.255.255.2    YES other  up                    up
```
Let's now add an additional loopback interface (we will be adding a Loopback100). NetPalm has an API for doing this, check it out [here](https://documenter.getpostman.com/view/2391814/SzYbxcQx?version=latest#68da9960-7c95-4045-8a28-15becdb2b104).

In below example, you see that we call the `setconfig` API and pass in the JSON body quite some information:

- URI: corresponds to the URI to deal with the interfaces on our IOS XE device 
- payload: the payload required to add an interface to the device (refer to use case 6 of [this](https://blog.wimwauters.com/networkprogrammability/2020-04-03_restconf_introduction_part2/) post to learn more on why we use that URI exactly).

Below is the screenshot where we add (POST) an interface to our device.

![netpalm](/images/2020-04-15-9.png)

In the below screenshot, you see we basically get back an HTTP status code 201 from the REST API, which is a response code indicated something got created on our device. 

![netpalm](/images/2020-04-15-10.png)

Let's see if the interface was added successfully.

![netpalm](/images/2020-04-15-11.png)

In below screenshot you can indeed see the interface 'BD100' was added successfully.

![netpalm](/images/2020-04-15-12.png)

Again as an alternative, let's login to the device. You can see in below snippet that the interface `BDI100` got successfully added to the device.

```bash
csr1000v-1#show ip interface brief
Interface              IP-Address      OK? Method Status                Protocol
GigabitEthernet1       10.10.20.48     YES other  up                    up
GigabitEthernet2       10.255.255.2    YES other  administratively down down
GigabitEthernet3       10.255.255.2    YES other  up                    up
BDI100                 unassigned      YES unset  down                  down
csr1000v-1#show interface description
Interface                      Status         Protocol Description
Gi1                            up             up       MANAGEMENT INTERFACE - DON'T TOUCH ME
Gi2                            admin down     down     Configured by RESTCONF
Gi3                            up             up       Configured by RESTCONF
BD100                          down           down     Interface added through NetPalm
```

### Use Case 3: changing interface via NetPalm - RESTCONF

Changing the interface works in a very similar way. We basically want to change the description of the interface we just created (BDI100). Let's change the description from 'Interface added through NetPalm' to something like 'Interface changed through NetPalm'.

See below screenshot to achieve this:

![netpalm](/images/2020-04-15-11.png)

Again we get back an HTTP status code of 201.

![netpalm](/images/2020-04-15-12.png)

Let's have a look at our device through SSH.

```bash
csr1000v-1#show interface description
Interface                      Status         Protocol Description
Gi1                            up             up       MANAGEMENT INTERFACE - DON'T TOUCH ME
Gi2                            admin down     down     Configured by RESTCONF
Gi3                            up             up       Configured by RESTCONF
BD100                          down           down     Interface changed through NetPalm
```
Sure enough our interface description was changed successfully.

### Use Case 4: remove interface via NetPalm - RESTCONF

Let's now also remove the interface we just added. We will be using [this](https://documenter.getpostman.com/view/2391814/SzYbxcQx?version=latest#f7c75846-aace-4bc4-ab58-2ec34054b4e6) API to achieve that.

Next, let's continue with deleting the interface. In below screenshot, you will notice that I point the URI to the exact interface we want to see deleted and we add the 'delete' method.

![netpalm](/images/2020-04-15-13.png)

In the response, you will get back a task_status mentioning the action has been finished. See below screenshot.

![netpalm](/images/2020-04-15-14.png)

Next, we will issue once again the `getconfig` Netmiko method to verify our list of interfaces.

![netpalm](/images/2020-04-15-15.png)

And as we expected, the interface `BNI100` is deleted from our device and hence is not showing up in the interface list.

![netpalm](/images/2020-04-15-16.png)



