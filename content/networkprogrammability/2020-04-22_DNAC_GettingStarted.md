---
title: Cisco DNA Center - Getting Started
date: 2020-04-22T08:32:50+01:00
draft: false
categories:
  - Network Programming
  - Programming
  - All
tags:
  - DNAC
  - Python
---
### DNAC Series

This is part of a DNAC series:

- Part 1 (this post): [Getting started](https://blog.wimwauters.com/networkprogrammability/2020-04-22_dnac_gettingstarted/)
- Part 2: [Cisco DNA Center - Devices](https://blog.wimwauters.com/networkprogrammability/2020-04-24_dnac_part1_pythonrequests/)
- Part 3: [Cisco DNA Center - Assurance](https://blog.wimwauters.com/networkprogrammability/2020-04-25_dnac_part2_pythonrequests/)
- Part 4: [Cisco DNA Center - Sites](https://blog.wimwauters.com/networkprogrammability/2020-04-27_dnac_part3_pythonrequests/)
- Part 5: [Cisco DNA Center - Discovery (POSTMAN)](https://blog.wimwauters.com/networkprogrammability/2020-04-29_dnac_part4_postman_networkdiscovery/)
- Part 6: [Cisco DNA Center - Discovery (Python)](https://blog.wimwauters.com/networkprogrammability/2020-05-01_dnac_part5_pythonrequests/)
- Part 7: [Cisco DNA Center - CommandRunner (Python)](https://blog.wimwauters.com/networkprogrammability/2020-05-02_dnac_part6_pythonrequests/)

### What is Cisco DNA

Cisco DNA is Cisco’s architecture for enterprise networks. It provides an open, extensible, and software-driven approach that makes the network simpler to manage and more agile and responsive to business needs. Cisco DNA is an intelligent system that encompasses policy, automation, analytics, and open platform capabilities to deliver on all required aspects of an intent-based network.

That's a mouth full of marketing terms but in my own words it boils down to: Cisco DNA allows you to provision and configure all your network devices in minutes while it uses advanced analytics to proactively monitor and optimize your network. Read more info on DNA [here](https://www.cisco.com/c/en/us/solutions/collateral/enterprise-networks/cisco-digital-network-architecture/nb-06-cisco-dna-soln-ovw-cte-en.html) and [this](https://www.cisco.com/c/dam/global/ru_kz/solutions/enterprise-networks/digital-network-architecture/pdf/white-paper-c11-736842.pdf) whitepaper.

As this is a blog about (network programmabily) more than anything else we'll focus more on its API.

### DNAC and its API

Cisco DNA architecture comes with a controller, which is called DNAC (Digital Network Architecture Center). Not only does it allow us to design and provision our network, it also allows for monitoring and analytics. And above all, it is an open and extensible platform to allow integration via its API (documentation [here](https://developer.cisco.com/docs/dna-center/api/1-3-3-x/)).

### Overview of the API

Cisco DNAC comes with an intend API, which is a Northbound API that exposes specific capabilities of the DNA Center platform. Reading the API can be a bit overwhelming so have a look at the below categories to understand the several subsections of the API ([source](https://developer.cisco.com/docs/dna-center/#!cisco-dna-center-platform-overview/intent-api-northbound)). Each category below corresponds to a particular section in the [API](https://developer.cisco.com/docs/dna-center/api/1-3-3-x/))
 
**Domain**
- Sites 
- Topology
- Devices
- Client
- Users
- Issues

**Site Management**
- Site Design
- Network Settings
- Software Image Management
- Configuration Templates

**Connectivity**
- Fabric Wired
- Non-Fabric Wireless

**Operational APIs**
- Command Runner
- Network Discovery => seperate blog post (Part 2)
- Path Trace
- File
- Task
- Tags

**Policy**
- Application Policy

**Event Management**
- Event Management

### Credits

Most of the examples I'm using here are using the Postman collection from Nick Russo. Find them at Nick's website ([here](http://njrusmc.net/jobaid/jobaid.html)). Also, Nick has a brilliant course on automation DNAC operations at Pluralsight. Really worth watching it if you want to explore how to run APIs against Cisco's SDA solution. Check out Nick's course on `Automating Cisco DNA Center Operations using APIs` [here](https://app.pluralsight.com/library/courses/automating-cisco-dna-center-operations-using-apis/table-of-contents).

### Getting started example (POSTMAN)

Enough intro...let's dive into some basic programming. Let's try this with POSTMAN first. 

> Note about equipment: For all the examples that involve retrieving information, we will use a Cisco sandbox environment delivered by Cisco Devnet. Go check out Devnet, really brilliant. In this tutorial, I'm using the DNAC Always On sandbox. For later tutorials, we will use a different DNAC as the always-on variant does not allow us to create/update information on the DNAC.

##### A) Preparing POSTMAN

Let's first configure our POSTMAN client a bit. As we are using the DEVNET DNAC Always-On sandbox, we need to set the credentials in POSTMAN. This allows us to use placeholders in our calls, so we can write things like `https://{dnac}` instead of `https://sandboxdnac2.cisco.com` which makes it much more flexible to work with other DNAC instances later on.

![DNAC](/images/2020-04-22-5.png)

##### B) Authentication

As a first step, we need to authenticate to the DNAC. Below is a screenshot from the API documentation. We need to issue a REST call towards the endpoint `/dna/system/api/v1/auth/token` and we will get back a `Token`.

![DNAC](/images/2020-04-22-1.png)

In the below screenshot, you'll see the endpoint set to `/dna/system/api/v1/auth/token`. For the Authorization, we need to use Basic Auth Base64 encoding according the the documentation. This is easy with POSTMAN, as we set the `Type` to Basic Auth and it'll work.

![DNAC](/images/2020-04-22-2.png)

As headers, simply use:

![DNAC](/images/2020-04-22-3.png)

Let's run this call, and you'll see we get a `Token` returned which we need to use in subsequent queries.

![DNAC](/images/2020-04-22-4.png)

As an extra, let's capture the token via some Javascript call and store it in the `api_token` environment variable, so we don't always need to manually update the token once it expires. Store the following code in the `Tests` tab in POSTMAN.

![DNAC](/images/2020-04-22-6.png)

##### C) Get Devices
Now, we have the token, we can easily use it for any subsequent REST call. Let's try it out through a simple call. For this example, we'll use the `Get Device list` API call. The documentation says to use `/dna/intent/api/v1/network-device`. Take a look at below screenshot.

![DNAC](/images/2020-04-22-7.png)


Hope you found this a bit useful. In next posts, I'll dive a bit deeper into DNAC API and provide some simple code (using Python)for some DNA use cases. Stay tuned!