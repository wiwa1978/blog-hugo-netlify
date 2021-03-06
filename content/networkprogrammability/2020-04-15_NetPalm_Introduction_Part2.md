---
title: NetPalm Introduction - Part 2
date: 2020-04-15T09:32:50+01:00
draft: false
categories:
  - Network Programming
  - Programming
  - All
tags:
  - NetPalm
---
### Introduction

This post is a follow up on [part 1](http://blog.wimwauters.com/networkprogrammability/2020-04-14_netpalm_introduction_part1/) where we focused on retrieving information from our devices using NetPalm. In part 2, we will focus on changing/updating information on our device. Again, we will use NetPalm's API to do so, yet NetPalm will use different underlying tools each time.

### Use Case 1: change configuration via NetPalm - Netmiko
In this use case, we want to change some configuration on our device. We will do the following:

1. verify the existing interfaces
2. update an interface
3. query the interface to see if the update did work

###### 1) Verify the existing interfaces

First, let's get an overview of the existing interfaces. We will be using the method we learned in use case 1 from [part 1](https://blog.wimwauters.com/networkprogrammability/2020-04-14_netpalm_introduction_part1/) above.

![netpalm](/images/2020-04-15-1.png)

Let's execute the task and you will see we get an overview of all interface descriptions, just as we requested.

![netpalm](/images/2020-04-15-2.png)

###### 2) Update an interface 
Next, we will change the description for interface GigabitEthernet3. We can do this via the `setconfig` method as documented [here](https://documenter.getpostman.com/view/2391814/SzYbxcQx?version=latest#c6c4ca08-6ba5-4272-b9cb-457e1a986d57).

You will see that in the `config` attribute, we pass the commands that we want to execute on our device. Please note that Netmiko does not require a `conf t`.

![netpalm](/images/2020-04-15-3.png)

In the response, we get back a view on the changes we performed. Also note that the `"status": "success"` in our response.

![netpalm](/images/2020-04-15-4.png)

######  3.Query the interface to see if the update did work

Let's have a look again at the interface description via the API we used in the first step.

![netpalm](/images/2020-04-15-5.png)

You will notice indeed that the interface description got changed to 'Set via Netpalm', just as we requested. 

![netpalm](/images/2020-04-15-6.png)

That's how easy it is in fact to change some configuration on our device using Netpalm. Note, we could have just as easily used NAPALM as well to accomplish the same.

### Use Case 2: Add interface via NetPalm - RESTCONF

RESTCONF in general is a bit of a tougher protocol so I wanted to take the opportunity to provide some examples as well. Let's give it a try to add an interface to our device.

Again we will do the following

1. verify the existing interfaces
2. add an interface
3. query the interface to see if the create worked

###### 1) Verify the existing interfaces

Let's first check what interfaces we currently have on our device. For this, we will be using RESTCONF to retrieve the information. In fact, it's exactly what we learned in use case 4 from [part 1](https://blog.wimwauters.com/networkprogrammability/2020-04-14_netpalm_introduction_part1/). We could just as easily have used Netmiko or NAPALM for this but we saw this already in use case 1 above.

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
###### 2) Add an interface 

Let's now add an additional interface (we will be adding a BDI100). NetPalm has an API for doing this, check it out [here](https://documenter.getpostman.com/view/2391814/SzYbxcQx?version=latest#68da9960-7c95-4045-8a28-15becdb2b104).

In below example, you see that we call the `setconfig` API and pass in the JSON body quite some information:

- URI: corresponds to the URI to deal with the interfaces on our IOS XE device 
- payload: the payload required to add an interface to the device 

Note: we have done this already before in use case 6 of [this](https://blog.wimwauters.com/networkprogrammability/2020-04-03_restconf_introduction_part2/) post. Use the information from that post to understand the URI and payload in this example.

Below is the screenshot where we add (POST) an interface to our device.

![netpalm](/images/2020-04-15-9.png)

In the below screenshot, you see we basically get back an HTTP status code 201 from the REST API, which is a response code indicating something got created successfully on our device. 

![netpalm](/images/2020-04-15-10.png)

###### 3. Query the interface to see if the create worked
Let's see if the interface was added successfully. To safe some cycles here, we will just login to the device via SSH. You can see in below snippet that the interface `BDI100` got successfully added to the device.

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

See below screenshot to achieve this. Pay attention the action `PATCH` which indicates we want to update the configuration.

![netpalm](/images/2020-04-15-11.png)

Again we get back a `"status": "success"` message indicating everything worked successfully.

![netpalm](/images/2020-04-15-12.png)

Again, let's have a look at our device through SSH.

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

In the response, you will get back a status mentioning the action has been successfull. See below screenshot.

![netpalm](/images/2020-04-15-14.png)

Next, we will issue once again the `getconfig` Netmiko method to verify our list of interfaces. Again, as mentioned already couple of times, we could have just used SSH to verify this, or NAPALM, or Netmiko or RESTCONF directly...

![netpalm](/images/2020-04-15-15.png)

And as we expected, the interface `BDI100` is deleted from our device and hence is not showing up in the interface list.

![netpalm](/images/2020-04-15-16.png)

Hope you enjoyed this part 2 of the introduction to NetPalm and you start to see the benefits of the tool 

In a [next](https://blog.wimwauters.com/networkprogrammability/2020-04-17_netpalm_introduction_part3/) blog post, we'll dive deeper into some more advanced topics. Hope to see you back!


