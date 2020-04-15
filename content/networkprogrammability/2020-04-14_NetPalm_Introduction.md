---
title: NetPalm Introduction
date: 2020-04-14T09:32:50+01:00
draft: false
categories:
  - Network Programming
  - Programming
tags:
  - Netpalm
---
### Introduction
Recently I came across [this](https://www.reddit.com/r/devops/comments/fes4w1/netpalm_open_source_rest_api_broker_for_your/) article. As I'm dealing with quite some network automation tools, it triggered my interest. I decided to give it a go. Below you can find a 'Getting started' tutorial.

### What is NetPalm
NetPalm essentially is a REST API broker for tools like NAPALM, Netmiko, RESTCONF or Python. It exposes a set of unified APIs (getconfig, setconfig...) to allow us to interact with a variety of devices using one of the aforementioned protocols. So no need to be intimate with the underlying tools (e.g NAPALM, Netmiko,...) as it's all abstracted away behind a simple REST API.

Internally NetPalm uses a REDIS queue to process the different tasks towards our devices. In other words, if we query NetPalm's REST API, our query will end up into a REDIS queue and NetPalm will process this queue one by one. We can query the tasks in the queue to retrieve the results of our tasks. It'll all become clear in the below use cases.

### Installing NetPalm

The installion instructions on NetPalm's [Github](https://github.com/tbotnz/netpalm) repo are pretty straightforward. Just do the following:
```
(venv) WAUTERW-M-65P7:Netpalm_Introduction wauterw$ git clone https://github.com/tbotnz/netpalm.git
(venv) WAUTERW-M-65P7:Netpalm_Introduction wauterw$ cd netpalm/
```
NetPalm runs inside docker containers, so we'll need to build these:
```
(venv) WAUTERW-M-65P7:netpalm wauterw$ sudo docker build -t netpalm .
Successfully built a56e1fb19bfb
Successfully tagged netpalm:latest
```
Let's check on the docker images to got pulled:
```
(base) WAUTERW-M-65P7:netpalm wauterw$ docker images
REPOSITORY                                  TAG                 IMAGE ID            CREATED             SIZE
netpalm_netpalm                             latest              13fcfbb459fc        5 hours ago         1.04GB
netpalm                                     latest              42b2d099b087        5 hours ago         1.04GB
python                                      3.8                 d47898c6f4b0        2 weeks ago         933MB
redis                                       latest              4cdbec704e47        2 weeks ago         98.2MB
```
In the above output, you can clearly see NetPalm uses Redis for queue management.
Next, we need to bring up the containers in such a way that the NetPalm containers can talk to the Redis container.
```
(venv) WAUTERW-M-65P7:netpalm wauterw$ docker-compose up -d
Creating network "netpalm_default" with the default driver
Creating netpalm_redis_1 ... done
Creating netpalm_netpalm_1 ... done
```
Let's check what happened here:
```
(venv) WAUTERW-M-65P7:netpalm wauterw$ docker ps
CONTAINER ID        IMAGE               COMMAND                  CREATED             STATUS              PORTS                    NAMES
603fe3088632        netpalm_netpalm     "/bin/sh -c 'python3…"   33 minutes ago      Up 33 minutes       0.0.0.0:9000->9000/tcp   netpalm_netpalm_1
654a0b2623ec        redis               "docker-entrypoint.s…"   33 minutes ago      Up 33 minutes       6379/tcp                 netpalm_redis_1
```
As mentioned in the documentation, we can reach our NetPalm container via the URL `http://localhost:9000`. Note: NetPalm is a REST API broker, so don't use a browser to go to that URL, but instead use a tool like POSTMAN. 

Pro tip: the Github repo also published the Postman collection (see [here](https://github.com/tbotnz/netpalm/blob/master/netpalm.postman_collection.json)).

Let's now look into some simple use cases. Have fun!

### Use Case 1: get configuration via Netmiko

Let's start with something fairly basic. We'll get an overvew of all the interfaces through Netpalm. According to the [API documentaton](https://documenter.getpostman.com/view/2391814/SzYbxcQx?version=latest#d0352af0-41d0-414e-ac04-3a4977b045f5) we need to call the `getconfig` method.  
![netpalm](/images/2020-04-14-1.png)

You'll notice we get back a `task_id` in the response. We can use this id to query the `task` API (see [here](https://documenter.getpostman.com/view/2391814/SzYbxcQx?version=latest#3615b003-1d94-4514-a90f-4da27e107085).) As a result, you will see that we get back an overview of the interfaces on our device.

![netpalm](/images/2020-04-14-2.png)

### Use Case 2: get configuration via NAPALM
As mentioned above, one of the nice things about Netpalm is that it provides the same API regardless of the underlying tool it uses to retrieve (or set) the configuration. In what follows, we will do exactly the same but instead of Netmiko, we'll use Napalm.

You will see the only change we have made is in the JSON body of the `getconfig` request. It's documented [here](https://documenter.getpostman.com/view/2391814/SzYbxcQx?version=latest#c44945e2-92a1-44cc-aee6-d151086ee9d6). One small note here is to pay attention when you are specifying an SSH port. The syntax in NAPALM to set a port is a little different from how it's done in Netmiko. 

![netpalm](/images/2020-04-14-3.png)

Next, we'll query the `task` API to retrieve the list of interfaces.

![netpalm](/images/2020-04-14-4.png)

### Use Case 3: get configuration via ncclient
Let's do something similar but with ncclient. We discussed this in [this](https://blog.wimwauters.com/networkprogrammability/2020-03-30-netconf_python_part1/) and [this](https://blog.wimwauters.com/networkprogrammability/2020-03-31_netconf_python_part2/) blog posts.

Let's call the `getconfig` API (documented [here](https://documenter.getpostman.com/view/2391814/SzYbxcQx?version=latest#b1c5c808-bbb3-4909-acf9-f11d0d402d77)).
![netpalm](/images/2020-04-14-5.png)

Pay particular attention to the JSON body of the request. You'll see we are using the ncclient library in this example. You also notice we can specify a filter. For this example, I have been using the following filter as I'm only interested to receive information about interface 'GigabitEthernet1'.

```
<filter>
  <interfaces xmlns='urn:ietf:params:xml:ns:yang:ietf-interfaces'>
    <interface>
       <name>GigabitEthernet1</name>
    </interface>
  </interfaces>
</filter>
```
Query again the `task` API and you will get back the interface configuration through ncclient.
![netpalm](/images/2020-04-14-6.png)

### Use Case 4: get configuration via RESTCONF
I imagine you start to get the big picture here, but I wanted to conclude this with Netpalm's RESTCONF API. Just for completeness more than anything else. If you want to learn more about RESTCONF, check out my [blog post](https://blog.wimwauters.com/networkprogrammability/2020-04-02_restconf_introduction/) on it.

Let's call the `getconfig` API (documented [here]((https://documenter.getpostman.com/view/2391814/SzYbxcQx?version=latest#3280e040-b27f-4609-990f-947754e6afef)). Note that we are using the RESTCONF URL `/restconf/data/ietf-interfaces:interfaces` to retrieve the interfaces on this device.

![netpalm](/images/2020-04-14-8.png)

Query again the `task` API and you will get back the interface configuration through RESTCONF.

![netpalm](/images/2020-04-14-9.png)

### Use Case 5: change configuration via Netmiko
So far we have only been reading configuration data from our device. What about changing some configuration data? Works easy with Netpalm.

First, let's get an overview of the existing interfaces. We will be using the method we learned in use case 1 above.

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


### Use Case 6: add interface via RESTCONF

RESTCONF in general is a bit of a tougher protocol. Let's give it a try to add an interface to our device. In the below example, we will be adding a Loopback interface.

Let's first check what interfaces we currently have on our device. For this, we will be using what we learned in use case 4.
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

### Use Case 7: remove interface via RESTCONF

Let's now also remove the interface we just added. We will be using [this](https://documenter.getpostman.com/view/2391814/SzYbxcQx?version=latest#f7c75846-aace-4bc4-ab58-2ec34054b4e6) API to achieve that.

Instead of using SSH to verify the current set of interfaces, let's use the NAPALM getconfig method (see use case 2). You will get back a list of the current interfaces. Verify that Loopback 2000 is mentioned.

![netpalm](/images/2020-04-14-20.png)

Next, let's continue with deleting the interface. In below screenshot, you will notice that I point the URI to the exact interface we want to see deleted and we add the 'delete' method.

![netpalm](/images/2020-04-14-21.png)

In the response, you will get back 204 HTTP message and a task_status mentioning the action has been finished.

![netpalm](/images/2020-04-14-22.png)

Next, we will issue again the `getconfig` NAPALM method to verify our list of interfaces.

![netpalm](/images/2020-04-14-23.png)

And as we expect, the interface `Loopback2000` is deleted from our device and hence is not showing up in the interface list.

![netpalm](/images/2020-04-14-24.png)


### Use Case 8: changing interface via RESTCONF

Here I'm specifying the exact interface `"uri":"/restconf/data/ietf-interfaces:interfaces/interface=Loopback2000"`
![netpalm](/images/2020-04-14-25.png)

But the description is not changed

![netpalm](/images/2020-04-14-26.png)

Here I'm not specifying the exact interface `"uri":"/restconf/data/ietf-interfaces:interfaces"`
![netpalm](/images/2020-04-14-27.png)

But the description is not changed

![netpalm](/images/2020-04-14-28.png)



