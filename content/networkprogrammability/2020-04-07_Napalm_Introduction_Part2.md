---
title: NAPALM Introduction - Part 2
date: 2020-04-07T12:32:50+01:00
draft: false
categories:
  - Network Programming
  - Programming
tags:
  - Python
  - Napalm
---
### Introduction

Let's continue where we left off in part 1. In this post, we will focus more on some specific use cases, like configuration validation and changing configurations onto our devices.

### Use Case: retrieve all interfaces
The below script shows the real power of NAPALM. We will be querying two device types, one of which is an IOSXR device and one of which is a IOSXE device.

The script will retrieve all network interfaces for both devices. It's pretty straightforward. In essence, we are looping over both devices, and for each we:
- make a connection to the device
- open the connection
- call the `get_interfaces()` function
- loop over the items in the result (of the `get_interfaces()` function)
- close the connection 

```python
from napalm import get_network_driver
import json

driver_xe = get_network_driver("ios")
driver_xr = get_network_driver("iosxr")

devices = [{
    <device1 parameters>
   },{
    <device2 parameters>
}]

for device in devices:
   if(device['device_type'] == "cisco_xe" ): 
      device_xe = driver_xe(hostname=device['ip'], username=device['username'], password=device['password'], optional_args={'port':device['port']})
      print("Connected to XE")
      print("---------------")
      device_xe.open()
      for key, value in device_xe.get_interfaces().items() :
         print(key)
      device_xe.close()
   if(device['device_type'] == "cisco_xr" ): 
      device_xr = driver_xr(hostname=device['ip'], username=device['username'], password=device['password'], optional_args={'port':device['port']})
      print("Connected to XR")
      print("---------------")
      device_xr.open()
      for key, value in device_xr.get_interfaces().items() :
         print(key)
      device_xr.close()
```
Output is as follows:

