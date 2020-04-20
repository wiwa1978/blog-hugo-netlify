---
title: RESTCONF with Python
date: 2020-04-04T12:32:50+01:00
draft: true
categories:
  - Network Programming
  - Programming
tags:
  - IOS XE
  - Python
  - RESTCONF
  - YANG
---

###
 Installation

```
WAUTERW-M-65P7:RestConf_Python wauterw$ pip3 install requests
***Truncated ***

WAUTERW-M-65P7:RestConf_Python wauterw$ pip3 install pprint
***Truncated***
```

## Use Case 1: Retrieve information (IETF YANG model)

```python
import requests
import json
from pprint import pprint


device = {
   "ip": "ios-xe-mgmt-latest.cisco.com",
   "username": "developer",
   "password": "C1sco12345",
   "port": "9443",
}

headers = {
      "Accept" : "application/yang-data+json", 
      "Content-Type" : "application/yang-data+json", 
   }

module = "ietf-interfaces:interfaces"

url = f"https://{device['ip']}:{device['port']}/restconf/data/{module}"
print(url)

requests.packages.urllib3.disable_warnings()
response = requests.get(url, headers=headers, auth=(device['username'], device['password']), verify=False).json()

interfaces = response['ietf-interfaces:interfaces']['interface']

for interface in interfaces:
   if (interface['name'].find('Gigabit') == 0):
      print(f"{interface['name']} -- {interface['description']} -- {interface['ietf-ip:ipv4']['address'][0]['ip']}")
```

```
(venv) WAUTERW-M-65P7:RestConf_Python wauterw$ python3 get_interfaces_ietf.py 
https://ios-xe-mgmt-latest.cisco.com:9443/restconf/data/ietf-interfaces:interfaces
GigabitEthernet1 -- MANAGEMENT INTERFACE - DON'T TOUCH ME -- 10.10.20.48
GigabitEthernet2 -- Configured through NETCONF -- 10.255.255.1
GigabitEthernet3 -- Configured by RESTCONF -- 10.255.250.2
```

### Use Case 2: Adding Loopback interface (IETF YANG model)

Before
```
csr1000v-1#show ip interface brief
Interface              IP-Address      OK? Method Status                Protocol
GigabitEthernet1       10.10.20.48     YES other  up                    up
GigabitEthernet2       10.255.255.2    YES other  up                    up
GigabitEthernet3       10.255.250.2    YES manual administratively down down
```
```python
import requests
import json
from pprint import pprint


device = {
   "ip": "ios-xe-mgmt-latest.cisco.com",
   "username": "developer",
   "password": "C1sco12345",
   "port": "9443",
}

headers = {
      "Accept" : "application/yang-data+json", 
      "Content-Type" : "application/yang-data+json", 
   }

module = "ietf-interfaces:interfaces"

url = f"https://{device['ip']}:{device['port']}/restconf/data/{module}"
print(url)

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

#print(response)

if (response.status_code == 201):
   print("Successfully added interface")
else:
   print("Issue with adding interface")
    
```


After
```
csr1000v-1#show ip interface brief
Interface              IP-Address      OK? Method Status                Protocol
GigabitEthernet1       10.10.20.48     YES other  up                    up
GigabitEthernet2       10.255.255.2    YES other  up                    up
GigabitEthernet3       10.255.250.2    YES manual administratively down down
Loopback10000          192.0.2.60      YES other  up                    up
```

### Use Case 3: Changing Loopback interface description (IETF YANG model)

Before
```
csr1000v-1#show interfaces description
Interface                      Status         Protocol Description
Gi1                            up             up       MANAGEMENT INTERFACE - DON'T TOUCH ME
Gi2                            up             up       Configured through NETCONF
Gi3                            admin down     down     Configured by RESTCONF
Lo10000                        up             up       Adding loopback10000
```

```python
import requests
import json
from pprint import pprint

device = {
   "ip": "ios-xe-mgmt-latest.cisco.com",
   "username": "developer",
   "password": "C1sco12345",
   "port": "9443",
}

headers = {
      "Accept" : "application/yang-data+json", 
      "Content-Type" : "application/yang-data+json", 
   }

module = "ietf-interfaces:interfaces"

url = f"https://{device['ip']}:{device['port']}/restconf/data/{module}/interface=Loopback10000"
print(url)

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


After
csr1000v-1#show interfaces description
Interface                      Status         Protocol Description
Gi1                            up             up       MANAGEMENT INTERFACE - DON'T TOUCH ME
Gi2                            up             up       Configured through NETCONF
Gi3                            admin down     down     Configured by RESTCONF
Lo10000                        up             up       Adding loopback10000 - changed

### Use Case 4: Removing Loopback interface (IETF YANG model)
```
csr1000v-1#show ip interface brief
Interface              IP-Address      OK? Method Status                Protocol
GigabitEthernet1       10.10.20.48     YES other  up                    up
GigabitEthernet2       10.255.255.2    YES other  up                    up
GigabitEthernet3       10.255.250.2    YES manual administratively down down
Loopback10000          192.0.2.60      YES other  up                    up
```


```python
import requests
import json
from pprint import pprint


