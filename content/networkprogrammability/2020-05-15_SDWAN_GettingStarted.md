---
title: Cisco SD-WAN - Getting Started
date: 2020-05-15T08:32:50+01:00
draft: false
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
In the above script, we retrieved the device controllers. With a small variation to the above script we can also retrieve a list of vEdge devices. Just call the `/dataservice/system/device/vedges` API. 

![sdwan](/images/2020-06-01-3.png)

Obviously, the POSTMAN response is also reflected in the 

![sdwan](/images/2020-06-01-3-a.png)

Let's say we would only be interested to have an overview of the CSR1000v edge devices. Then the API allows us also to 'filter' the response by using a query parameters. See below for the relevant POSTMAN example.

![sdwan](/images/2020-06-01-4.png)

As you would expect, the returned list of CSR1000v correspond to the list in the SD-WAN user interface.

![sdwan](/images/2020-06-01-4-a.png)
 
### Get templates

In order to retrieve templates from vManage, we will use the `/dataservice/template/` API.

![sdwan](/images/2020-06-01-5.png)

Checking the templates in the user interface of course gets us back the same list of templates.

![sdwan](/images/2020-06-01-5-a.png)

### Get feature template

SD-WAN works with so called feature templates. Retrieving these feature templates is very similar to retrieving templates. We will use the `/dataservice/template/feature` API.

![sdwan](/images/2020-06-01-6.png)

Below the UI with the same feature templates.

![sdwan](/images/2020-06-01-6-a.png)

### Get alarm count

We can also retrieve the alarm counts. Use the `/dataservice/alarms/count` API` for that.

![sdwan](/images/2020-06-01-7.png)

You'll see we receive back a response with the alarm count and the amount of cleared alarms.

![sdwan](/images/2020-06-01-7-a.png)


### Add user

Adding users is done through sending a POST request to the `/dataservice/admin/user` API. Note the capital N in userName in the JSON body (if spelled wrong, the user will not be added).

![sdwan](/images/2020-06-01-8.png)

If all works well, the user is added here.

![sdwan](/images/2020-06-01-8-a.png)

### Change Password

Also changing the password through the API is possible. For this, use the `/dataservice/admin/user/password/<user>` API.

![sdwan](/images/2020-06-01-9.png)

### Get Certificates

In order to retrieve the certificates, you can use the `/dataservices/certificate/vsmart/list` API. 

![sdwan](/images/2020-06-01-10.png)

Below is a screenshot showing the list of certificates.

![sdwan](/images/2020-06-01-10-a.png)

The intention of this post is to provide a short overview of some relevant SD-WAN APIs, nothing really more. In a next post, I will provide some Python scripts that implement these API calls.