```bash
(venv) WAUTERW-M-65P7:Napalm_Introduction wauterw$ python3 get_interfaces.py 
Connected to XE
---------------
GigabitEthernet1
GigabitEthernet2
GigabitEthernet2.4
GigabitEthernet3
GigabitEthernet3.4
Loopback101
Loopback105
VirtualPortGroup0
Connected to XR
---------------
GigabitEthernet0/0/0/0
GigabitEthernet0/0/0/0.144
GigabitEthernet0/0/0/1
GigabitEthernet0/0/0/2
GigabitEthernet0/0/0/3
GigabitEthernet0/0/0/4
GigabitEthernet0/0/0/5
GigabitEthernet0/0/0/6
Loopback100
Loopback200
MgmtEth0/RP0/CPU0/0
```
### Use Case: Execute simple commands
In this use case, we will execute some simple commands. Nothing complicated but just wanted to add it here for your information. In essence, we create a list of the commands we would like to execute which we then pass to the `cli` method (see [here](https://napalm.readthedocs.io/en/latest/base.html#napalm.base.base.NetworkDriver.cli) for more information).
```python
from napalm import get_network_driver
import json

driver_xr = get_network_driver("iosxr")

device = {
    "device_type": "cisco_xr",
    "ip": "sbx-iosxr-mgmt.cisco.com",
    "username": "admin",
    "password": "C1sco12345",
    "port": "8181",
}

device_xr = driver_xr(hostname=device['ip'], username=device['username'], password=device['password'], optional_args={'port':device['port']})
device_xr.open()
cmds = ['show version', 'show ip int brief']

input = device_xr.cli(cmds)

for i in input.keys():
   input[i] = input[i].split('\n')

print(json.dumps(input, sort_keys=True, indent=4))

device_xr.close()
```
Running this script will give you the following output.
```bash
venv) WAUTERW-M-65P7:Napalm_Introduction wauterw$ python3 execute_commands.py 
{
    "show ip int brief": [
        "Interface                      IP-Address      Status          Protocol Vrf-Name",
        "Loopback100                    1.1.1.100       Up              Up       default ",
        "Loopback200                    1.1.1.200       Shutdown        Down     default ",
        "MgmtEth0/RP0/CPU0/0            10.10.20.175    Up              Up       default ",
        "GigabitEthernet0/0/0/0         unassigned      Shutdown        Down     default ",
        "GigabitEthernet0/0/0/0.144     10.10.20.20     Shutdown        Down     default ",
        "GigabitEthernet0/0/0/1         unassigned      Shutdown        Down     default ",
        "GigabitEthernet0/0/0/2         unassigned      Shutdown        Down     default ",
        "GigabitEthernet0/0/0/3         unassigned      Shutdown        Down     default ",
        "GigabitEthernet0/0/0/4         unassigned      Shutdown        Down     default ",
        "GigabitEthernet0/0/0/5         unassigned      Shutdown        Down     default ",
        "GigabitEthernet0/0/0/6         unassigned      Up              Up       default"
    ],
    "show version": [
        "Cisco IOS XR Software, Version 6.5.3",
        "Copyright (c) 2013-2019 by Cisco Systems, Inc.",
        "",
        "Build Information:",
        " Built By     : ahoang",
        " Built On     : Tue Mar 26 06:52:25 PDT 2019",
        " Built Host   : iox-ucs-019",
        " Workspace    : /auto/srcarchive13/prod/6.5.3/xrv9k/ws",
        " Version      : 6.5.3",
        " Location     : /opt/cisco/XR/packages/",
        "",
        "cisco IOS-XRv 9000 () processor",
        "System uptime is 22 hours 55 minutes"
    ]
}
```
### Use Case: Change configuration (add loopback interface)
This use case will be a little more complicated. We want to add a loopback interface to our device. We will be reading the configuration from a Jinja2 template but also use a NAPALM feature to compare the configuration before committing it to our device. Let's get started.

I will only show the most relevant parts, refer to [here](https://github.com/wiwa1978/blog-hugo-netlify-code/blob/master/Napalm_Introduction/change_config.py) for the entire script. 

First, we will read the [YML file](https://github.com/wiwa1978/blog-hugo-netlify-code/blob/master/Napalm_Introduction/change_config.yml) which contains an overview of the configuration change we want to push through. Next, we will use the Jinja2 library to retrieve the [Jinja2](https://github.com/wiwa1978/blog-hugo-netlify-code/blob/master/Napalm_Introduction/change_config.j2) template and change the placeholder values with the values from our YML file. If this is kind of new to you, I invite you to check out my post on Jinja2 templates [here](https://blog.wimwauters.com/networkprogrammability/2020-02-27-aci_python_requests_jinja_part1).

```python
config_data = yaml.load(open('change_config.yml'), Loader=yaml.SafeLoader)
env = Environment(loader = FileSystemLoader('.'), trim_blocks=True, lstrip_blocks=True)
template = env.get_template('change_config.j2')
response = template.render(config_data)
```
Next, we will use the `load_merge_candidate` function (see [here](https://napalm.readthedocs.io/en/latest/base.html#napalm.base.base.NetworkDriver.load_merge_candidate) for more info). As the documentation tells us, it will merge the existing configuration with the new 'candidate' configuration (but it will NOT make the changes yet). Let's tackle this in below snippet:
```python
facts = device_xr.load_merge_candidate(config=response)
```
Next, we will also use the `compare_config()` method (see [here](https://napalm.readthedocs.io/en/latest/base.html#napalm.base.base.NetworkDriver.compare_config)) to show the difference between the 'running' configuration and the 'candidate' configuration.

```python
print('\nDiff:')
diff = device_xr.compare_config()
print(diff)
```
This will print the following:
```bash
Diff:
--- 
+++ 
@@ -34,12 +34,12 @@
  !
 !
 interface Loopback100
- description ***MERGE LOOPBACK 100****
- ipv4 address 1.1.1.100 255.255.255.255
+ description ***TEMPLATE LOOPBACK 100****
+ ipv4 address 4.4.4.101 255.255.255.255
 !
 interface Loopback200
- description ***MERGE LOOPBACK 200****
- ipv4 address 1.1.1.200 255.255.255.255
+ description ***TEMPLATE LOOPBACK 200****
+ ipv4 address 4.4.4.201 255.255.255.255
```
You can see there is already a Loopback100 and Loopback200 configuration, but it uses different IP addresses.

As mentioned before, up till this point, we have only looked at how the configuration 'would' change but the change itself did not happen yet. Therefore we have to use the `commit_config` (see [here](https://napalm.readthedocs.io/en/latest/base.html#napalm.base.base.NetworkDriver.commit_config)) method which will push/commit the changes to the running configuration.

```python
device_xr.commit_config()
```
In order to test whether our script works, we can use a previous script ([this](https://github.com/wiwa1978/blog-hugo-netlify-code/blob/master/Napalm_Introduction/execute_commands.py) one). You will notice that the Loopback100 and Loopback200 interfaces are indeed configured with the IP addresses we provided in our YML file.

```bash
(venv) WAUTERW-M-65P7:Napalm_Introduction wauterw$ python3 execute_commands.py 
{
    "show ip int brief": [
        "Interface                      IP-Address      Status          Protocol Vrf-Name",
        "Loopback100                    4.4.4.101       Up              Up       default ",
        "Loopback200                    4.4.4.201       Shutdown        Down     default ",
        ***Truncated***
```
### Use Case: Validation
Last use case for NAPALM will be to validate the configuration of our device. Say we want to check whether our device is running a specific software version...or we want to verify if our device has a loopback interface with a given IP address. Let's do that in the following example.

We will create a [YML](https://github.com/wiwa1978/blog-hugo-netlify-code/blob/master/Napalm_Introduction/validation.yml) file which contains the data to be validated. In our example, we will check that the SW version of our network device is running a particular version and we'll also check the IP address of Loopback200.

The key method here is NAPALM's `compliance_report()` (see [here](https://napalm.readthedocs.io/en/latest/base.html#napalm.base.base.NetworkDriver.compliance_report) for more info). As you can read in the documentation, it will verify of the device configuration complies with a particular validation file (our YML file).

```python
device_xr = driver_xr(hostname=device['ip'], username=device['username'], password=device['password'], optional_args={'port':device['port']})
device_xr.open()
response = device_xr.compliance_report("validation.yml")
compliance_status = response['complies']
print(f"Overall compliance status: {compliance_status}")
```
The output of this script will be:
```bash
(venv) WAUTERW-M-65P7:Napalm_Introduction wauterw$ python3 validate.py 
Overall compliance status: False
```
Hmmm, it's false which means either the router is running a wrong software version or the Loopback200 is configured differently. Let's digg a little deeper by adding the following statement to our Python script.

```python
print(json.dumps(response, sort_keys=True, indent=4))
```
Running this script (see below) reveals there is an issue with the Loopback200 interface. To understand why I'm saying this, check both these values: 

- "get_facts": {
        "complies": true,
  }

- "get_interfaces_ip": {
        "complies": false,
  }

In the output, it's even telling us what is missing:

- "missing": [
      "1.1.1.200"
  ]

```
(venv) WAUTERW-M-65P7:Napalm_Introduction wauterw$ python3 validate.py 
Overall compliance status: False
{
    "complies": false,
    "get_facts": {
        "complies": true,
        "extra": [],
        "missing": [],
        "present": {
            "os_version": {
                "complies": true,
                "nested": false
            },
            "vendor": {
                "complies": true,
                "nested": false
            }
        }
    },
    "get_interfaces_ip": {
        "complies": false,
        "extra": [],
        "missing": [],
        "present": {
            "Loopback200": {
                "complies": false,
                "diff": {
                    "complies": false,
                    "extra": [],
                    ***Truncated***
                                "missing": [
                                    "1.1.1.200"
                                ],
                                "present": {}
                            },
                            "nested": true
                        }
                    }
                },
                "nested": true
            }
        }***
    },
    "skipped": []
}
```
Note that this is entirely within expectations as we wanted to check (in our validation YML file) that the Loopback200 is configured to have IP address 1.1.1.200 and we have in fact changed the Loopback200 configuration in the previous use case (we configured it to be 4.4.4.201 if you remember). Hence this validation will indeed fail.

Let's now use [this](https://github.com/wiwa1978/blog-hugo-netlify-code/blob/master/Napalm_Introduction/change_config.py) script to reconfigure the Loopback200 interface to 1.1.1.200. To do so, ensure you change the values in the YML file ([this](https://github.com/wiwa1978/blog-hugo-netlify-code/blob/master/Napalm_Introduction/change_config.yml) one).

Use the following information
```yml
  Loopback200:
    ipv4_addr: 1.1.1.200
    ipv4_mask: 255.255.255.255
```
Let's first change this configuration:

```bash
(venv) WAUTERW-M-65P7:Napalm_Introduction wauterw$ python3 change_config.py 
***Truncated***
Committing ...
``` 
And let's finally re-run the validation script. If all went well, we are expecting no errors any longer as we reconfigured the Loopback200 interface to the correct IP address.

```bash
(venv) WAUTERW-M-65P7:Napalm_Introduction wauterw$ python3 validate.py 
Overall compliance status: True
```
All went well as it seems. 

That's it. Pretty neat! No? Really love NAPALM as it has some really cool features that make it a breeze to work with all kinds of network devices.

For all the examples we discussed in part 1 and part 2, please refer to my [Github](https://github.com/wiwa1978/blog-hugo-netlify-code/tree/master/Napalm_Introduction) repo.