device = {
   "ip": "ios-xe-mgmt-latest.cisco.com",
   "username": "developer",
   "password": "C1sco12345",
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

```
csr1000v-1#show ip interface brief
Interface              IP-Address      OK? Method Status                Protocol
GigabitEthernet1       10.10.20.48     YES other  up                    up
GigabitEthernet2       10.255.255.2    YES other  up                    up
GigabitEthernet3       10.255.250.2    YES manual administratively down down
```

### Use Case 5: Retrieve information (Cisco YANG model)

```python
import requests
import json
from pprint import pprint


device = {
   "ip": "ios-xe-mgmt-latest.cisco.com",
   "username": "developer",
   "password": "C1sco12345",
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
```
(venv) WAUTERW-M-65P7:RestConf_Python wauterw$ python3 get_interfaces_cisco.py 
https://ios-xe-mgmt-latest.cisco.com:9443/restconf/data/Cisco-IOS-XE-native:native/interface
1 -- MANAGEMENT INTERFACE - DON'T TOUCH ME
2 -- Configured through NETCONF
3 -- Configured by RESTCONF
```

### Use Case 6: Adding interface (Cisco YANG model)

```
csr1000v-1#show ip interface brief
Interface              IP-Address      OK? Method Status                Protocol
GigabitEthernet1       10.10.20.48     YES other  up                    up
GigabitEthernet2       10.255.255.2    YES other  up                    up
GigabitEthernet3       10.255.250.2    YES manual administratively down down
```

```python
import requests
import json
from pprint import pprint

device = {
   "ip": "ios-xe-mgmt-latest.cisco.com",
   "username": "developer",
   "password": "C1sco12345",
   "port": "9443",
}

headers = {
      "Accept" : "application/yang-data+json", 
      "Content-Type" : "application/yang-data+json", 
   }

module = "Cisco-IOS-XE-native:native/interface"

url = f"https://{device['ip']}:{device['port']}/restconf/data/{module}"
print(url)

payload = {
   "Cisco-IOS-XE-native:BDI": 
    {
      "name": "60",
      "description": "Set via RestCONF Python",
    }
 }

requests.packages.urllib3.disable_warnings()
response = requests.post(url, headers=headers, data=json.dumps(payload), auth=(device['username'], device['password']), verify=False)

#print(response)

if (response.status_code == 201):
   print("Successfully added interface")
else:
   print("Issue with adding interface")  
```

```
csr1000v-1#show ip interface brief
Interface              IP-Address      OK? Method Status                Protocol
GigabitEthernet1       10.10.20.48     YES other  up                    up
GigabitEthernet2       10.255.255.2    YES other  up                    up
GigabitEthernet3       10.255.250.2    YES manual administratively down down
BDI60                  unassigned      YES unset  down                  do
```

### Use Case 7: Changing interface description (Cisco YANG model)

```
csr1000v-1#show interfaces description
Interface                      Status         Protocol Description
Gi1                            up             up       MANAGEMENT INTERFACE - DON'T TOUCH ME
Gi2                            up             up       Configured through NETCONF
Gi3                            admin down     down     Configured by RESTCONF
BD60                           down           down     Set via RestCONF Python
```

```python
import requests
import json
from pprint import pprint


device = {
   "ip": "ios-xe-mgmt-latest.cisco.com",
   "username": "developer",
   "password": "C1sco12345",
   "port": "9443",
}

headers = {
      "Accept" : "application/yang-data+json", 
      "Content-Type" : "application/yang-data+json", 
   }

module = "Cisco-IOS-XE-native:native/interface"

url = f"https://{device['ip']}:{device['port']}/restconf/data/{module}/BDI=60"
print(url)

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

```
csr1000v-1#show interfaces description
Interface                      Status         Protocol Description
Gi1                            up             up       MANAGEMENT INTERFACE - DON'T TOUCH ME
Gi2                            up             up       Configured through NETCONF
Gi3                            admin down     down     Configured by RESTCONF
BD60                           down           down     Set via RestCONF Python - changed
```
### Use Case 8: Remove interface description (Cisco YANG model)

```
csr1000v-1#show ip interface brief
Interface              IP-Address      OK? Method Status                Protocol
GigabitEthernet1       10.10.20.48     YES other  up                    up
GigabitEthernet2       10.255.255.2    YES other  up                    up
GigabitEthernet3       10.255.250.2    YES manual administratively down down
BDI60                  unassigned      YES unset  down                  down
```

```python
import requests
import json
from pprint import pprint


device = {
   "ip": "ios-xe-mgmt-latest.cisco.com",
   "username": "developer",
   "password": "C1sco12345",
   "port": "9443",
}

headers = {
      "Accept" : "application/yang-data+json", 
      "Content-Type" : "application/yang-data+json", 
   }

module = "Cisco-IOS-XE-native:native/interface"

url = f"https://{device['ip']}:{device['port']}/restconf/data/{module}/BDI=60"
print(url)
 
requests.packages.urllib3.disable_warnings()
response = requests.delete(url, headers=headers, auth=(device['username'], device['password']), verify=False)


if (response.status_code == 204):
   print("Successfully deleted interface")
else:
   print("Issue with deleting interface")
```


```
csr1000v-1#show ip interface brief
Interface              IP-Address      OK? Method Status                Protocol
GigabitEthernet1       10.10.20.48     YES other  up                    up
GigabitEthernet2       10.255.255.2    YES other  up                    up
GigabitEthernet3       10.255.250.2    YES manual administratively down down
```