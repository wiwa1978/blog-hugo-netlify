---
title: Cisco SD-WAN - Getting Started
date: 2020-05-01T08:32:50+01:00
draft: true
categories:
  - Network Programming
  - Programming
  - All
tags:
  - SDWAN
  - Python
---
### Introduction

SD-WAN is a software-defined approach to managing the WAN. In the past, MPLS networks were used to ensure reliable connectivity between users connecting at the branch to applications running on servers in the datacenter. SD-WAN is a new approach to building out WAN networks. It will help organizations to manage their WAN network from a single pane of glass, ensuring optimized routing paths, ensuring high availability for enterprise applications and easily connect your WAN Network to the cloud. 

### Authenticate

First off, we need to authenticate with the SDWAN solution. Therefore we need to send a POST request to `https://{{vmanage}}:{{port}}/j_security_check`. In the body, we need to specify the username (called `j_username`) and password (`j_password`). Have a look [here](https://sdwan-docs.cisco.com/Product_Documentation/Command_Reference/Command_Reference/vManage_REST_APIs/vManage_REST_APIs_Overview/Using_the_vManage_REST_APIs) to get acquainted with the API

![sdwan](/images/2020-06-01-1.png)

### Get device controllers

With authentication out of the way, we can now focus on retrieving some data. In the next script, we will retrieve a list of device controllers from our SD-WAN setup. Code is pretty easy.

![sdwan](/images/2020-06-01-2.png)

![sdwan](/images/2020-06-01-2-a.png)

### Get edge devices

![sdwan](/images/2020-06-01-3.png)

![sdwan](/images/2020-06-01-3-a.png)

![sdwan](/images/2020-06-01-4.png)

![sdwan](/images/2020-06-01-4-a.png)
 
### Get templates

![sdwan](/images/2020-06-01-5.png)

![sdwan](/images/2020-06-01-5-a.png)

### Get feature template

![sdwan](/images/2020-06-01-6.png)


![sdwan](/images/2020-06-01-6-a.png)

### Get alarm count

![sdwan](/images/2020-06-01-7.png)

![sdwan](/images/2020-06-01-7-a.png)


### Add user

![sdwan](/images/2020-06-01-8.png)

![sdwan](/images/2020-06-01-8-a.png)

### Change Password

![sdwan](/images/2020-06-01-9.png)


### Get Certificates

![sdwan](/images/2020-06-01-10.png)

![sdwan](/images/2020-06-01-10-a.png)

