---
title: Use Python requests to configure ACI
date: 2020-03-19T07:39:50+01:00
draft: false
categories:
  - Programming
  - Network Programmability
tags:
  - ACI
  - Python
---

## Introduction
In this blog post, we will address how we can use Python to work with Cisco ACI. It'll focus on the very basics, trusting it gives you enough meat to tackle the more complex tasks on your own (e.g. create EPGs, contracts etc...)

## Code to login to the APIC Controller

Cisco ACI uses a controller, called the APIC. The APIC exposes a REST API which is documented [here](https://developer.cisco.com/docs/aci/). I encourage you to go through the information on that page as it gives you essentially all the knowledge you need to work programmatically with ACI. The REST API itself can be found [here](https://www.cisco.com/c/en/us/td/docs/switches/datacenter/aci/apic/sw/4-x/rest-api-config/Cisco-APIC-REST-API-Configuration-Guide-401.html)

First things first, we'll need to login to the APIC and retrieve a token. According to the API guide, the REST API looks as follows.

![APICLogin](/images/2020-03-19-1.png)

So we need to use the POST method to send JSON body the the following URL. https://\<APICclusterIP>/api/aaaLogin.json

```python
import requests
import json

def get_token():  
   url = "https://10.48.109.10/api/aaaLogin.json"

   payload = {
      "aaaUser": {
         "attributes": {
            "name":"admin",
            "pwd":"---"
         }
      }
   }

   headers = {
      "Content-Type" : "application/json"
   }

   requests.packages.urllib3.disable_warnings()
   response = requests.post(url,data=json.dumps(payload), headers=headers, verify=False).json()

   token = response['imdata'][0]['aaaLogin']['attributes']['token']
   return token

def main():
   token = get_token()
   print("The token is: " + token)

if __name__ == "__main__":
   main()
```
In the script, you'll that we prepare the 'payload' json body according to the REST API. We'll insert the username and password in the appropriate place. Nothing more to it.

Then we send the HTTP request itself. We pass the payload via the json.dumps format, we pass the headers and we disable SSL verification (for demo purposes only).

APIC will then send back a JSON response (again as documented in the above screenshot). We can parse the JSON response in Python as it essentially is a dict object. In order to parse the object, you need to look at the response and you'll note the token is in the attribites section (['imdata'][0]['aaaLogin']['attributes']).

Note: I'm creating a header. Strictly speaking this is not required with ACI as the .json extension in the REST URL indicates that we expect JSON format. ACI also supports XML.

When we execute the above script, it will print the token that is returned in the JSON body of the response.

```bash
(venv) WAUTERW-M-65P7:aci_rest_examples wauterw$ python3 aci_login.py 
6jUCAAAAAAAAAAAAAAAAABCOfgYX5j0IXhy0ZKPV7x05BEhyd00p5yVn9s59oAR5ZTlSjCZ9wlTY9VSEJbT2pJJPc02hfPsGh/2C1dmQHe9QmNuS9Qq5avaBcfoS12PUWi1rD4lnJ3ul0w4kfbNex/C2cg1g99v5BlSUa47PFbwsf78ig7Vdv8o0l2ZuxRFp3AF5uaN+1BtOxE9fGlw8JA==
```

This token will now be used for the subsequent REST calls.

## Code to create an ACI tenant
In many cases, your first action on Cisco ACI will be to create a Tenant. We'll also use the REST API documentation to achieve this.

![APICTenant](/images/2020-03-19-2.png)

Again, this is pretty straightforward and is following the same 'template' as the login script above.
```python
import requests
import json
from aci_login import get_token

tenant_name = "Tenant_Python"

def main():
  
   token = get_token()

   url = "https://10.48.109.10/api/mo/uni.json"
   
   payload = {
      "fvTenant": {
         "attributes": {
            "name": tenant_name
         }
      }
   }

   headers = {
      "Cookie" : f"APIC-Cookie={token}", 
   }

   requests.packages.urllib3.disable_warnings()
   response = requests.post(url,data=json.dumps(payload), headers=headers, verify=False)

   if (response.status_code == 200):
      print("Tenant was successfully created")
   else:
      print("Issue with creating tenant")

def get_tenant():
   return tenant_name

if __name__ == "__main__":
   main()
```
First, we will need to retrieve the token. Therefore we call the get_token() function. Note that we imported that function in the third line of the script. According to the documentation, we need to pass that token in a Cookie called APIC-Cookie. The rest of the script is pretty self explanatory I would think.

## The result

Executing the script is as simple as running any other Python script.

```bash
(venv) WAUTERW-M-65P7:aci_rest_examples wauterw$ python3 aci_login.py 
s1ICAAAAAAAAAAAAAAAAAIodgsFHXkastXMmKf7Hr8x/rLC1f/L+z3iXiFbqJXk6ufdu0w3oSXoTgLIVwK4Mqgl4TDHnoDYbmdEcC7GW1GZOZRn+PmrFUzuM8HK98Dc03AzfrsfVp
q0fThEgCfJjzCN2fkR0NUIg2SsLN5ZnPhqw9gWyTKyjT/wfFedZ+BdW1Bxz1kCU/SGqa/cRsPa99A==

(venv) WAUTERW-M-65P7:aci_rest_examples wauterw$ python3 create_tenant.py 
Successfully created tenant
```

In the below screenshot, you'll notice a Tenant called 'Tenant_Python' got created.

![APICLogin](/images/2020-03-19-3.png)

The code can be found at [Github](https://github.com/wiwa1978/blog-hugo-netlify-code/tree/master/ACI_Python_Requests).