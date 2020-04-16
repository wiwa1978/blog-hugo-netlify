---
title: RESTCONF intro with Postman - Part 2
date: 2020-04-03T12:32:50+01:00
draft: True
categories:
  - Network Programming
  - Programming
tags:
  - IOS XE
  - Python
  - RESTCONF
  - YANG
---
### Introduction
In part 1, we have introduced RESTCONF and explored it a bit using POSTMAN. We focused mainly on retrieving information from our devices. In this post, I would like to provide some examples on changing and deleting configurations on our device. For simplicity, I will be using the `IETF YANG model` throughout the next examples.

In these examples, it's important to specify the correct headers:

![Restconf](/images/2020-04-03-9.png)

### Use Case 1: Retrieve information (IETF YANG model)


![Restconf](/images/2020-04-03-1.png)

```bash
csr1000v-1#show ip interface brief
Interface              IP-Address      OK? Method Status                Protocol
GigabitEthernet1       10.10.20.48     YES other  up                    up
GigabitEthernet2       10.255.255.2    YES other  administratively down down
GigabitEthernet3       10.255.255.2    YES other  up                    up
Loopback76             unassigned      YES unset  up                    up
Loopback100            unassigned      YES unset  up                    up
Loopback1000           unassigned      YES unset  up                    up
Loopback2000           10.0.0.40       YES other  up                    up
Loopback2500           10.0.0.42       YES other  up                    up
Loopback2600           10.0.0.43       YES other  up                    up
Loopback3001           200.200.200.230 YES other  up                    up
Loopback3002           200.200.200.231 YES other  up                    up
Loopback6000           192.0.2.44      YES other  up                    up
```

### Use Case 2: Adding Loopback interface  (IETF YANG model)


![Restconf](/images/2020-04-03-2.png)

```bash
csr1000v-1#show ip interface brief
Interface              IP-Address      OK? Method Status                Protocol
GigabitEthernet1       10.10.20.48     YES other  up                    up
GigabitEthernet2       10.255.255.2    YES other  administratively down down
GigabitEthernet3       10.255.255.2    YES other  up                    up
Loopback76             unassigned      YES unset  up                    up
Loopback100            unassigned      YES unset  up                    up
Loopback1000           unassigned      YES unset  up                    up
Loopback2000           10.0.0.40       YES other  up                    up
Loopback2500           10.0.0.42       YES other  up                    up
Loopback2600           10.0.0.43       YES other  up                    up
Loopback3001           200.200.200.230 YES other  up                    up
Loopback3002           200.200.200.231 YES other  up                    up
Loopback6000           192.0.2.44      YES other  up                    up
Loopback10000          192.0.2.60      YES other  up                    up
```


### Use Case 3: Changing Loopback interface description  (IETF YANG model)

![Restconf](/images/2020-04-03-3.png)

```bash
csr1000v-1#show interface desc
Interface                      Status         Protocol Description
Gi1                            up             up       MANAGEMENT INTERFACE - DON'T TOUCH ME
Gi2                            admin down     down     Configured by RESTCONF
Gi3                            up             up       Configured by RESTCONF
Lo76                           up             up       scripted with Go
Lo100                          up             up       test it
Lo1000                         up             up
Lo2000                         up             up       Added via Netpalm
Lo2500                         up             up       changed12344
Lo2600                         up             up       Added via Netpalm
Lo3001                         up             up       Description for Loopback 3001
Lo3002                         up             up       Description for Loopback 3002
Lo6000                         up             up       Changed description and IP address via RESTCONF
Lo10000                        up             up       Adding loopback10000 - changed description a bit
```
In the example here I only show the PUT variant. Changing the interface description works for both PUT and PATCH. 

### Use Case 4: Removing Loopback interface  (IETF YANG model)

![Restconf](/images/2020-04-03-4.png)

```bash
csr1000v-1#show ip interface brief
Interface              IP-Address      OK? Method Status                Protocol
GigabitEthernet1       10.10.20.48     YES other  up                    up
GigabitEthernet2       10.255.255.2    YES other  administratively down down
GigabitEthernet3       10.255.255.2    YES other  up                    up
BDI59                  unassigned      YES unset  down                  down
Loopback76             unassigned      YES unset  up                    up
Loopback100            unassigned      YES unset  up                    up
Loopback1000           unassigned      YES unset  up                    up
Loopback2000           10.0.0.40       YES other  up                    up
Loopback2500           10.0.0.42       YES other  up                    up
Loopback2600           10.0.0.43       YES other  up                    up
Loopback3001           200.200.200.230 YES other  up                    up
Loopback3002           200.200.200.231 YES other  up                    up
Loopback6000           192.0.2.44      YES other  up                    up
```

