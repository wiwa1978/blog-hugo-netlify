---
title: Interacting with IOS XE and IOS XR using Netmiko
date: 2020-03-25T12:32:50+01:00
draft: false
categories:
  - Network Programming
  - Programming
tags:
  - IOS XE
  - IOS XR
  - Python
  - Netmiko
---

### What is Netmiko

Netmiko is a multi-vendor SSH Python library which makes connecting to network devices via SSH really easy and straightforward. The library  simplifies SSH management to network devices is based on the Paramiko SSH library.

As per the documentation, the purposes of this library are the following:

- Successfully establish an SSH connection to the device
- Simplify the execution of show commands and the retrieval of output data
- Simplify execution of configuration commands including possibly commit actions
- Do the above across a broad set of networking vendors and platforms

### Why Netmiko

The question is: if we have Paramiko, why would be need something like Netmiko. As it turns out, Paramiko can sometimes be quite hard to work with. Netmiko is hiding many details of common devive communication. It uses Paramiko for the underlying SSH connectivity, but provides a higher abstraction for the communication with networking devices.

From the creator of Netmiko Kirk Byers:

> At the time, I had observed that many individuals encountered similar issues with Python-SSH and network devices.

In this post, we have largely used similar use cases as the ones we described in the Paramiko related posts.

### Installing Netmiko
```
WAUTERW-M-65P7:Netmiko_Introduction wauterw$ python3 -m venv venv
WAUTERW-M-65P7:Netmiko_Introduction wauterw$ source venv/bin/activate
(venv) WAUTERW-M-65P7:Netmiko_Introduction wauterw$ pip3 install netmiko
Collecting netmiko
***Truncated***
Installing collected packages: pycparser, cffi, pynacl, bcrypt, cryptography, paramiko, scp, pyserial, future, textfsm, netmiko
  Running setup.py install for future ... done
Successfully installed bcrypt-3.1.7 cffi-1.14.0 cryptography-2.8 future-0.18.2 netmiko-3.1.0 paramiko-2.7.1 pycparser-2.20 pynacl-1.3.0 pyserial-3.4 scp-0.13.2 textfsm-1.1.0
```

### Note about equipment

