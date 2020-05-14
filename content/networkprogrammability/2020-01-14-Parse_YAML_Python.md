---
title: Parse YAML file with Python
date: 2020-01-14T10:19:50+01:00
draft: false
categories:
  - Network Programming
  - Programming
  - All
tags:
  - Python
  - YAML
---
### Introduction

In this post we'll go over how to parse a YAML file with Python. Easy enough but I'm using it so often that it helps to have a little post on it. As mentioned before, this blog is mainly to serve my memory :-). Refer to [this](https://blog.wimwauters.com/networkprogrammability/2020-01-11-parse_json_python/) post for getting to know how to parse JSON and check out [this](https://blog.wimwauters.com/networkprogrammability/2020-01-09-parse_xml_python/) one to find out how to parse XML


### Install libraries to work with YAML

First use [this](https://blog.wimwauters.com/2020-01-08-PythonVirtualEnvironment) post to create a virtual environment. 

Python has a nice package called `pyyaml` to work with YAML files. Install it as follows:
```bash
(venv) WAUTERW-M-65P7:Parse_YAML_Python wauterw$ pip3 install pyyaml
Collecting pyyaml
  Downloading https://files.pythonhosted.org/packages/64/c2/b80047c7ac2478f9501676c988a5411ed5572f35d1beff9cae07d3
  ***Truncated***
```
Another nice package is called `pprint`. Note this is not strictly required to work with YAML, but it makes the output of our print statements nicer (in this case to output yaml files).

```bash
(venv) WAUTERW-M-65P7:Parse_YAML_Python wauterw$ pip3 install pprint
Collecting pprint
  Using cached https://files.pythonhosted.org/packages/99/12/b6383259ef85c2b942ab9135f322c0dce83fdca8600d87122d2b0181451f/pprint-0.1.tar.gz
  ***Truncated***
```
### YAML file
First and foremost we need to have a YAML file. A YAML file starts with the three dashes and then it's mostly always key-value pairs. See below YAML file to check things out.

```yaml
---
tenant: "Tenant_Wim"
vrf: "VRF_Wim"
bridge_domains:
  - bd: "BD_Wim"
    gateway: "10.16.100.1"
    mask: "24"
    scope: "shared"
    l3_out: "labinfra-l3out-ro"
```

### Python script to parse YAML
Now onto the real meat of this blog post...how to parse a YAML file. We will use the YAML file we just introduced to test our Python script.

First, we need to import the packages we installed through pip3 earlier in this post. Next, we open the file and use the `read()` function to store everything in a variable called `yml_file`. Note that the type of `yml_file` is just a string, which we can print with `pprint`.

As it's rather difficult to continue working with a String object, we need to convert this String object to a Python type that makes it easier to use. PyYAML comes to the rescue here as the `yaml.load` function is essentially converting the String object to a Python Dict object. As we know, dict objects make it easy to read the different values.

In order to retrieve an object, we simply need to specify the correct key. Speaks for itself I think. A somewhat more complex situation is when we have arrays (or lists) in the YAML file. In the original file, we have an item called `bridge_domain`, which contains a list of bridge domains. You know that because of the `-` sign in the front (in YAML syntax this `-` means it's a list).

Therefore, if we want to retrieve the value of the first bridge domain, we need to specify this with the `[0]`. It's nothing more than an array. Essentially, we tell our code to find the first item (here there is only one) and from that item retrieve the name `['bd']`.
```python
import yaml
from pprint import pprint

# Read YAML configuration
yml_file = open("variables.yaml").read()
pprint(yml_file)
yml_dict = yaml.load(yml_file, yaml.SafeLoader)

tenant_name = yml_dict['tenant']
vrf_name = yml_dict['vrf']
bd_name = yml_dict['bridge_domains'][0]['bd']

print("The variables are: ")
print(f"Tenant name {tenant_name}")
print(f"VRF name {vrf_name}")
print(f"BD name {bd_name}")
```
### Output
Below is the output when we run the script.
```bash
(venv) WAUTERW-M-65P7:Parse_YAML_Python wauterw$ python3 parseYAML.py 
('---\n'
 'tenant: "Tenant_Wim"\n'
 'vrf: "VRF_Wim"\n'
 'bridge_domains:\n'
 '  - bd: "BD_Wim"\n'
 '    gateway: "10.16.100.1"\n'
 '    mask: "24"\n'
 '    scope: "shared"\n'
 '    l3_out: "labinfra-l3out-ro"')
The variables are: 
Tenant name Tenant_Wim
VRF name VRF_Wim
BD name BD_Wim
```
Code can be found at [Github](https://github.com/wiwa1978/blog-hugo-netlify-code/tree/master/Parse_YAML_Python). 

### Deactivating the virtual enviroment
Just for completeness, if you want to deactivate your virtual environment you can do as follows:
```bash
(venv) WAUTERW-M-65P7:Parse_YAML_Python wauterw$ deactivate
WAUTERW-M-65P7:Parse_YAML_Python wauterw$ 
```
And you will see that the `(venv)` in front of the prompt is gone now.
