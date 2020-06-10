---
title: RESTCONF with Python
date: 2020-04-04T12:32:50+01:00
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

### Installation

In [this](https://blog.wimwauters.com/networkprogrammability/2020-04-03_restconf_introduction_part2/) post, we went over a number of use cases. We used POSTMAN to understand the details. Here is the list of use cases for your convenience.

- Use Case 1: Retrieve information (IETF YANG model)
- Use Case 2: Adding Loopback interface (IETF YANG model)
- Use Case 3: Changing Loopback interface description (IETF YANG model)
- Use Case 4: Removing Loopback interface (IETF YANG model)
- Use Case 5: Retrieve information (Cisco YANG model)
- Use Case 6: Adding interface (Cisco YANG model)
- Use Case 7: Changing interface description (Cisco YANG model)
- Use Case 8: Removing interface description (Cisco YANG model)

In this post, we will just create a simple Python script for each of these use cases. It's dead easy once you understand the REST API through POSTMAN. Mostly, whenever I come across a new REST API, I would experiment a bit with POSTMAN before I write Python scripts. This post will show you it's very easy once you have gone through the POSTMAN examples.

>Disclaimer: the code in this post is not production-grade code obviously. One should never store the username and password in the clear, not in the source code itself. The examples in the post are merely conceptual and for informational purposes.

## Use Case 1: Retrieve information (IETF YANG model)
The URL to use is: `https://{device['ip']}:{device['port']}/restconf/data/ietf-interfaces:interfaces`

Once we have executed the call via Python requests (GET), we just loop over the list of interfaces and print out the name, description and IP address.

```python
import requests
import json
from pprint import pprint

device = {
   "ip": "ios-xe-mgmt-latest.cisco.com",
   "username": "***",
   "password": "***",
   "port": "9443",
}

headers = {
      "Accept" : "application/yang-data+json", 
      "Content-Type" : "application/yang-data+json", 
   }

module = "ietf-interfaces:interfaces"

url = f"https://{device['ip']}:{device['port']}/restconf/data/{module}"

requests.packages.urllib3.disable_warnings()
response = requests.get(url, headers=headers, auth=(device['username'], device['password']), verify=False).json()

interfaces = response['ietf-interfaces:interfaces']['interface']

for interface in interfaces:
   if bool(interface['ietf-ip:ipv4']): //check if IP address is available
      print(f"{interface['name']} -- {interface['description']} -- {interface['ietf-ip:ipv4']['address'][0]['ip']}")

```
Output is as follows:
```
(venv) WAUTERW-M-65P7:RestConf_Python wauterw$ python3 get_interfaces_ietf.py 
https://ios-xe-mgmt-latest.cisco.com:9443/restconf/data/ietf-interfaces:interfaces
GigabitEthernet1 -- MANAGEMENT INTERFACE - DON'T TOUCH ME -- 10.10.20.48
GigabitEthernet2 -- Configured through NETCONF -- 10.255.255.1
GigabitEthernet3 -- Configured by RESTCONF -- 10.255.250.2
```
Code for this one can be found [here](https://github.com/wiwa1978/blog-hugo-netlify-code/blob/master/RestConf_Python/get_interfaces_ietf.py). A similar example to retrieve the static routes on our IOSXE device can be found [here](https://github.com/wiwa1978/blog-hugo-netlify-code/blob/master/RestConf_Python/get_static_routes_ietf.py).

### Use Case 2: Adding Loopback interface (IETF YANG model)
Next, we'll be adding an interface. Before we do so, let's login to the device via SSH and list all interfaces. We use the following URL again `https://{device['ip']}:{device['port']}/restconf/data/ietf-interfaces:interfaces`.

```
csr1000v-1#show ip interface brief
Interface              IP-Address      OK? Method Status                Protocol
GigabitEthernet1       10.10.20.48     YES other  up                    up
GigabitEthernet2       10.255.255.2    YES other  up                    up
GigabitEthernet3       10.255.250.2    YES manual administratively down down
```
In terms of Python code, it's pretty similar to the example above. In order to add an interface, we need to pass a JSON body. Take a look at the POSTMAN equivalent in the previous post, to understand what we need to pass in the body. Also, as we are creating an interface, we need to use the POST verb.
```python
import requests
import json
from pprint import pprint

device = {
   "ip": "ios-xe-mgmt-latest.cisco.com",
   "username": "***",
   "password": "***",
   "port": "9443",
}

headers = {
      "Accept" : "application/yang-data+json", 
      "Content-Type" : "application/yang-data+json", 
   }

module = "ietf-interfaces:interfaces"

url = f"https://{device['ip']}:{device['port']}/restconf/data/{module}"

payload = {
   "interface": [
    {
      "name": "Loopback10000",
      "description": "Adding loopback10000",
      "type": "iana-if-type:softwareLoopback",
      "enabled": "true",
      "ietf-ip:ipv4": {
        "address": [
          {
            "ip": "192.0.2.60",
            "netmask": "255.255.255.255"
          }
        ]
      }
    }
  ]
 }
requests.packages.urllib3.disable_warnings()
response = requests.post(url, headers=headers, data=json.dumps(payload), auth=(device['username'], device['password']), verify=False)

if (response.status_code == 201):
   print("Successfully added interface")
else:
   print("Issue with adding interface")
    
```
Let's check again through SSH to see if it worked:

```
csr1000v-1#show ip interface brief
Interface              IP-Address      OK? Method Status                Protocol
GigabitEthernet1       10.10.20.48     YES other  up                    up
GigabitEthernet2       10.255.255.2    YES other  up                    up
GigabitEthernet3       10.255.250.2    YES manual administratively down down
Loopback10000          192.0.2.60      YES other  up                    up
```
Code for this one can be found [here](https://github.com/wiwa1978/blog-hugo-netlify-code/blob/master/RestConf_Python/add_interfaces_ietf.py)

### Use Case 3: Changing Loopback interface description (IETF YANG model)
Next, we will change the description of the interface. Before we do so, let's see the current description of our loopback interface.
```
csr1000v-1#show interfaces description
Interface                      Status         Protocol Description
Gi1                            up             up       MANAGEMENT INTERFACE - DON'T TOUCH ME
Gi2                            up             up       Configured through NETCONF
Gi3                            admin down     down     Configured by RESTCONF
Lo10000                        up             up       Adding loopback10000
```
In our Python script, we need to use the following URL `https://{device['ip']}:{device['port']}/restconf/data/ietf-interfaces:interfaces/interface=Loopback10000`. As we are changing the description, we are using the put verb.
```python
import requests
import json
from pprint import pprint

device = {
   "ip": "ios-xe-mgmt-latest.cisco.com",
   "username": "***",
   "password": "***",
   "port": "9443",
}

headers = {
      "Accept" : "application/yang-data+json", 
      "Content-Type" : "application/yang-data+json", 
   }

module = "ietf-interfaces:interfaces"

url = f"https://{device['ip']}:{device['port']}/restconf/data/{module}/interface=Loopback10000"

payload = {
   "interface": [
    {
      "name": "Loopback10000",
      "description": "Adding loopback10000 - changed",
      "type": "iana-if-type:softwareLoopback",
      "enabled": "true",
      "ietf-ip:ipv4": {
        "address": [
          {
            "ip": "192.0.2.60",
            "netmask": "255.255.255.255"
          }
        ]
      }
    }
  ]
 }
requests.packages.urllib3.disable_warnings()
response = requests.put(url, headers=headers, data=json.dumps(payload), auth=(device['username'], device['password']), verify=False)

if (response.status_code == 204):
   print("Successfully updated interface")
else:
   print("Issue with updating interface")
    
```
Let's execute the code. You'll notice our changes went through successfully.
csr1000v-1#show interfaces description
Interface                      Status         Protocol Description
Gi1                            up             up       MANAGEMENT INTERFACE - DON'T TOUCH ME
Gi2                            up             up       Configured through NETCONF
Gi3                            admin down     down     Configured by RESTCONF
Lo10000                        up             up       Adding loopback10000 - changed

Code for this one can be found [here](https://github.com/wiwa1978/blog-hugo-netlify-code/blob/master/RestConf_Python/change_interfaces_ietf.py)

### Use Case 4: Removing Loopback interface (IETF YANG model)
Lastly, let's remove our interface. As a reminder, this is the current list of interfaces.
```
csr1000v-1#show ip interface brief
Interface              IP-Address      OK? Method Status                Protocol
GigabitEthernet1       10.10.20.48     YES other  up                    up
GigabitEthernet2       10.255.255.2    YES other  up                    up
GigabitEthernet3       10.255.250.2    YES manual administratively down down
Loopback10000          192.0.2.60      YES other  up                    up
```
In the Python script, we use the delete verb against the URL `https://{device['ip']}:{device['port']}/restconf/data/ietf-interfaces:interfaces/interface=Loopback10000`.
```python
import requests
import json
from pprint import pprint


device = {
   "ip": "ios-xe-mgmt-latest.cisco.com",
   "username": "***",
   "password": "***",
   "port": "9443",
}

headers = {
      "Accept" : "application/yang-data+json", 
      "Content-Type" : "application/yang-data+json", 
   }

module = "ietf-interfaces:interfaces"

url = f"https://{device['ip']}:{device['port']}/restconf/data/{module}/interface=Loopback10000"
print(url)

requests.packages.urllib3.disable_warnings()
response = requests.delete(url, headers=headers, auth=(device['username'], device['password']), verify=False)

print(response)

if (response.status_code == 204):
   print("Successfully deleted interface")
else:
   print("Issue with deleting interface")
```
And as we can expect, the interface is gone now.
```
csr1000v-1#show ip interface brief
Interface              IP-Address      OK? Method Status                Protocol
GigabitEthernet1       10.10.20.48     YES other  up                    up
GigabitEthernet2       10.255.255.2    YES other  up                    up
GigabitEthernet3       10.255.250.2    YES manual administratively down down
```

Code for this one can be found [here](https://github.com/wiwa1978/blog-hugo-netlify-code/blob/master/RestConf_Python/delete_interfaces_ietf.py)

### Use Case 5: Retrieve information (Cisco YANG model)
What follows is merely the same as what we did already. Instead of using the IETF models, we will be using the Cisco specific YANG models. You'll notice the scripts are exactly the same.

Here is the Python script to retrieve a list of all interfaces.
```python
import requests
import json
from pprint import pprint


device = {
   "ip": "ios-xe-mgmt-latest.cisco.com",
   "username": "***",
   "password": "***",
   "port": "9443",
}

headers = {
      "Accept" : "application/yang-data+json", 
      "Content-Type" : "application/yang-data+json", 
   }

module = "Cisco-IOS-XE-native:native/interface"

url = f"https://{device['ip']}:{device['port']}/restconf/data/{module}"
print(url)

requests.packages.urllib3.disable_warnings()
response = requests.get(url, headers=headers, auth=(device['username'], device['password']), verify=False).json()

#pprint(response)
interfaces = response['Cisco-IOS-XE-native:interface']['GigabitEthernet']

for interface in interfaces:
   print(f"{interface['name']} -- {interface['description']}")
```
This results in the following output:
```
(venv) WAUTERW-M-65P7:RestConf_Python wauterw$ python3 get_interfaces_cisco.py 
https://ios-xe-mgmt-latest.cisco.com:9443/restconf/data/Cisco-IOS-XE-native:native/interface
1 -- MANAGEMENT INTERFACE - DON'T TOUCH ME
2 -- Configured through NETCONF
3 -- Configured by RESTCONF
```
Code for this one can be found [here](https://github.com/wiwa1978/blog-hugo-netlify-code/blob/master/RestConf_Python/get_interfaces_cisco.py)

### Use Case 6: Adding interface (Cisco YANG model)
Let's also add an interface. Current interfaces are:
```
csr1000v-1#show ip interface brief
Interface              IP-Address      OK? Method Status                Protocol
GigabitEthernet1       10.10.20.48     YES other  up                    up
GigabitEthernet2       10.255.255.2    YES other  up                    up
GigabitEthernet3       10.255.250.2    YES manual administratively down down
```
The Python script is exactly the same as use case 2. Do note, we are using a different JSON body (go and look to the corresponding POSTMAN use case in the previous post).
```python
import requests
import json
from pprint import pprint

device = {
   "ip": "ios-xe-mgmt-latest.cisco.com",
   "username": "***",
   "password": "***",
   "port": "9443",
}

headers = {
      "Accept" : "application/yang-data+json", 
      "Content-Type" : "application/yang-data+json", 
   }

module = "Cisco-IOS-XE-native:native/interface"

url = f"https://{device['ip']}:{device['port']}/restconf/data/{module}"

payload = {
   "Cisco-IOS-XE-native:BDI": 
    {
      "name": "60",
      "description": "Set via RestCONF Python",
    }
 }

requests.packages.urllib3.disable_warnings()
response = requests.post(url, headers=headers, data=json.dumps(payload), auth=(device['username'], device['password']), verify=False)

if (response.status_code == 201):
   print("Successfully added interface")
else:
   print("Issue with adding interface")  
```
Here's the corresponding output:
```
csr1000v-1#show ip interface brief
Interface              IP-Address      OK? Method Status                Protocol
GigabitEthernet1       10.10.20.48     YES other  up                    up
GigabitEthernet2       10.255.255.2    YES other  up                    up
GigabitEthernet3       10.255.250.2    YES manual administratively down down
BDI60                  unassigned      YES unset  down                  do
```
Code for this one can be found [here](https://github.com/wiwa1978/blog-hugo-netlify-code/blob/master/RestConf_Python/add_interfaces_cisco.py)

### Use Case 7: Changing interface description (Cisco YANG model)
In this once, we'll change the description of our newly added interface. The current list of interfaces is:
```
csr1000v-1#show interfaces description
Interface                      Status         Protocol Description
Gi1                            up             up       MANAGEMENT INTERFACE - DON'T TOUCH ME
Gi2                            up             up       Configured through NETCONF
Gi3                            admin down     down     Configured by RESTCONF
BD60                           down           down     Set via RestCONF Python
```
Python code is similar to use case 3 above. Again, note we are using the following URL: `https://{device['ip']}:{device['port']}/restconf/data/Cisco-IOS-XE-native:native/interface/BDI=60`.

```python
import requests
import json
from pprint import pprint


device = {
   "ip": "ios-xe-mgmt-latest.cisco.com",
   "username": "***",
   "password": "***",
   "port": "9443",
}

headers = {
      "Accept" : "application/yang-data+json", 
      "Content-Type" : "application/yang-data+json", 
   }

module = "Cisco-IOS-XE-native:native/interface"

url = f"https://{device['ip']}:{device['port']}/restconf/data/{module}/BDI=60"

payload = {
   "Cisco-IOS-XE-native:BDI": 
    {
      "name": "60",
      "description": "Set via RestCONF Python - changed",
    }
 }

requests.packages.urllib3.disable_warnings()
response = requests.put(url, headers=headers, data=json.dumps(payload), auth=(device['username'], device['password']), verify=False)

if (response.status_code == 204):
   print("Successfully updated interface")
else:
   print("Issue with updating interface")
```
Let's check if the changes went through successfully. Yes it worked!
```
csr1000v-1#show interfaces description
Interface                      Status         Protocol Description
Gi1                            up             up       MANAGEMENT INTERFACE - DON'T TOUCH ME
Gi2                            up             up       Configured through NETCONF
Gi3                            admin down     down     Configured by RESTCONF
BD60                           down           down     Set via RestCONF Python - changed
```
Code for this one can be found [here](https://github.com/wiwa1978/blog-hugo-netlify-code/blob/master/RestConf_Python/change_interfaces_cisco.py)

### Use Case 8: Remove interface description (Cisco YANG model)
Lastly, let's remove the interface. We'll verify the current list of interfaces.
```
csr1000v-1#show ip interface brief
Interface              IP-Address      OK? Method Status                Protocol
GigabitEthernet1       10.10.20.48     YES other  up                    up
GigabitEthernet2       10.255.255.2    YES other  up                    up
GigabitEthernet3       10.255.250.2    YES manual administratively down down
BDI60                  unassigned      YES unset  down                  down
```
In the Python script, we will have to use the delete verb again against the real interface.
```python
import requests
import json
from pprint import pprint


device = {
   "ip": "ios-xe-mgmt-latest.cisco.com",
   "username": "***",
   "password": "***",
   "port": "9443",
}

headers = {
      "Accept" : "application/yang-data+json", 
      "Content-Type" : "application/yang-data+json", 
   }

module = "Cisco-IOS-XE-native:native/interface"

url = f"https://{device['ip']}:{device['port']}/restconf/data/{module}/BDI=60"
 
requests.packages.urllib3.disable_warnings()
response = requests.delete(url, headers=headers, auth=(device['username'], device['password']), verify=False)

if (response.status_code == 204):
   print("Successfully deleted interface")
else:
   print("Issue with deleting interface")
```
When you execute above script, you'll see the interface has been removed.
```
csr1000v-1#show ip interface brief
Interface              IP-Address      OK? Method Status                Protocol
GigabitEthernet1       10.10.20.48     YES other  up                    up
GigabitEthernet2       10.255.255.2    YES other  up                    up
GigabitEthernet3       10.255.250.2    YES manual administratively down down
```
Code for this one can be found [here](https://github.com/wiwa1978/blog-hugo-netlify-code/blob/master/RestConf_Python/delete_interfaces_cisco.py)

That's it. A post with quite some redundancy as it's mostly more of the same. Yet I wanted to provide a complete overview of the CRUD operations against a RESTCONF API.

All code can be found on [Github](https://github.com/wiwa1978/blog-hugo-netlify-code/tree/master/RestConf_Python).