For all the examples, we will use a Cisco sandbox environment delivered by [Cisco Devnet](https://developer.cisco.com). Go check out Devnet, really brilliant. To get a list of all sandboxes, check out [this](https://devnetsandbox.cisco.com/) link. For this tutorial, I'm using the IOS XE sandbox (see [here](https://devnetsandbox.cisco.com/RM/Diagram/Index/38ded1f0-16ce-43f2-8df5-43a40ebf752e?diagramType=Topology)) and the IOS XR sandbox (see [here](https://devnetsandbox.cisco.com/RM/Diagram/Index/e83cfd31-ade3-4e15-91d6-3118b867a0dd?diagramType=Topology)). 


### Use case: retrieve device uptimes

Let's start with an easy example: we will write a Python script that interacts with an IOS XE device and returns the uptime of the router.

We store both devices (IOS XE and IOS XR) in a list of dictionaries (key:value pairs). It allows up to loop over the devices list and execute the command for each device.

Within the for-loop, we first make a connection via Netmiko's connection method (called Netmiko). This is also the reason why we import the Netmiko library at the top of the script. See [here](https://github.com/ktbyers/netmiko/blob/develop/examples/use_cases/case2_using_dict/conn_with_dict.py) an example from Kirk Byers.

Executing commands is really easy. All we need to do is pass the command we want to execute to the `send_command` method and capture the output.

Important note: The ConnectHandler class is identical to the Netmiko class. This means that we can also use the ConnectHandler. If you would like to do so then couple of things would change:

- the `net_connect = Netmiko(**device)` would become `net_connect = ConnectHandler(**device)` 
- import the ConnectHandler library. `from netmiko import Netmiko` would become `from netmiko import ConnectHandler`.

In the below example, we will simply use the Netmiko method.

```python
from netmiko import Netmiko

devices = [{
    "device_type": "cisco_xr",
    "ip": "sbx-iosxr-mgmt.cisco.com",
    "username": "admin",
    "password": "C1sco12345",
    "port": "8181",
}, {
    "device_type": "cisco_xe",
    "ip": "ios-xe-mgmt-latest.cisco.com",
    "username": "developer",
    "password": "C1sco12345",
    "port": "8181",
}]

for device in devices:
    net_connect = Netmiko(**device)
    output = net_connect.send_command("show version")
    net_connect.disconnect()
    result = output.find('uptime is')
    begin = int(result)
    end = begin + 38
    print(device['ip'] + " => " + output[int(begin):int(end)])
```
In the Github repo, the above script can be found [here](https://github.com/wiwa1978/blog-hugo-netlify-code/blob/master/Netmiko_Introduction/show_uptime.py). Another simple and straightforward example can be found [here](https://github.com/wiwa1978/blog-hugo-netlify-code/blob/master/Netmiko_Introduction/show_ip_int_brief.py). It's a small variant on the above script and retrieves an overview of all interfaces on both devices.


### Use case: Set interface description

In the previous use case, we have shown an example to execute some simple "show" commands, in other words to `read` information from the device. In this use case, we want also to give an example on how to `write` information to the devices.

We will essentially change the interface description of a particular port. It follows the same approach as the previous example. We store the devices in a list of dictionaries and we loop over them in a for-loop. Inside the for-loop, we use the `send_config_set` method (see documentation [here](https://ktbyers.github.io/netmiko/docs/netmiko/index.html)). At a minimum, we need to pass the config_commands, which is an iterable contain all of the configuration commands. Hence, we have created a list called `description_config` which contains a list of all the commands we want to execute (in order). 

Important to understand is that NetMiko will already bring you in the `conf t` state. Next, we will go into 'interface GigabitEthernet3' and then set the description.


```python
from netmiko import ConnectHandler

devices = [{
   "device_type": "cisco_xe",
   "ip": "ios-xe-mgmt-latest.cisco.com",
   "username": "developer",
   "password": "C1sco12345",
   "port": "8181",
}]

description = 'Description set with Netmiko'

description_config = [
    "interface GigabitEthernet3",
    f"description {description}",
]

for device in devices:
   net_connect = ConnectHandler(**device)
   output = net_connect.send_config_set(description_config)
   print(output)
   net_connect.disconnect()
```
Before executing this script, let's check the interface descriptions (pay attention to 'Gi3') by just logging into the device using a terminal.

![interface](/images/2020-03-25-1.png)

Let's now execute our Python script. 

![interface](/images/2020-03-25-2.png)

And then let's check again by logging into the device via our terminal. You will notice that indeed the interface description has been changed to the string we used in our little script.

![interface](/images/2020-03-25-3.png)

### Use case: Variant to 'Set interface description'

In the previous use case, we have read the various commands from a list we declared in our Python script. Netmiko also supports a method (`send_config_from_file` method) to read these commands from a file. It's pretty similar to the script we used in previous use case.

```python
from netmiko import ConnectHandler
import logging

devices = [{
   "device_type": "cisco_xe",
   "ip": "ios-xe-mgmt-latest.cisco.com",
   "username": "developer",
   "password": "C1sco12345",
   "port": "8181",
}]

for device in devices:
   net_connect = ConnectHandler(**device)
   output = net_connect.send_config_from_file('changes.txt')
   print(output)
   net_connect.disconnect()
 ```  
 As you can see, the commands are read from the changes.txt file. See below the contents of that file:

```
# Changes.txt
interface GigabitEthernet3
description This is set via Netmiko and text file
``` 
Let's execute the script:
![interface](/images/2020-03-25-4.png)   
    
And verify if things got changed indeed:

![interface](/images/2020-03-25-5.png)  

### Use case: Add Loopback interfaces
In this use case, we will use some things we learned in previous posts. We will do the following:
- read device information from a YML file
- read variables (containing loopback information) from a YML file
- Use Jinja2 template
- Use Netmiko to configure the loopback interfaces

In case some of this is new, I recommend you to have a look at the following blog posts:
- Parsing YML file: see [here](https://blog.wimwauters.com/networkprogrammability/2020-01-14-parse_yaml_python/)
- Working with Jinja2: see [here](https://blog.wimwauters.com/networkprogrammability/2020-02-27-aci_python_requests_jinja_part1/)

Let's have a look at the YML file first:
```yaml
interfaces:
  - name: Loopback2001
    description: Description for Loopback 2000
    ipv4_addr: 200.200.200.200
    ipv4_mask: 255.255.255.255
  - name:  Loopback2002
    description: Description for Loopback 2001
    ipv4_addr: 200.200.200.201
    ipv4_mask: 255.255.255.255
```
As mentioned, this file contains the required information to configure the loopback interfaces. We will also read the host information from a file called hosts.yml.

```yaml
# Define list of hosts
---
hosts:
  - name: sbx-iosxr-mgmt.cisco.com
    username: admin
    password: C1sco12345
    port: 8181
    cmd: "show running-config"
    type: cisco_xr
  - name: ios-xe-mgmt-latest.cisco.com
    username: developer
    password: C1sco12345
    port: 8181
    cmd: "show running-config"
    type: cisco_xe
```
The Jinja2 file is relative straightforward as well. Essentially, we use a for loop to iterate over the loopback interfaces. Each time, we use the data that is passed to this template from the Python file. 

```jinja
{% for interface in data.interfaces %}
interface {{interface.name}}
description {{interface.description}}
 ip address {{interface.ipv4_addr}} {{interface.ipv4_mask}}
{% endfor %}
```
The Python file is first reading in the host information from the hosts.yml file. The interface information we read from the YML file. The host and interface information is now available to us in a Python dictionary.

Next, we use the Jinja2 library to render the template with the variables. The end result of this is that `loopback_config` contains the completed (absolute) file with all loopback information. 

Next, we simply loop over the devices (hosts) and for each host we create an SSH connection via Netmiko. And finally we use the `send_config_set` method to which we pass the loopback_config information (which contains nothing more than the commands to configure a loopback interface).
```python
import yaml
from netmiko import Netmiko
from jinja2 import Environment, FileSystemLoader

hosts = yaml.load(open('hosts.yml'), Loader=yaml.SafeLoader)
interfaces = yaml.load(open('loopback.yml'), Loader=yaml.SafeLoader)

env = Environment(loader = FileSystemLoader('.'), trim_blocks=True, autoescape=True)
template = env.get_template('loopback.j2')
loopback_config = template.render(data=interfaces)

for host in hosts["hosts"]:
   net_connect = Netmiko(
      host = host["name"],
      username = host["username"],
      password = host["password"],
      port = host["port"],
      device_type = host["type"]
   )
   print(f"Logged into {host['name']} successfully")
   output = net_connect.send_config_set(loopback_config.split("\n"))
```
Let's run this example.

```bash
(base) WAUTERW-M-65P7:test wauterw$ python3 script.py 
***Truncated***
Logged into ios-xe-mgmt-latest.cisco.com successfully
config term
Enter configuration commands, one per line.  End with CNTL/Z.
csr1000v-1(config)#interface Loopback3001
csr1000v-1(config-if)#description Description for Loopback 3001
csr1000v-1(config-if)# ip address 200.200.200.230 255.255.255.255
csr1000v-1(config-if)#interface Loopback3002
csr1000v-1(config-if)#description Description for Loopback 3002
csr1000v-1(config-if)# ip address 200.200.200.231 255.255.255.255
csr1000v-1(config-if)#
csr1000v-1(config-if)#end
```
To see if the interfaces were correctly added, we can use a script we mentioned earlier in this blogpost, e.g [this](https://github.com/wiwa1978/blog-hugo-netlify-code/blob/master/Netmiko_Introduction/show_ip_int_brief.py) one.

```bash
Interface              IP-Address      OK? Method Status                Protocol
GigabitEthernet1       10.10.20.48     YES other  up                    up      
GigabitEthernet2       10.121.23.120   YES other  up                    up      
GigabitEthernet2.4     unassigned      YES manual deleted               down    
***Truncated***     
Loopback3001           200.200.200.230 YES manual up                    up      
Loopback3002           200.200.200.231 YES manual up                    up      
Port-channel1          unassigned      YES unset  down                  down    
VirtualPortGroup0      10.10.20.48     YES unset  up                    up   
```
This particular example can be found [here](https://github.com/wiwa1978/blog-hugo-netlify-code/tree/master/Netmiko_Introduction/loopback).

That's it for this post. Hope you liked this little introduction to Netmiko. To have a view on all examples discussed in this post, you can check out the Github repo [here](https://github.com/wiwa1978/blog-hugo-netlify-code/tree/master/Netmiko_Introduction).