### Use Case 5 : Retrieve information (Cisco YANG model)

![Restconf](/images/2020-04-03-5.png)

```bash
csr1000v-1#show ip interface brief
Interface              IP-Address      OK? Method Status                Protocol
GigabitEthernet1       10.10.20.48     YES other  up                    up
GigabitEthernet2       10.255.255.2    YES other  administratively down down
GigabitEthernet3       10.255.255.2    YES other  up                    up
BDI59                  unassigned      YES unset  down                  down
Loopback76             unassigned      YES unset  up                    up
Loopback100            unassigned      YES unset  up                    up
Loopback1000           unassigned      YES unset  up                    up
Loopback2000           10.0.0.40       YES other  up                    up
Loopback2500           10.0.0.42       YES other  up                    up
Loopback2600           10.0.0.43       YES other  up                    up
Loopback3001           200.200.200.230 YES other  up                    up
Loopback3002           200.200.200.231 YES other  up                    up
Loopback6000           192.0.2.44      YES other  up                    up
```

### Use Case 6 : Adding interface (Cisco YANG model)

![Restconf](/images/2020-04-03-6.png)
```bash
csr1000v-1#show ip interface brief
Interface              IP-Address      OK? Method Status                Protocol
GigabitEthernet1       10.10.20.48     YES other  up                    up
GigabitEthernet2       10.255.255.2    YES other  administratively down down
GigabitEthernet3       10.255.255.2    YES other  up                    up
BDI59                  unassigned      YES unset  down                  down
BDI60                  unassigned      YES unset  down                  down
Loopback76             unassigned      YES unset  up                    up
Loopback100            unassigned      YES unset  up                    up
Loopback1000           unassigned      YES unset  up                    up
Loopback2000           10.0.0.40       YES other  up                    up
Loopback2500           10.0.0.42       YES other  up                    up
Loopback2600           10.0.0.43       YES other  up                    up
Loopback3001           200.200.200.230 YES other  up                    up
Loopback3002           200.200.200.231 YES other  up                    up
Loopback6000           192.0.2.44      YES other  up                    up
```


### Use Case 7: Changing interface description (Cisco YANG model)
![Restconf](/images/2020-04-03-7.png)

```bash
csr1000v-1#show inter descr
Interface                      Status         Protocol Description
Gi1                            up             up       MANAGEMENT INTERFACE - DON'T TOUCH ME
Gi2                            admin down     down     Configured by RESTCONF
Gi3                            up             up       Configured by RESTCONF
BD59                           down           down     meme
BD60                           down           down     Changed via RestCONF
Lo76                           up             up       scripted with Go
Lo100                          up             up       test it
Lo1000                         up             up
Lo2000                         up             up       Added via Netpalm
Lo2500                         up             up       changed12344
Lo2600                         up             up       Added via Netpalm
Lo3001                         up             up       Description for Loopback 3001
Lo3002                         up             up       Description for Loopback 3002
Lo6000                         up             up       Changed description and IP address via RESTCONF

```

### Use Case 8: Remove interface description (Cisco YANG model)
![Restconf](/images/2020-04-03-8.png)
```bash
csr1000v-1#show ip interface brief
Interface              IP-Address      OK? Method Status                Protocol
GigabitEthernet1       10.10.20.48     YES other  up                    up
GigabitEthernet2       10.255.255.2    YES other  administratively down down
GigabitEthernet3       10.255.255.2    YES other  up                    up
BDI59                  unassigned      YES unset  down                  down
Loopback76             unassigned      YES unset  up                    up
Loopback100            unassigned      YES unset  up                    up
Loopback1000           unassigned      YES unset  up                    up
Loopback2000           10.0.0.40       YES other  up                    up
Loopback2500           10.0.0.42       YES other  up                    up
Loopback2600           10.0.0.43       YES other  up                    up
Loopback3001           200.200.200.230 YES other  up                    up
Loopback3002           200.200.200.231 YES other  up                    up
Loopback6000           192.0.2.44      YES other  up                    up
```
