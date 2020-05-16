---
title: Cisco Meraki - Getting Started
date: 2020-06-01T08:32:50+01:00
draft: true
categories:
  - Network Programming
  - Programming
  - All
tags:
  - Meraki
  - POSTMAN
---
### Introduction


### Credits

Most of the examples below are making use of Nick Russo's SD-WAN POSTMAN collection. Find them at Nick's website ([here](http://njrusmc.net/jobaid/jobaid.html)). Also, Nick has a brilliant course on SD-WAN at Pluralsight. Really worth watching it if you want to explore how to run APIs against Cisco's SD-WAN solution. Check out Nick's course `Automating Cisco SD-WAN Operations Using APIs` [here](https://app.pluralsight.com/library/courses/automating-cisco-sd-wan-operations-using-apis/table-of-contents)).


### Authorization

You can authorize by passing the `X-Cisco-Meraki-API-Key` in the header of your requests.

### Dashboard API

For the official documentation, please check [this](https://developer.cisco.com/meraki/api/#/rest/guides/rest-api-quick-start) link.

##### Find organization

![meraki](/images/2020-06-01-1.png)

Find Network ID

`/organizations/:organizationId/networks`

![meraki](/images/2020-06-01-2.png)

