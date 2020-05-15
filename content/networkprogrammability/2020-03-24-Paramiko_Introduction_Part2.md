---
title: Interacting with IOS XE using Paramiko (part 2)
date: 2020-03-24T12:32:50+01:00
draft: false
categories:
  - Network Programming
  - Programming
  - All
tags:
  - IOS XE
  - Python
  - Paramiko
---
### Introduction
In the [previous](https://blog.wimwauters.com/networkprogrammability/2020-03-23-paramiko_introduction_part1/) post, we experimented a bit with Paramiko. The post was describing a number of use cases to read information from a Cisco IOS XE device. In this post, we will continue on what we achieved but we will focus on making configuration changes onto our Cisco IOS XE device.

### Script 1: Read the 'Interface Description'

We will write a small script to read the interface description via Paramiko. I know we said in the introduction that we would focus on making configuration changes, but we will use the below script in the final script.

>Disclaimer: this is not production-grade code obviously. One should never store the username and password in the clear, not in the source code itself. The examples in the post are merely conceptual and for informational purposes.

```python
import paramiko
from connection import get_connection

def get_description(devices):
   host = 'ios-xe-mgmt-latest.cisco.com'
   username = '***'
   password = '***'
   port = 8181

   command = 'show interface description \n'

   for device in devices.keys(): 
      print(f"Executing on device: {devices[device]['ip']}\n\n")
      ssh = get_connection(host=devices[device]['ip'], username=devices[device]['username'], password=devices[device]['password'], port=devices[device]['port'])
      stdin, stdout, stderr = ssh.exec_command(command)
      output = stdout.readlines()
      newoutput = ' '.join(map(str, output))
      print(newoutput)

```
This above script will return a list of interfaces with its description.

```bash
Interface                      Status         Protocol Description
 Gi1                            up             up       MANAGEMENT INTERFACE - DON'T TOUCH ME
 Gi2                            up             up       Nader was here
 Gi3                            admin down     down     Dummy Description
 Lo2                            up             up       
 Lo3                            up             up       
 Lo72                           up             up       testing go_ssh
 Lo101                          up             up       
 Lo111                          up             up       Creating a Loopback interface using Python
 Lo1234                         up             up       
 Lo1999                         up             up       Creating loopback
```

### Script 2: Change the 'Interface Description'
As you surely know, changing an interface description on an IOS devices goes as follows:

- conf t
- (config) interface GigabitEthernet3
- (config-if) description 'new description'

In the below script, we have specified the above list of commands in a list called `commands`. We will loop over the devices and we will simply execute these commands. Towards the end of the script, we will execute the `GetDescription` method to verify if the description was indeed changed.

```python
import paramiko
import time
from GetDescription import get_description

description = "This is description A"
commands = ['conf t\n', 'interface GigabitEthernet3\n', f"description {description}\n"]

max_buffer = 65535

devices = {
   'iosxe1': {
      'ip': 'ios-xe-mgmt-latest.cisco.com',
      'username': '***',
      'password': '***',
      'port': '8181'
      }
   }

print('Description BEFORE change')
print(get_description(devices))

def get_connection(host, username, password, port):
   ssh = paramiko.SSHClient()
   ssh.set_missing_host_key_policy(paramiko.AutoAddPolicy())
   ssh.connect(hostname=host,username=username, password=password, port=port, look_for_keys=False, allow_agent=False)
   return ssh

def clear_buffer(connection):
   if connection.recv_ready():
      return connection.recv(max_buffer)


for device in devices.keys(): 
   connection = get_connection(host=devices[device]['ip'], username=devices[device]['username'], password=devices[device]['password'], port=devices[device]['port'])
   new_connection = connection.invoke_shell()
   output = clear_buffer(new_connection)
   time.sleep(2)
    
   for command in commands:
      print(f"Executing command {command}")
      new_connection.send(command)
      time.sleep(2)
      output = new_connection.recv(max_buffer)
    
   new_connection.close()

print('Description AFTER change')
print(get_description(devices))
```
The output is below:
```
(venv) WAUTERW-M-65P7:Paramiko_Introduction wauterw$ python3 SetDescription_modular.py 

Description BEFORE change

 Interface                      Status         Protocol Description
 Gi1                            up             up       MANAGEMENT INTERFACE - DON'T TOUCH ME
 Gi2                            up             up       Nader was here
 Gi3                            admin down     down     This description was changed a lot
 Lo2                            up             up       
 Lo3                            up             up       
 Lo72                           up             up       testing go_ssh
 Lo101                          up             up       
 Lo111                          up             up       Creating a Loopback interface using Python
 Lo1234                         up             up       
 Lo1999                         up             up       edited by a logi keyboard do something about it

Executing command conf t

Executing command interface GigabitEthernet3

Executing command description This is description A

Description AFTER change
 
 Interface                      Status         Protocol Description
 Gi1                            up             up       MANAGEMENT INTERFACE - DON'T TOUCH ME
 Gi2                            up             up       Nader was here
 Gi3                            admin down     down     This is description A
 Lo2                            up             up       
 Lo3                            up             up       
 Lo72                           up             up       testing go_ssh
 Lo101                          up             up       
 Lo111                          up             up       Creating a Loopback interface using Python
 Lo1234                         up             up       
 Lo1999                         up             up       edited by a logi keyboard do something about it
None
```
 
The code can be found [here](https://github.com/wiwa1978/blog-hugo-netlify-code/tree/master/Paramiko_Introduction). Check out the following files for this tutorial:

- GetDescription.py
- SetDescription_modular.py


