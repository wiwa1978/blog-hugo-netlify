---
title: Use Python requests and Jinja2 to configure ACI
date: 2020-03-11T07:39:50+01:00
draft: false
categories:
  - Network Programming
  - Programming
  - All
tags:
  - ACI
  - Python
  - Jinja2
---

### Introduction
In a [previous](http://blog.wimwauters.com/development/2020-03-19/) post, we introduced how we could interact with Cisco's ACI solution using Python. This post will build on the things we learned in that blog post. 

One of the painpoints was that we had to construct the payload in the script itself. That's difficult to maintain in the long run. So a better way to tackle this would be to use Jinja2 templates. We have come across Jinja2 already before in this blogpost.

In what follows, we are combining the following two concepts:
- make REST calls to API using request library
- use Jinja2 templates to create the JSON bodies


### Code for login functionality
In our case, we will create a template for the login function. The template (Jinja2) file looks as follows:

```jinja
{
   "aaaUser": {
     "attributes": {
       "name": "{{ username }}",
       "pwd": "{{ password }}"
     }
   }
 }
```
As we learned in the Jinja2 introduction post, the items in curly braces will get replaced with the data that we pass to the template. The result will be a static document that contains all the information. That document will then be used further in our Python script.

As we will be using these Jinja templates in our Python script, we need to import the required Jinja libraries (line 3 and 4 in the below script).

The variable `JSON_TEMPLATES` contains the location where we store our template files. In the `get_token` function we will specify which template we are interested in using the `get_template("login.j2.json")` call. The `render` function will replace the dynamic variables in the template (the ones in curly braces) with the proper values that are passed via the render function.

```python
import requests
import json
from jinja2 import Environment
from jinja2 import FileSystemLoader

JSON_TEMPLATES = Environment(loader=FileSystemLoader('templates'), trim_blocks=True)
url = "https://10.48.109.10/"

def get_token(username, password):
   partial_url = "api/aaaLogin.json"
   template = JSON_TEMPLATES.get_template("login.j2.json")
   payload = template.render(username=username, password=password)
   new_url=url+partial_url
   requests.packages.urllib3.disable_warnings()
   response = requests.post(new_url, data=payload, verify=False).json()
   token = response['imdata'][0]['aaaLogin']['attributes']['token']
   print(token)
   return token

def main():
   token = get_token("admin", "---")

if __name__ == "__main__":
   main()
```
The rest is business as usual (already explained in the [previous](http://blog.wimwauters.com/development/2020-03-19/) post:

- send a request via REST POST method, specifying the payload
- parse the token from the returned response

### Code for tenant functionality
To add a tenant, we will use the same methodology. The Jinja template is specified below. 

```jinja
{
   "fvTenant": {
     "attributes": {
       "dn": "uni/tn-{{name}}",
       "name": "{{name}}",
       "rn": "tn-{{name}}",
       "status": "created"
     },
     "children": []
   }
}
```
In order to create a tenant, add the following function to your Python script. It will follow exactly the same structure as the login function. It will:

- point to the correct Jinja template to be used (`tenant.js.json`)
- render the template (by passing the tenant_name)
- create a cookie (required in order to execute REST calls towards APIC)
- send the REST request (POST method as we are creating a tenant)
- interpret the response values

```python
def create_tenant(tenant_name, token):
   partial_url = "api/mo/uni.json"
   template = JSON_TEMPLATES.get_template("tenant.j2.json")
   payload = template.render(name=tenant_name)

   cookies = {'APIC-Cookie': token }
   new_url=url+partial_url
   response = requests.post(new_url, data=payload, cookies=cookies, verify=False)

   if (response.status_code == 200):
      print("Successfully created tenant")
   else:
      print("Issue with creating tenant")
```
The entire script can be found at [Github](https://github.com/wiwa1978/blog-hugo-netlify-code/tree/master/ACI_Python_Requests_Jinja).

Note: while the above looks relatively easy, it's not always as straigthforward as it seems. As the scenarios we are trying the achieve are becoming more complex we might need to explore other solutions. Stay tuned for that as we will address this in a future blog post.