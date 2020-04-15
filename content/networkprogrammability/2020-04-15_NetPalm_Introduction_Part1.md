---
title: NetPalm Introduction - Part 1
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
