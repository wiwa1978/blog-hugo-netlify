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

##### Find organizations

[Documentation](https://developer.cisco.com/meraki/api/#/rest/api-endpoints/organizations/get-organizations)

![meraki](/images/2020-06-01-1.png)

##### Find specific organization

[Documentation](https://developer.cisco.com/meraki/api/#/rest/api-endpoints/organizations/get-organization)

![meraki](/images/2020-06-01-2.png)

##### Get Organizations inventory

[Documentation](https://developer.cisco.com/meraki/api/#/rest/api-endpoints/organizations/get-organization-inventory)

![meraki](/images/2020-06-01-3.png)

##### Get Organizations devices

![meraki](/images/2020-06-01-4.png)

##### Get Organizations devices status

[Documentation](https://developer.cisco.com/meraki/api/#/rest/api-endpoints/organizations/get-organization-device-statuses)

![meraki](/images/2020-06-01-5.png)

##### Get Networks for organization

[Documentation](https://developer.cisco.com/meraki/api/#/rest/api-endpoints/networks/get-organization-networks)

![meraki](/images/2020-06-01-6.png)

##### Get Network

[Documentation](https://developer.cisco.com/meraki/api/#/rest/api-endpoints/networks/get-network)

![meraki](/images/2020-06-01-7.png)

##### Create combined Network

[Documentation](https://developer.cisco.com/meraki/api/#/rest/api-endpoints/networks/combine-organization-networks)

> Needs reserved instance and replace screenshot

![meraki](/images/2020-06-01-8.png)


##### Create WIFI Network

> Needs reserved instance and replace screenshot

![meraki](/images/2020-06-01-9.png)

##### Claim Network Device

[Documentation](https://developer.cisco.com/meraki/api/#/rest/api-endpoints/devices/claim-network-devices)

> Needs reserved instance and replace screenshot

![meraki](/images/2020-06-01-10.png)