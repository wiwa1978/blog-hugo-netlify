---
title: NETCONF/YANG and Python - part 2
date: 2020-03-31T12:32:50+01:00
draft: false
categories:
  - Network Programming
  - Programming
tags:
  - IOS XE
  - Python
  - NETCONF
  - YANG
---

### Introduction
This is a follow up post on part 1 where we mainly focused on retrieving information from our devices using NETCONF. This post will focus on how to make configuration changes to our devices configuration. It is highly recommended to first work your way through part 1.

### Set Interface description
Setting the description of a particular interface is pretty straightforward. For simplicity reasons, we will read the configuration from an XML file we have stored in a templates folder.

The interface.xml file in the templates folder looks as follows:
```
<config>
   <interfaces xmlns="urn:ietf:params:xml:ns:yang:ietf-interfaces">
      <interface>
         <name xmlns:nc="urn:ietf:params:xml:ns:netconf:base:1.0">GigabitEthernet3</name>
         <description>{description}</description>
         <type xmlns:ianaift="urn:ietf:params:xml:ns:yang:iana-if-type">ianaift:ethernetCsmacd</type>
         <enabled>false</enabled>
         <ipv4 xmlns="urn:ietf:params:xml:ns:yang:ietf-ip">
            <address>
               <ip>87.2.3.4</ip>
               <netmask>255.255.255.0</netmask>
            </address>
         </ipv4>
         <ipv6 xmlns="urn:ietf:params:xml:ns:yang:ietf-ip"/>
      </interface>
   </interfaces>
</config>
```
This file contains the configuration for the interface in XML format. Note the `{description}` variable in particular. It's in fact a placeholder in the XML document. Hence, we need to pass the description from our Python script, which is done through the format method. It will then render the XML file with our input and fill in the description variable with the correct value.

Also note that we are using the `edit_config` method (see [documentation](https://ncclient.readthedocs.io/en/latest/manager.html?highlight=edit_config#ncclient.manager.Manager.edit_config)). The `edit_config` needs a target and a config which must be rooted in a config element (which is also the reason why the interface XML file is wrapped in a <config></config> block).


```python
from ncclient import manager
from pprint import pprint

router = {
   'ip': 'ios-xe-mgmt-latest.cisco.com',
   'port': '10000',
   'username': 'developer',
   'password': 'C1sco12345'
}

netconf_template = open('templates/interface.xml').read()
netconf_payload = netconf_template.format(description="Changing through Netconf")

print(netconf_payload)

m = manager.connect(host=router['ip'], port=router['port'], username=router['username'],
                    password=router['password'], device_params={'name':'iosxe'}, hostkey_verify=False)

response = m.edit_config(netconf_payload, target="running")

print(response)
```
Let's execute this script.

```bash
(venv) WAUTERW-M-65P7:Netconf_Python wauterw$ python3 edit_config_format.py 
<?xml version="1.0" encoding="UTF-8"?>
<rpc-reply xmlns="urn:ietf:params:xml:ns:netconf:base:1.0" message-id="urn:uuid:09b2c3d8-0ac8-4d02-9a09-851a
39d410fc" xmlns:nc="urn:ietf:params:xml:ns:netconf:base:1.0">
  <ok/>
</rpc-reply>
```
As you can see, we receive an RPC reply from our IOS XE device with a message indicating that the configuration update was successfull. Let's use one of our previous scripts to see the changes. We will read the interface configuration by specifying a filter (that only retrieves information related to GigabitEthernet3)

```bash
(venv) WAUTERW-M-65P7:Netconf_Python wauterw$ python3 netconf_filter_part2.py 
Interface name: GigabitEthernet3
Interface description: Changing through Netconf
Interface IP address: 87.2.3.4
Interface IP netmask: 255.255.255.0
```
As you can see, the information was set successfully (as expected from the OK message of course) in our IOS XE device.

### Set Interface description - small variant

If you are an avid reader of my blog, you might have wondered why we are not using Jinja2 templates to manipulate the interfaces.xml file. You are right! Let's see how that would work.

First we define first a Jinja2 model.

```
<config>
   <interfaces xmlns="urn:ietf:params:xml:ns:yang:ietf-interfaces">
      <interface>
         <name xmlns:nc="urn:ietf:params:xml:ns:netconf:base:1.0">GigabitEthernet3</name>
         <description>{{description}}</description>
         <type xmlns:ianaift="urn:ietf:params:xml:ns:yang:iana-if-type">ianaift:ethernetCsmacd</type>
         <enabled>false</enabled>
         <ipv4 xmlns="urn:ietf:params:xml:ns:yang:ietf-ip">
            <address>
               <ip>87.2.3.4</ip>
               <netmask>255.255.255.0</netmask>
            </address>
         </ipv4>
         <ipv6 xmlns="urn:ietf:params:xml:ns:yang:ietf-ip"/>
      </interface>
   </interfaces>
</config>
```
Pretty similar of course. Note the double curly braces here. Again, we need to pass the description variable to the template. In below snippet you can see how this works. We define a variable called `template_vars` containing a key/value pair. The `get_template` method will retrieve the Jinja2 file and the `render` method will essentially do the heavy lifting of replacing the variable and provide back the 'static'(completed) file. In exactly the same way as previous Python script, we can then just pass the content to the `edit_config` method.

```python
from ncclient import manager
from jinja2 import Environment
from jinja2 import FileSystemLoader

router = {
   'ip': 'ios-xe-mgmt-latest.cisco.com',
   'port': '10000',
   'username': 'developer',
   'password': 'C1sco12345'
}

my_template = Environment(loader=FileSystemLoader('templates'))

template_vars = {
   "description": "Changed again"
}

template = my_template.get_template("interface.j2.xml")
netconf_payload = template.render(template_vars)

# Alternative: it can also be a little easier.
# description = "Changed description"
# template = my_template.get_template("interface.j2.xml")
# netconf_payload = template.render(description=description)
 
print(netconf_payload)

m = manager.connect(host=router['ip'], port=router['port'], username=router['username'],
                    password=router['password'], device_params={'name':'iosxe'}, hostkey_verify=False)

response = m.edit_config(netconf_payload, target="running")

print(response)
```
The code for this blogpost can be found at [Github](https://github.com/wiwa1978/blog-hugo-netlify-code/tree/master/Netconf_Python).

- Set Interface description: [source code](https://github.com/wiwa1978/blog-hugo-netlify-code/blob/master/Netconf_Python/edit_config_format.py)
- Set Interface description - small variant: [source code](https://github.com/wiwa1978/blog-hugo-netlify-code/blob/master/Netconf_Python/edit_config_jinja2.py)
- Templates used: [source code](https://github.com/wiwa1978/blog-hugo-netlify-code/tree/master/Netconf_Python/templates)


