---
title: RESTCONF intro with Postman - Part 2
date: 2020-04-03T12:32:50+01:00
draft: false
categories:
  - Network Programming
  - Programming
  - All
tags:
  - IOS XE
  - Python
  - RESTCONF
  - YANG
---
### Introduction
In [part 1](https://blog.wimwauters.com/networkprogrammability/2020-04-02_restconf_introduction_part1/), we have introduced RESTCONF and explored it a bit using POSTMAN. I admit that post was a little dry and could leave some bad taste in your mouth :-). 

In any case, in that post, we focused mainly on retrieving information from our devices. In this post, I would like to provide some examples on changing and deleting configurations on our device and make it a bit more practical. 

If you went through that blog post, you surely will remember RESTCONF uses YANG models and sometimes there are more YANG models to achieve the same. Hence, in what follows, I will be using two YANG models that deal with `interfaces`: the standardised IETF YANG model and Cisco's native Cisco XE interface YANG model.

As a side note in case you are following along: in the following examples, it's important to specify the correct headers:

![Restconf](/images/2020-04-03-9.png)

### Use Case 1: Retrieve information (IETF YANG model)
In this use case, we'll be retrieving a list of device interfaces through the IETF interfaces YANG model.

![Restconf](/images/2020-04-03-1.png)

The response in POSTMAN is quite lengthy, so let's also do the same through SSH (directly to the device).
```bash
csr1000v-1#show ip interface brief
Interface              IP-Address      OK? Method Status                Protocol
GigabitEthernet1       10.10.20.48     YES other  up                    up
GigabitEthernet2       10.255.255.2    YES other  administratively down down
GigabitEthernet3       10.255.255.2    YES other  up                    up
Loopback6000           192.0.2.44      YES other  up                    up
```

### Use Case 2: Adding Loopback interface (IETF YANG model)
Next, let's add an interface through the IETF YANG interface model. In order to achieve that, ensure you define a POST action and provide the required JSON body. An example is indicated in below screenshot.

![Restconf](/images/2020-04-03-2.png)

For simplicity, let's login via SSH to our device and show the interfaces. Note: we could have used also the method from use case 1 of course.
```bash
csr1000v-1#show ip interface brief
Interface              IP-Address      OK? Method Status                Protocol
GigabitEthernet1       10.10.20.48     YES other  up                    up
GigabitEthernet2       10.255.255.2    YES other  administratively down down
GigabitEthernet3       10.255.255.2    YES other  up                    up
Loopback10000          192.0.2.60      YES other  up                    up
```
In any case, you can see the interface got added to our device.

### Use Case 3: Changing Loopback interface description (IETF YANG model)
What if we wanted to update the interface configuration on our device. Again, let's use the IETS YANG model for that. In below screenshot, pay particular attention to the PUT verb and the URL which specifies the exact interface we want to update.

![Restconf](/images/2020-04-03-3.png)

And let's verify again using SSH:
```bash
csr1000v-1#show interface desc
Interface                      Status         Protocol Description
Gi1                            up             up       MANAGEMENT INTERFACE - DON'T TOUCH ME
Gi2                            admin down     down     Configured by RESTCONF
Gi3                            up             up       Configured by RESTCONF
Lo10000                        up             up       Adding loopback10000 - changed description a bit
```
You'll notice that indeed the description has been altered. In the example here I have only shown the PUT variant. Changing the interface description works for both PUT and PATCH. 

### Use Case 4: Removing Loopback interface (IETF YANG model)
How about we remove this interface. Easy enough! Just use the DELETE verb and done.

![Restconf](/images/2020-04-03-4.png)

Let's check it out:
```bash
csr1000v-1#show ip interface brief
Interface              IP-Address      OK? Method Status                Protocol
GigabitEthernet1       10.10.20.48     YES other  up                    up
GigabitEthernet2       10.255.255.2    YES other  administratively down down
GigabitEthernet3       10.255.255.2    YES other  up                    up
```
Indeed, the `Loopback10000` interface has been successfully removed.

### Use Case 5: Retrieve information (Cisco YANG model)
In this use case, we'll retrieve the interface overview through another YANG model. This use case is very similar to use case 1 above but instead of using the IETF interface YANG model, we will be using Cisco's YANG model.

![Restconf](/images/2020-04-03-5.png)
Let's verify through SSH:
```bash
csr1000v-1#show ip interface brief
Interface              IP-Address      OK? Method Status                Protocol
GigabitEthernet1       10.10.20.48     YES other  up                    up
GigabitEthernet2       10.255.255.2    YES other  administratively down down
GigabitEthernet3       10.255.255.2    YES other  up                    up
```

### Use Case 6: Adding interface (Cisco YANG model)
Adding an interface through Cisco's YANG model is just as easy. See below where we add an interface called BDI60.

![Restconf](/images/2020-04-03-6.png)

Let's check things out via SSH again:

```bash
csr1000v-1#show ip interface brief
Interface              IP-Address      OK? Method Status                Protocol
GigabitEthernet1       10.10.20.48     YES other  up                    up
GigabitEthernet2       10.255.255.2    YES other  administratively down down
GigabitEthernet3       10.255.255.2    YES other  up                    up
BDI60                  unassigned      YES unset  down                  down
```
And indeed, you'll see that the interface got added successfully.

### Use Case 7: Changing interface description (Cisco YANG model)
Next, we'll be updating the interface through Cisco's interface YANG model. Pay particular attention to the PUT verb here.

![Restconf](/images/2020-04-03-7.png)

Let's check things out:
```bash
csr1000v-1#show inter descr
Interface                      Status         Protocol Description
Gi1                            up             up       MANAGEMENT INTERFACE - DON'T TOUCH ME
Gi2                            admin down     down     Configured by RESTCONF
Gi3                            up             up       Configured by RESTCONF
BD60                           down           down     Changed via RestCONF
```

### Use Case 8: Remove interface description (Cisco YANG model)
And lastly, let's again remove the interface. Pay attention to the DELETE verb in below screenshot.

![Restconf](/images/2020-04-03-8.png)

Let's check it out once more:
```bash
csr1000v-1#show ip interface brief
Interface              IP-Address      OK? Method Status                Protocol
GigabitEthernet1       10.10.20.48     YES other  up                    up
GigabitEthernet2       10.255.255.2    YES other  administratively down down
GigabitEthernet3       10.255.255.2    YES other  up                    up
```
You'll see the interface got deleted successfully.

In this post, I wanted to provide some practical examples on how to perform CRUD operations on a Cisco device through RESTCONF. Hope it was somehow usefull and hope to see you back soon!
