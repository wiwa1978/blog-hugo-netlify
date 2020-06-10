---
title: Model-Driven Telemetry on IOS-XE
date: 2020-06-09T10:19:50+01:00
draft: false
categories:
  - Network Programming
  - Programming
  - All
tags:
  - Python
  - Telemetry
---
### What is Model-Driven Telemetry

Model-driven telemetry is a new approach for network monitoring in which data is streamed from network devices continuously using a push model and provides near real-time access to operational statistics. The idea is that applications can subscribe to specific data items they need, by using standard-based YANG data models. Applications often used for Model Driven Telemetry are TIG (Telegraf, InfluxDB and Grafana) or ELK (Elastisearch, Logstash and Kibana) Stack. For both the application stacks, the idea is that a receiving application collects/receives the pushed telemetry data (Telegraf or Logstash), a database stores these telemetry items (InfluxDB and ElasticSearch) and a visualization application show the telemetry data in nice graphs (Grafana or Kibana). A blog post worth reading on this subject is definitely [this](https://blogs.cisco.com/developer/getting-started-with-model-driven-telemetry) one.

In this blog post, we will primarily focus on how to configure Model Driven Telemetry on the Cisco IOS XE device and not so much on the application stacks (TIG or ELK). Maybe I will keep that for a later blog post if time permits.

### Note about equipment

For all the examples in this post, we will use a Cisco sandbox environment delivered by [Cisco Devnet](https://developer.cisco.com). To get a list of all sandboxes, check out [this](https://devnetsandbox.cisco.com/) link. For this tutorial on Model-Driven Telemetry, I'm using [this](https://devnetsandbox.cisco.com/RM/Diagram/Index/0e053963-b039-4a15-94f6-54db2f5ad61c?diagramType=Topology) sandbox. 

### How to configure Model-Driven Telemetry (MDT)

Configuration of MDT can be done either by:

1) CLI (`telemetry ietf subscription`)
2) programmatically through Netmiko (using `send_config_set`)
3) programmatically through Netconf (using ncclient)
4) programmatically through Restconf 

Below we will go in more detail for each of the above methods. We will enable 5-second CPU utilization telemetry to be pushed out every 5 seconds.

### 1) CLI: create MDT subscriptions

First, let's verify if there are existing Telemetry subscriptions. We can use the `show telemetry ietf subscription all` for this.

```bash
csrv1000#show telemetry ietf subscription all
csrv1000#
```
It appears there no subscriptions which indicates MDT is not configured yet. 

Let's tackle the configuration next:

```bash
csrv1000#conf t
Enter configuration commands, one per line.  End with CNTL/Z.
csrv1000(config)#telemetry ietf subscription 101
csrv1000(config-mdt-subs)#encoding encode-kvgpb
csrv1000(config-mdt-subs)#filter xpath /process-cpu-ios-xe-oper:cpu-usage/cpu-utilization/five-seconds
csrv1000(config-mdt-subs)#stream yang-push
csrv1000(config-mdt-subs)#update-policy periodic 500
csrv1000(config-mdt-subs)#receiver ip address 10.10.20.50 57500 protocol grpc-tcp
csrv1000(config-mdt-subs)#exit
csrv1000(config)#exit
csrv1000#
```
In the above snippet, you will notice a number of interesting things:

- we use kvgpb encoding: key-value Google Protocol Buffers (GPB) encoding which indicates we are using `grpc` (Google RPC)
- we use an xpath to tell the IOS-XE device what data we are interested in
- we use an push intervals of 500 centiseconds (this is 1/100 of a second, so 5 seconds)
- we push the data to a receiver at 10.10.20.50 over port 57500 using the grpc protocol

Once it has been configured, we can look at the current list of subscriptions:

```bash
csrv1000#show telemetry ietf subscription all
  Telemetry subscription brief

  ID               Type        State       Filter type
  --------------------------------------------------------
  101              Configured  Valid       xpath
```
We can also drill down in more detail if we want to:

