---
title: Cisco SDWAN - Python Requests
date: 2020-05-16T10:32:50+01:00
draft: false
categories:
  - Network Programming
  - Programming
  - All
tags:
  - SDWAN
  - Python
---
### Introduction
In [this](https://blog.wimwauters.com/networkprogrammability/__2020-06-01_sdwan_gettingstarted/) post, we walked through a number of SD-WAN API's using POSTMAN. As a follow-up, we will explore in today's post some Python scripts that implement these APIs.

### Authenticate
As mentioned in the previous post, we need to authenticate with vManage  by sending a POST request to `https://{{vmanage}}:{{port}}/j_security_check`. In the body, we need to specify the username (called `j_username`) and password (`j_password`). Have a look [here](https://sdwan-docs.cisco.com/Product_Documentation/Command_Reference/Command_Reference/vManage_REST_APIs/vManage_REST_APIs_Overview/Using_the_vManage_REST_APIs) to get acquainted with the API

So create a file called `authenticate.py`. You will see that we specify the usual headers and we create a body dictionary containing the username and password. As the SDWAN API returns back a cookie (called `JSESSIONID`), it's easier to work with the Requests Session object. The Session object allows you to persist certain parameters across requests. So if you’re making several requests to the same host, the underlying TCP connection will be reused. Note that we need to use the session object then for the REST API POST call. vManage API does not return content when authenticating but returns an HTML page when authentication is not successful. So what we do is checking for the <html> tag to see if our authentication was successful.

```python
import requests
import json

def login():
    baseurl = "https://10.50.221.182:8443"
    authentication_endpoint = "/j_security_check"
    
    headers = {
        "Content-Type": "application/json",
        "Accept": "application/json"
    }

    body = {    
        "j_username": "admin",
        "j_password": "admin",
    }

    session = requests.session()
    requests.packages.urllib3.disable_warnings()

    url = f"{baseurl}{authentication_endpoint}"
    login_response = session.post(url, data=body, verify=False)

    if b'<html>' in login_response.content:
        print("Login Failed")
        import sys
        sys.exit(0)
    else:
        #print("Login succeeded")
        return session
        
if __name__ == "__main__":
   response = login()
   print(response)
```
### Get device controllers
In this example, we will retrieve a list of device controllers from our SD-WAN setup. Code is pretty easy.

We will first call the login function that we created above. Therefore we added the line `from authenticate import login` at the top of the script. Next, we will specify the correct endpoint, e.g. the API call we want to use is `/dataservice/system/device/controllers`. What rests is simply parsing the response. If you follow along you will note that the response contains a list of devices, so in the script we just loop through the list of devices and we print the required information (in our case the device type and its IP address).

```python
import requests
import json
from authenticate import login

def get_devicecontrollers():
    session = login()

    baseurl = "https://10.50.221.182:8443"

    controller_endpoint = "/dataservice/system/device/controllers"
    url = f"{baseurl}{controller_endpoint}"
    print(url)
    response_controller = session.get(url, verify=False)

    devices = response_controller.json()['data']

    for device in devices:
        print(f"Device controller => {device['deviceType']} with IP address {device['deviceIP']}")

if __name__ == "__main__":
   response = get_devicecontrollers()
```
Executing this script will give you a list of all our device controllers.

```bash
wauterw@WAUTERW-M-65P7 SDWAN_PythonRequests % python3 get_devicecontrollers.py
https://10.50.221.182:8443/dataservice/system/device/controllers
Device controller => vmanage with IP address 10.0.0.11
Device controller => vsmart with IP address 10.0.0.14
Device controller => vbond with IP address 10.0.0.12
```

### Get edge devices
With a small variation to the above script we can also retrieve a list of vEdge devices. To achieve that, we need to call the `/dataservice/system/device/vedges` API and parse the result in a similar way as above script. Oh and you'll see I make some variations on the API call as well. The API allows us to filter what we want to query so we also used a small variation on the above API (e.g. `/dataservice/system/device/vedges?model=vedge-CSR-1000v`) to retrieve only the CSR100V.

```python
import requests
import json
from pprint import pprint
from authenticate import login

def get_vedges():
    session = login()

    baseurl = "https://10.50.221.182:8443"

    vedge_endpoint = "/dataservice/system/device/vedges"
    url = f"{baseurl}{vedge_endpoint}"
    
    response_controller = session.get(url, verify=False).json()
    vedges = response_controller['data']

    for vedge in vedges:
        print(f"vEdge device => {vedge['deviceModel']} with serialnumber {vedge['serialNumber']}")

def get_csr1000v():
    session = login()

    baseurl = "https://10.50.221.182:8443"
    controller_endpoint = "/dataservice/system/device/vedges?model=vedge-CSR-1000v"
    url = f"{baseurl}{controller_endpoint}"
    
    response_controller = session.get(url, verify=False).json()
    devices = response_controller['data']

    for device in devices:
        print(f"CSR1000v device => {device['deviceModel']} with serialnumber {device['serialNumber']}")

if __name__ == "__main__":
    print(20 * "--" + "vEdge devices" + 20 * "--" )
    vedges = get_vedges()
    print(20 * "--" + "CSR1000v devices" + 20 * "--" )
    csr1000v = get_csr1000v()
```
Running the script will indeed provide you with a list of all devices.

```bash
wauterw@WAUTERW-M-65P7 SDWAN_PythonRequests % python3 get_vEdges.py
----------------------------------------vEdge devices----------------------------------------
vEdge device => vedge-CSR-1000v with serialnumber 6F153727
vEdge device => vedge-cloud with serialnumber 900f1b1195feaf5e5b0df8933335e078
***Truncated***
vEdge device => vedge-CSR-1000v with serialnumber f58f847554233766d8d21be40ad8df1e
----------------------------------------CSR1000v devices----------------------------------------
CSR1000v device => vedge-CSR-1000v with serialnumber 6F153727
CSR1000v device => vedge-CSR-1000v with serialnumber EB1CF137
CSR1000v device => vedge-CSR-1000v with serialnumber 382BD76F
CSR1000v device => vedge-CSR-1000v with serialnumber c70df321be57f6d9938bf050e15bf9f2
***Truncated***
CSR1000v device => vedge-CSR-1000v with serialnumber 2779d9ad885751f85e1e96bff1962645
CSR1000v device => vedge-CSR-1000v with serialnumber eae00557d7b111d6c626969dffd053b3
```

### Get templates
In order to retrieve templates from vManage, we will use the `/dataservice/template/device` API. It's very similar to what we did above in terms of the Python script. We will just iterate over the list of templates and print out some information.

```python
import requests
import json
from authenticate import login
from pprint import pprint

def get_templates():
    session = login()

    baseurl = "https://10.50.221.182:8443"
    
    template_endpoint = "/dataservice/template/device"
    url = f"{baseurl}{template_endpoint}"
    
    response_template = session.get(url, verify=False).json()
    #pprint(response_template)

    templates = response_template['data']

    for template in templates:
        print(f"Template => {template['deviceType']} with id {template['templateId']}")

if __name__ == "__main__":
   response = get_templates()
```
Running the script gives us a list of all templates with its corresponding id.

```bash
wauterw@WAUTERW-M-65P7 SDWAN_PythonRequests % python3 get_templates.py 
Template => vedge-CSR-1000v with id 8efa8c0a-c9ef-42f1-a92a-2e2f79c2bbe3
```

If you are interested in seeing the full code, please check out my Github [repo](https://github.com/wiwa1978/blog-hugo-netlify-code/tree/master/SDWAN_PythonRequests).


