---
title: Interacting with IOS XE using Paramiko (part 1)
date: 2020-03-23T12:32:50+01:00
draft: false
categories:
  - Network Programmability
tags:
  - IOS XE
  - Python
  - Paramiko
---

## Introduction

In this blog post, we will explore Paramiko to interact with Cisco devices. We will do a couple of things:
- Execute a single command against a Cisco IOS XE device
- Execute a single command against multiple Cisco IOS XE devices
- Execute a multiple commands against multiple Cisco IOS XE devices
- Set interface descriptions on Cisco IOS devices

For all the examples, we will use a Cisco sandbox environment delivered by [Cisco Devnet](https://developer.cisco.com). Go check out Devnet, really brilliant. To get a list of all sandboxes, check out [this](https://devnetsandbox.cisco.com/RM/Topology) link.

To use the sandbox for this blog post, refer to [this page](https://devnetsandbox.cisco.com/RM/Diagram/Index/27d9747a-db48-4565-8d44-df318fce37ad?diagramType=Topology))

## Installing Paramiko

We will start with instakllation of the Paramiko library. That's pretty straigthforward, please refer to the below output. I'm recommending to work inside a virtual enviroment (see [this](http://localhost:1313/development/2020-03-01/) post to check how to do this)

```bash
WAUTERW-M-65P7:Paramiko_Introduction wauterw$ python3 -m venv venv
WAUTERW-M-65P7:Paramiko_Introduction wauterw$ source venv/bin/activate
(venv) WAUTERW-M-65P7:Paramiko_Introduction wauterw$ pip3 install paramiko
Collecting paramiko
  Downloading https://files.pythonhosted.org/packages/06/1e/1e08baaaf6c3d3df1459fd85f0e7d2d6aa916f33958f151ee1ecc9800971/paramiko-2.7.1-py2.py3-none-any.whl (206kB)
 
***Truncated***

Installing collected packages: pycparser, cffi, six, bcrypt, cryptography, pynacl, paramiko
Successfully installed bcrypt-3.1.7 cffi-1.14.0 cryptography-2.8 paramiko-2.7.1 pycparser-2.20 pynacl-1.3.0 six-1.14.0
```

## Script 1: single command - single device
Below script will execute the `show ip interface brief` against a Cisco IOS XE device. We start obviously with importing the `Paramiko` library. Then we specify the required parameters to login to our device (this is all documented in the above 'sandbox' link).

The main part of this script is the instantiation of the ssh client object. We will first create the ssh client and then make a connection (via the `connect` method) to our Cisco IOS XE device.

Next, we will execute the specified command using the `ssh.exec_command` command. Our SSH client will now send this command to the IOS XE device which will execute the command and return the output via stdout. All we have to do is print the output, using the `stdout.readlines()` method.

```python
import paramiko
import time

host = 'ios-xe-mgmt-latest.cisco.com'
username = 'developer'
password = 'C1sco12345'
port = 8181

command = 'show ip interface brief \n'

ssh = paramiko.SSHClient()
ssh.set_missing_host_key_policy(paramiko.AutoAddPolicy())
ssh.connect(hostname=host,username=username, password=password, port=port, look_for_keys=False, allow_agent=False)

stdin, stdout, stderr = ssh.exec_command(command)
output = stdout.readlines()
print(' '.join(map(str, output)))
```

Running the script will show the interfaces overview.

```bash
(venv) WAUTERW-M-65P7:Paramiko_Introduction wauterw$ python3 ExecuteSingleCommand.py

 Welcome to the DevNet Sandbox for CSR1000v and IOS XE
 
 ***Truncated*** 
 
 Thanks for stopping by.
 
 Interface              IP-Address      OK? Method Status                Protocol
 GigabitEthernet1       10.10.20.48     YES other  up                    up      
 GigabitEthernet2       unassigned      YES manual up                    up      
 GigabitEthernet3       unassigned      YES NVRAM  administratively down down    

 ***Truncated*** 
```

## Script 2: single command - multiple devices

This is a small addition to the above script. We want to run the same command against multiple IOS XE devices. We will take the opportunity to optimize our script a little bit.

For the next scripts, we will put the functionality to create an SSH client and make an SSH connection into a seperate file. Reason is that we will use this multiple times going forward.

We will create a file called `connection.py`:
```python
import paramiko

def get_connection(host, username, password, port):
   ssh = paramiko.SSHClient()
   ssh.set_missing_host_key_policy(paramiko.AutoAddPolicy())
   ssh.connect(hostname=host,username=username, password=password, port=port, look_for_keys=False, allow_agent=False)
   return ssh
```
With that out of the way, we can continue. As we created this seperate `connection` package, we need to import that into our code using `from connection import get_connection`. Next, as we want to target multiple devices, we store the configuration into a dictionary specifying the IP, username, password and port. I will specifiy two devices, but because I only have one IOS XE device available here, I will run my commands against the same device.

Rest of the script is pretty self explanatory. We will loop over the devices dictionary and for each create a SSH client and make a connection. Next we will run the command against the device.

```python
import paramiko
from connection import get_connection
import time

devices = {
   'iosxe1': {
      'ip': 'ios-xe-mgmt-latest.cisco.com',
      'username': 'developer',
      'password': 'C1sco12345',
      'port': '8181'
      },
   'iosxe2': {
      'ip': 'ios-xe-mgmt-latest.cisco.com',
      'username': 'developer',
      'password': 'C1sco12345',
      'port': '8181'
      }
   }

command = 'show ip interface brief \n'

for device in devices.keys(): 
   print(f"Executing on device: {devices[device]['ip']}\n\n")
   ssh = get_connection(host=devices[device]['ip'], username=devices[device]['username'], password=devices[device]['password'], port=devices[device]['port'])
   stdin, stdout, stderr = ssh.exec_command(command)
   output = stdout.readlines()

   print(' '.join(map(str, output)))
```
I won't print the output here, but as you can imagine it will just print the output of the `show ip interface brief` command twice.


## Script 3: multiple commands - multiple devices

After the second script, you will surely think why we are limiting ourselves to a single command. Let's get that fixed in this third script. Easy enough right? Let's see...

We won't go in very much detail as it's mostly based on the script 2 from above. Instead of specifying command as a `String`, we specify command as a `list`.

```python
import paramiko
from connection import get_connection
import time

devices = {
   'iosxe1': {
      'ip': 'ios-xe-mgmt-latest.cisco.com',
      'username': 'developer',
      'password': 'C1sco12345',
      'port': '8181'
      }
   }

commands = ['show ip interface brief\n', 'show run\n']

for device in devices.keys():
   ssh = get_connection(host=devices[device]['ip'], username=devices[device]['username'], password=devices[device]['password'], port=devices[device]['port'])
   for command in commands:
      stdin, stdout, stderr = ssh.exec_command(command)
      output = stdout.readlines()
      time.sleep(2)
      print(' '.join(map(str, output)))

ssh.close()
```
Let's run this script now...

```bash
(venv) WAUTERW-M-65P7:Paramiko_Introduction wauterw$ python3 ExecuteMultipleCommandsMultipleDevicesIssue
.py

 Welcome to the DevNet Sandbox for CSR1000v and IOS XE
 
 ***Truncated***
 
 Interface              IP-Address      OK? Method Status                Protocol
 GigabitEthernet1       10.10.20.48     YES other  up                    up      
 GigabitEthernet2       unassigned      YES manual up                    up      
 GigabitEthernet3       unassigned      YES NVRAM  administratively down down    
 ***Truncated***
      
Traceback (most recent call last):
  File "ExecuteMultipleCommandsMultipleDevicesIssue.py", line 19, in <module>
    stdin, stdout, stderr = ssh.exec_command(command)
  File "/Users/wauterw/Dropbox/Programming/blog-hugo-netlify-code/Paramiko_Introduction/venv/lib/python3.8/site-packages/paramiko/client.py", line 508, in exec_command
    chan = self._transport.open_session(timeout=timeout)
  File "/Users/wauterw/Dropbox/Programming/blog-hugo-netlify-code/Paramiko_Introduction/venv/lib/python3.8/site-packages/paramiko/transport.py", line 875, in open_session
    return self.open_channel(
  File "/Users/wauterw/Dropbox/Programming/blog-hugo-netlify-code/Paramiko_Introduction/venv/lib/python3.8/site-packages/paramiko/transport.py", line 969, in open_channel
    raise SSHException("SSH session not active")
paramiko.ssh_exception.SSHException: SSH session not active
```
You will notice that the script returns an error `SSH session not active`.

The reason for this is that the `exec_command` on Cisco only allows a single command. In order to run multiple commands, you need to run the `invoke_shell()` method. We'll show this in the next script.

## Script 4:  multiple commands - multiple devices: fixed

As mentioned above, instead of using `exec_command()` we need to use the `invoke_shell()` method. Let's change our script to accomodate that.

```python
import paramiko
import time

commands = ['show version\n', 'show run\n']

max_buffer = 65535
devices = {
   'iosxe1': {
      'ip': 'ios-xe-mgmt-latest.cisco.com',
      'username': 'developer',
      'password': 'C1sco12345',
      'port': '8181'
      },
   'iosxe2': {
      'ip': 'ios-xe-mgmt-latest.cisco.com',
      'username': 'developer',
      'password': 'C1sco12345',
      'port': '8181'
      }
   }

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
   new_connection.send("terminal length 0\n")
   output = clear_buffer(new_connection)
    
   for command in commands:
      print(f"Executing command {command}")
      new_connection.send(command)
      time.sleep(2)
      output = new_connection.recv(max_buffer)
      print(output)
           
new_connection.close()
```
When you run this script, you will notice that it now works. We are looping over the devices and for each device, we create an SSH connection and execute both commands (before moving to the next device).

## Script 5:  multiple commands - multiple devices: write to file

If you are following this tutorial and you have executed script 4, you will notice that the terminal output is really messy. The output is just written to the terminal as it is received by the IOS XE device. A small improvement could be to write the output to a text file.

You can do this by changing the content in our for loop.

```python
for device in devices.keys(): 
    outputFileName = device + ' ' + command + '_output.txt'
    connection = get_connection(host=devices[device]['ip'], username=devices[device]['username'], password=devices[device]['password'], port=devices[device]['port'])
    new_connection = connection.invoke_shell()
    output = clear_buffer(new_connection)
    time.sleep(2)
    new_connection.send("terminal length 0\n")
    output = clear_buffer(new_connection)

    with open(outputFileName, 'wb') as f:
        for command in commands:
           print(f"Executing command {command}")
           new_connection.send(command)
           time.sleep(2)
           output = new_connection.recv(max_buffer)
           f.write(output)
    
    new_connection.close()
```
In this script, you can see that we are writing the output of the two commands to an output file per device. So there will be two files, one for each device. Each file containts the output for both these commands.

The code can be found [here](https://github.com/wiwa1978/blog-hugo-netlify-code/tree/master/Paramiko_Introduction). Check out the following files for this tutorial:
- Script 1: ExecuteSingleCommand.py	
- Script 2: ExecuteSingleCommandMultipleDevices.py
- Script 3: ExecuteMultipleCommandsMultipleDevicesIssue.py
- Script 4: ExecuteMultipleCommandsMultipleDevices.py	
- Script 5: ExecuteMultipleCommandsMultipleDevices_toFile.py	I
