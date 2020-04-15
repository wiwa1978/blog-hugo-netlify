---
title: NetPalm Introduction - Part 3
date: 2020-04-15T12:32:50+01:00
draft: True
categories:
  - Network Programming
  - Programming
tags:
  - Netpalm
---
### Introduction

This post is a follow up on [part 1](http://blog.wimwauters.com/networkprogrammability/2020-04-14_netpalm_introduction_part1/)  and [part 2](https://blog.wimwauters.com/networkprogrammability/2020-04-15_netpalm_introduction_part2/). 

In part 1, we focused on retrieving information from our devices using NetPalm. 
In part 2, we will focus on changing/setting information in our device.

In this post, we will explore some more advanced features of NetPalm. We will be covering:

- using Jinja2 templates to change configuration
- using service templates

### Using Jinja2 templates
NetPalm allows you to change configuration on devices using Jinja2 templates in combination with NAPALM and Netmiko. If you are a regular reader of my blog, we have been changing configurations already using NAPALM and Netmiko individually:

- NAPALM: we added a loopback interface to our device using a Jinja2 template (check [this](https://blog.wimwauters.com/networkprogrammability/2020-04-07_napalm_introduction_part2/) post )

- Netmiko: we added a loopback interface to our device using a Jinja2 template and we also changed the description of an interface (check [this](https://blog.wimwauters.com/networkprogrammability/2020-03-25-netmiko_introduction/) post)

In this post, we will add some loopback interfaces to our device but using NetPalm (which under the hood will use NAPALM or Netmiko).

Let's first create the Jinja2 template. Create a file called `loopback.j2` and place this in the `netpalm/backend/plugins/jinja2_templates` folder. We will be using the following template:

```jinja
{% for interface in interfaces %}
   interface {{interface.name}}
   description {{interface.description}}
   ip address {{interface.ipv4_addr}} {{interface.ipv4_mask}}
{% endfor %}
```
NetPalm provides an API to change the configuration through a Jinja2 template (see [here](https://documenter.getpostman.com/view/2391814/SzYbxcQx?version=latest#85056012-e546-41d7-b832-19e9285823f7) for Netmiko and [here](https://documenter.getpostman.com/view/2391814/SzYbxcQx?version=latest#44fdde62-c21c-417b-95d4-54fa2640d135) for NAPALM.

The example is the following:

![netpalm](/images/2020-04-17-1.png)

Essentially the `template` argument points to the template we have stored in the `netpalm/backend/plugins/jinja2_templates` folder, in our case this will be `loopback` (not loopback.j2). Next, in the args, we need to pass the variables for our Jinja2 template. In our case, we want to provide the following information.

``` json
{
  "interfaces": [
    {
      "name": "Loopback3001",
      "description": "Description for Loopback 3001",
      "ipv4_addr": "200.200.200.230",
      "ipv4_mask": "255.255.255.255"
    },
    {
      "name": "Loopback3002",
      "description": "Description for Loopback 3002",
      "ipv4_addr": "200.200.200.231",
      "ipv4_mask": "255.255.255.255"
    }
  ]
}
```
Below is an example of the complete REST API call:

![netpalm](/images/2020-04-17-2.png)

And a view on the task (pay attendtion to the changes attribute):

![netpalm](/images/2020-04-17-3.png)

Next, let's check if it worked correctly. We will be using the Netmiko `getconfig` API for this (this [one](https://documenter.getpostman.com/view/2391814/SzYbxcQx?version=latest#4d45bf0a-f408-47e0-8dba-c0b26529591c)).

![netpalm](/images/2020-04-17-4.png)

In below screenshot, you can see these two loopback interfaces got added successfully.

![netpalm](/images/2020-04-17-5.png)

Or via SSH into our devices, we see them listed indeed:

```bash
csr1000v-1#show ip interface brief
Interface              IP-Address      OK? Method Status                Protocol
GigabitEthernet1       10.10.20.48     YES other  up                    up
GigabitEthernet2       unassigned      YES NVRAM  administratively down down
GigabitEthernet3       unassigned      YES NVRAM  administratively down down
Loopback76             unassigned      YES unset  up                    up
Loopback100            unassigned      YES unset  up                    up
Loopback1000           unassigned      YES unset  up                    up
Loopback2000           10.0.0.40       YES other  up                    up
Loopback3001           200.200.200.230 YES manual up                    up
Loopback3002           200.200.200.231 YES manual up                    up
```