```bash
csrv1000#show telemetry ietf subscription 101 detail
Telemetry subscription detail:

  Subscription ID: 101
  Type: Configured
  State: Valid
  Stream: yang-push
  Filter:
    Filter type: xpath
    XPath: /process-cpu-ios-xe-oper:cpu-usage/cpu-utilization/five-seconds
  Update policy:
    Update Trigger: periodic
    Period: 500
  Encoding: encode-kvgpb
  Source VRF:
  Source Address:
  Notes:

  Receivers:
    Address                                    Port     Protocol         Protocol Profile
    -----------------------------------------------------------------------------------------
    10.10.20.50                                57500    grpc-tcp
```


### 2)  Create MDT subscriptions using Netmiko

As an alternative, I have written a small script that uses Netmiko to retrieve all subscriptions. For an introduction to Netmiko, please check one of my earlier [posts](https://blog.wimwauters.com/networkprogrammability/2020-03-25-netmiko_introduction/). The script is below:

```python
from netmiko import Netmiko

router = {
   'ip': '10.10.20.30',
   'port': '22',
   'username': 'admin',
   'password': 'Cisco123',
   'device_type': 'cisco_xe'
}

net_connect = Netmiko(**router)
output = net_connect.send_command("show telemetry ietf subscription all") 
print(output)
net_connect.disconnect()
```

Obviously, it will show there are no telemetry subscriptions at the moment.

Next, we will create a telemetry subscription on the IOS-XE device using Netmiko:

```python
from netmiko import Netmiko
from yaml import safe_load
from netmiko import Netmiko
from jinja2 import Environment, FileSystemLoader

router = {
   'ip': '10.10.20.30',
   'port': '22',
   'username': 'admin',
   'password': 'Cisco123',
   'device_type': 'cisco_xe'
}

with open("vars/variables.yml", "r") as handle:
    data = safe_load(handle)

my_template = Environment(loader=FileSystemLoader('templates'))
template = my_template.get_template("netmiko.j2")
netmiko_payload = template.render(data=data)

net_connect = Netmiko(**router)
output = net_connect.send_config_set(netmiko_payload.split('\n')) 
print(f"Added subscriptions successfully. Here are the commands we used:")
print(output)
net_connect.disconnect()
```
Next, let's check if the subscription was created successfully. We just run the above script, it will mention that the subscriptions were added successfully (including the commands that were used):

```bash
➜  netmiko git:(master) ✗ python3 mdt_netmiko.py
Added subscriptions successfully. Here are the commands we used:
config term
Enter configuration commands, one per line.  End with CNTL/Z.
csrv1000(config)#
csrv1000(config)#telemetry ietf subscription 120
csrv1000(config-mdt-subs)# encoding encode-kvgpb
csrv1000(config-mdt-subs)# filter xpath /process-cpu-ios-xe-oper:cpu-usage/cpu-utilization/five-seconds
csrv1000(config-mdt-subs)# stream yang-push
csrv1000(config-mdt-subs)# update-policy periodic 1000
csrv1000(config-mdt-subs)# receiver ip address 10.10.20.50 57500 protocol grpc-tcp
csrv1000(config-mdt-subs)#
csrv1000(config-mdt-subs)#telemetry ietf subscription 130
csrv1000(config-mdt-subs)# encoding encode-kvgpb
csrv1000(config-mdt-subs)# filter xpath /memory-ios-xe-oper:memory-statistics/memory-statistic
csrv1000(config-mdt-subs)# stream yang-push
csrv1000(config-mdt-subs)# update-policy periodic 1000
csrv1000(config-mdt-subs)# receiver ip address 10.10.20.50 57500 protocol grpc-tcp
csrv1000(config-mdt-subs)#
csrv1000(config-mdt-subs)#end
csrv1000#
```
And obviously we can also check the existing subscriptions. No surprise that the subscriptions we just configured are displayed here.

```bash
csrv1000#show telemetry ietf subscription all
  Telemetry subscription brief

  ID               Type        State       Filter type
  --------------------------------------------------------
  120              Configured  Valid       xpath
  130              Configured  Valid       xpath

```

### 3)  Create MDT subscriptions using Netconf (ncclient)

In previous sections we created some MDT subscriptions using the CLI and using Netmiko. However, we could also use Netconf to create these subscriptions. 

###### 3.A - get subscriptions through ncclient
We will however first start with a Python script that uses ncclient to retrieve a list of telemetry subscriptions.

```python
from ncclient import manager
import xmltodict
import xml.dom.minidom

router = {
   'ip': '10.10.20.30',
   'port': '830',
   'username': 'admin',
   'password': 'Cisco123'
}

netconf_filter = """
    <filter>
       <mdt-config-data xmlns="http://cisco.com/ns/yang/Cisco-IOS-XE-mdt-cfg">
         <mdt-subscription>
            <subscription-id></subscription-id>
        </mdt-subscription>
      </mdt-config-data>
    </filter>
"""

m = manager.connect(host=router['ip'], port=router['port'], username=router['username'],
                    password=router['password'], device_params={'name':'iosxe'}, hostkey_verify=False)

running_config = m.get_config('running', netconf_filter)

print(xml.dom.minidom.parseString(str(running_config)).toprettyxml())

m.close_session()
```
When the above script executes, you will get back a list of all existing subscriptions (retrieved through Netconf). Note: in the above script I only indicated I want to get back the subscription id's.

```bash
➜  IOSXE_ModelDrivenTelemetry git:(master) ✗ python3 getsubscriptions.py
<?xml version="1.0" ?>
<rpc-reply message-id="urn:uuid:87ddecaf-3035-4f8d-b00f-db40ca7136a8" xmlns="urn:ietf:params:xml:ns:netconf:base:1.0" xmlns:nc="urn:ietf:params:xml:ns:netconf:base:1.0">
        <data>
                <mdt-config-data xmlns="http://cisco.com/ns/yang/Cisco-IOS-XE-mdt-cfg">
                        <mdt-subscription>
                                <subscription-id>100</subscription-id>
                        </mdt-subscription>
                        <mdt-subscription>
                                <subscription-id>150</subscription-id>
                        </mdt-subscription>
                        <mdt-subscription>
                                <subscription-id>200</subscription-id>
                        </mdt-subscription>
                </mdt-config-data>
        </data>
</rpc-reply>
```

###### 3.B - create subscriptions through ncclient with XML
Next, we will focus on creating subscriptions with Netconf. There are several possibilities, we will start with a pure XML payload that we will read in from an external XML file.

```bash
from ncclient import manager

router = {
   'ip': '10.10.20.30',
   'port': '830',
   'username': 'admin',
   'password': 'Cisco123'
}

netconf_template = open('templates/netconf_mdt_xml.xml').read()
netconf_payload = netconf_template.format(subscription="100", period="5000")

m = manager.connect(host=router['ip'], port=router['port'], username=router['username'],
                    password=router['password'], device_params={'name':'iosxe'}, hostkey_verify=False)

response = m.edit_config(netconf_payload, target="running")
if response.ok:
   print("Subscription added successfully")
```
Next, let's execute this script:
```bash
➜  IOSXE_ModelDrivenTelemetry git:(master) ✗ python3 netconf_mdt_xml.py
Subscription added successfully
```
We can use the CLI directly to check whether the subscription was added:
```bash
csrv1000#show telemetry ietf subscription all
  Telemetry subscription brief

  ID               Type        State       Filter type
  --------------------------------------------------------
  100              Configured  Valid       xpath
```

###### 3.C - create subscriptions through ncclient with Jinja 2 (easy way)
In previous section, we used XML directly. We could also use Jinja2 templates of course. The way I do it below does not bring extremely more value compared to XML but I wanted to provide it as an example nevertheless.

```python
from ncclient import manager
from pprint import pprint
from jinja2 import Environment
from jinja2 import FileSystemLoader

router = {
   'ip': '10.10.20.30',
   'port': '830',
   'username': 'admin',
   'password': 'Cisco123'
}

my_template = Environment(loader=FileSystemLoader('templates'))

template_vars = {
   "subscription": "200",
   "period": "5000"
}

template = my_template.get_template("netconf_mdt_jinja2.j2")
netconf_payload = template.render(template_vars)

m = manager.connect(host=router['ip'], port=router['port'], username=router['username'],
                    password=router['password'], device_params={'name':'iosxe'}, hostkey_verify=False)

response = m.edit_config(netconf_payload, target="running")

if response.ok:
  print("Subscription added successfully")
```
Next let's execute this script:

```bash
➜  IOSXE_ModelDrivenTelemetry git:(master) ✗ python3 netconf_mdt_jinja2.py 
Subscription added successfully
```
We can use the CLI directly to check whether the subscription was added:

```bash
csrv1000#show telemetry ietf subscription all
  Telemetry subscription brief

  ID               Type        State       Filter type
  --------------------------------------------------------
  100              Configured  Valid       xpath
  200              Configured  Valid       xpath
```


######  3.D - create subscriptions through ncclient with Jinja 2 (variant)

Let's start again from a clean state, meaning we have no subscriptions at all. 
```bash
csrv1000#show telemetry ietf subscription all
csrv1000#
```
or we could also use our `get subscriptions` (section 3.A) example from above:
```bash
➜  IOSXE_ModelDrivenTelemetry git:(master) ✗ python3 getsubscriptions.py          
<?xml version="1.0" ?>
<rpc-reply message-id="urn:uuid:63f3a9ca-ca82-457d-b80d-84c09b3e7e16" xmlns="urn:ietf:params:xml:ns:netconf:base:1.0" xmlns:nc="urn:ietf:params:xml:ns:netconf:base:1.0">
        <data/>
</rpc-reply>
```
In this section, we will use Jinja2 at full strength. Meaning, we will create a variables file that contains an overview of the subscriptions we want to add. Then we will use Jinja2 to render these variables in a Jinja2 XML template.

Let's have a look at the variables file:

```yaml
subscriptions:
  160:
    xpath: "/process-cpu-ios-xe-oper:cpu-usage/cpu-utilization/five-seconds"
    period: 1000
  180:
    xpath: "/memory-ios-xe-oper:memory-statistics/memory-statistic"
    period: 1000
```
Next, let's see how the template looks like. You will notice there is not that much difference with the XML templates we were using above. The only remarkable difference is that we use a for-loop in the Jinja2 template to iterate over the subcriptions in the variable file.
```xml
<config>
 <mdt-config-data xmlns="http://cisco.com/ns/yang/Cisco-IOS-XE-mdt-cfg">
{% for id, sub in data["subscriptions"].items() %}
  <mdt-subscription>
   <subscription-id>{{ id }}</subscription-id>
   <base>
    <stream>yang-push</stream>
    <encoding>encode-kvgpb</encoding>
    <period>{{ sub["period"] }}</period>
    <xpath>{{ sub["xpath"] }}</xpath>
   </base>
   <mdt-receivers>
    <address>10.10.20.50</address>
    <port>57500</port>
    <protocol>grpc-tcp</protocol>
   </mdt-receivers>
  </mdt-subscription>
{% endfor %}
 </mdt-config-data>
</config>
```
The Python script to add subscriptions to the IOSXE device using Jinja2 and Netconf looks as follows:
```python
from ncclient import manager
from jinja2 import Environment
from jinja2 import FileSystemLoader
from yaml import safe_load

router = {
   'ip': '10.10.20.30',
   'port': '830',
   'username': 'admin',
   'password': 'Cisco123'
}

with open("vars/variables.yml", "r") as handle:
    data = safe_load(handle)

my_template = Environment(loader=FileSystemLoader('templates'))
template = my_template.get_template("netconf_mdt_jinja2-variant.j2.xml")
netconf_payload = template.render(data=data)

m = manager.connect(host=router['ip'], port=router['port'], username=router['username'],
                    password=router['password'], device_params={'name':'iosxe'}, hostkey_verify=False)

response = m.edit_config(netconf_payload, target="running")

if response.ok:
   print("Subscription added successfully")
```
Let's execute the script:
```bash
➜  IOSXE_ModelDrivenTelemetry git:(master) ✗ python3 netconf_mdt_jinja2-variant.py
Subscription added successfully
```
And subsequently retrieve all the subscriptions through our CLI. You will see both subscriptions (specified in the variable file) have been added successfully.

```bash
csrv1000#show telemetry ietf subscription all
  Telemetry subscription brief

  ID               Type        State       Filter type
  --------------------------------------------------------
  160              Configured  Valid       xpath
  180              Configured  Valid       xpath
```
Alternatively, we can also use the `get subscriptions` Python script we developed in section 3.A.

```bash
➜  IOSXE_ModelDrivenTelemetry git:(master) ✗ python3 getsubscriptions.py          
<?xml version="1.0" ?>
<rpc-reply message-id="urn:uuid:9aa90084-3710-466a-a6b0-860d22254f57" xmlns="urn:ietf:params:xml:ns:netconf:base:1.0" xmlns:nc="urn:ietf:params:xml:ns:netconf:base:1.0">
        <data>
                <mdt-config-data xmlns="http://cisco.com/ns/yang/Cisco-IOS-XE-mdt-cfg">
                        <mdt-subscription>
                                <subscription-id>160</subscription-id>
                        </mdt-subscription>
                        <mdt-subscription>
                                <subscription-id>180</subscription-id>
                        </mdt-subscription>
                </mdt-config-data>
        </data>
</rpc-reply>
```



### 4. Create MDT subscriptions using RestConf 

We can also use RestConf to create MDT subscriptions. Unfortunately, the reserved Devnet Sandbox does not allow RestConf so I will limit myself to provider the Python scripts. They should work, but note that I was not able to test these yet.

First, we will use RestConf to retrieve a list of our subscriptions. 
###### 4.A - Get subscriptions

```python
import requests
import json

device = {
   'ip': '10.10.20.30',
   'port': '443',
   'username': 'admin',
   'password': 'Cisco123'
}

headers = {
      "Accept" : "application/yang-data+json", 
      "Content-Type" : "application/yang-data+json", 
   }

module = "Cisco-IOS-XE-mdt-cfg:mdt-config-data"

url = f"https://{router['ip']}:{router['port']}/restconf/data/{module}"
print(url)

requests.packages.urllib3.disable_warnings()
response = requests.get(url, headers=headers, auth=(device['username'], device['password']), verify=False).json()

print(response)
      
```
Next, we will create a Python script that creates subscriptions on the IOSXE device using Restconf.

###### 4.B - Create subscriptions

```python
import requests
import json

router = {
   'ip': '10.10.20.30',
   'port': '443',
   'username': 'admin',
   'password': 'Cisco123'
}

headers = {
      "Accept" : "application/yang-data+json", 
      "Content-Type" : "application/yang-data+json"
   }

module = "Cisco-IOS-XE-mdt-cfg:mdt-config-data"

url = f"https://{router['ip']}:{router['port']}/restconf/data/{module}"
print(url)

payload = {
    "mdt-config-data": {
        "mdt-subscription": 
            {
                "subscription-id": 100,
                "base": {
                    "stream": "yang-push",
                    "encoding": "encode-kvgpb",
                    "xpath": "/process-cpu-ios-xe-oper:cpu-usage/cpu-utilization/five-seconds",
                    "period": 1000
                },
                "mdt-receivers": {
                    "address": "10.0.19.188",
                    "port": 42518,
                    "protocol": "grpc-tcp"
                }
            }
          }
}

print(payload)

requests.packages.urllib3.disable_warnings()
response = requests.post(url, headers=headers, data=json.dumps(payload), auth=(router['username'], router['password']), verify=False)

print(response)
```

Hope you liked this overview of how to programmatically set Model Driven Telemetry subscriptions. The source code for all these scripts can be found [here](https://github.com/wiwa1978/blog-hugo-netlify-code/tree/master/IOSXE_ModelDrivenTelemetry).







