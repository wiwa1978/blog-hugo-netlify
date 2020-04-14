---
title: Python and Jinja2 introduction
date: 2020-02-27T07:39:50+01:00
draft: false
categories:
  - Network Programming
  - Programming
tags:
  - Python
  - Jinja2
---

### What is Jinja2?
Jinja is a modern and designer-friendly templating language for Python. The idea is to combine a template with data to produce documents. A template contains variables which are replaced by the values which are passed in when the template is rendered. The easiest would be to give a very straightforward example.

In the below example, we define a template `Hello {{ something }}!`. The items in curly braces will get replaced with the data that we pass to the template. In out case, we pass the value 'World' (through the `template.render` method) to the template. Jinja2 will take this value and insert it into the template. The result will be a static document that contains all the information pre-filled. That resulting document will then be used further in our Python script.

```python
from jinja2 import Template
template = Template("Hello {{ something }}!")
print(template.render(something="World"))
```

Let's dive into some more examples.

### VLANs: template inside code

In this example, we will be taking a vlans object and insert it into a template to create a vlan overview.

As we will be using Jinja templates in our Python script, we need to import the required Jinja libraries (line 1 in the below script). The script is pretty straigthforward. We will first define a dict called `template_vars`. That dict will contain a number of key-value pairs related to our vlan configuration.

We also define an inline template which would use the values from our dict object.

The `vlan_template` is nothing more than a String, but Jinja2 needs it to be a `Template`. Hence, we are using the `jinja2.Template` method to convert it to a `Template`. On that template object, we can now call the `render` method, which will do the heavy lifting. It would take the template variables (from template_vars) and insert them (render) into the template. The surrounding print statement obviously just prints the result of that operation.

```python
import jinja2

template_vars = {
   "vlan_id": 620, 
   "vlan_name": "vlan-620"
}

vlan_template = """
vlan {{ vlan_id }}
   name {{ vlan_name }}
"""

template = jinja2.Template(vlan_template)
print(template.render(template_vars))
```
The output of the above script is:

```bash
WAUTERW-M-65P7:ACI_Python_Requests_Jinja wauterw$ python3 vlan.py 

vlan 620
   name vlan-620
```
This example can be found [here](https://github.com/wiwa1978/blog-hugo-netlify-code/blob/master/ACI_Python_Requests_Jinja/vlans/vlan.py).
### VLANs: template inside code with for-loop

What if we have more than one vlan. How would we address this? 

Fortunately Jinja2 suppors for-loops as well. In the below snippet, we have a dictionary object where the vlan id is the key and the vlan name is the value. The template contains a for loop to iterate over the different key/value pairs from our dictionary object. Pretty neat, no?

```python
import jinja2

vlans = {
   "620": "VLAN-620",
   "621": "VLAN-621", 
   "622": "VLAN-622", 
   "633": "VLAN-623",   
}

template_vars = {
   "vlans": vlans
}

vlan_template = """
{% for vlan_id, vlan_name in vlans.items() %}
vlan {{ vlan_id }}
   name {{ vlan_name }}
{% endfor %}
"""

template = jinja2.Template(vlan_template)
print(template.render(template_vars))
```
The output of above script will be as below:

```bash
WAUTERW-M-65P7:ACI_Python_Requests_Jinja wauterw$ python3 vlans_multiple.py 

vlan 620
   name VLAN-620

vlan 621
   name VLAN-621

vlan 622
   name VLAN-622

vlan 633
   name VLAN-623
```

Notice the white spaces and empty lines? Have a look at the [documentation](https://jinja.palletsprojects.com/en/2.11.x/templates/#whitespace-control) on how to deal with white spaces in more depth. If you want to get rid of these, you could also use the below template (pay attention to the - sign).

```jinja
vlan_template = """
{%- for vlan_id, vlan_name in vlans.items() %}
vlan {{ vlan_id }}
   name {{ vlan_name }}
{%- endfor %}
"""
```

That will generate a more condensed output.

```python
WAUTERW-M-65P7:ACI_Python_Requests_Jinja wauterw$ python3 vlans_multiple.py 

vlan 620
   name VLAN-620
vlan 621
   name VLAN-621
vlan 622
   name VLAN-622
vlan 633
   name VLAN-623
```

This example can be found [here](https://github.com/wiwa1978/blog-hugo-netlify-code/blob/master/ACI_Python_Requests_Jinja/vlans/vlans_multiple.py).

### VLANs: template in seperate file

What if we don't want to have our template defined in our code itself but rather read it in from an external file? Jinja2 has that covered as well.

Let's start with defining the template. It's essentially the same template as the one we used above but now we store it in a specific templates folder. Create in the templates folder a file called vlans.j2.

```jinja
{%- for vlan_id, vlan_name in vlans.items() %}
vlan {{ vlan_id }}
   name {{ vlan_name }}
{%- endfor %}
```
Our Python script will change a bit. We will need to import the `FileSystemLoader` method. The 4th line in our script essentially tells Jinja2 where it can find the template folder (in our case in the templates folder).

To pass the template file, we will use the `get_template` method. This will pass the vlans.j2 file from the templates folder into the template object. On that template object, we then call the render function while passing the vlans dictionary. Again, all pretty straightforward.

```python
from jinja2 import Environment
from jinja2 import FileSystemLoader

my_template = Environment(loader=FileSystemLoader('../templates'))

vlans = {
   "620": "VLAN-620",
   "621": "VLAN-621", 
   "622": "VLAN-622", 
   "623": "VLAN-623",  
}

template = my_template.get_template("vlans.j2")
result = template.render(vlans=vlans)
print(result)
```
As expected the output will be:

```bash
WAUTERW-M-65P7:ACI_Python_Requests_Jinja wauterw$ python3 vlans_file.py 

vlan 620
   name VLAN-620
vlan 621
   name VLAN-621
vlan 622
   name VLAN-622
vlan 623
   name VLAN-623
```
This file can be found [here](https://github.com/wiwa1978/blog-hugo-netlify-code/blob/master/ACI_Python_Requests_Jinja/vlans/vlans_file.py).

### VLANs: template in seperate file, input from YAML file

A small improvement on the previous script could be achieved by reading the vlan information from a YML file. Let's see how that works:

Define a YML file:
```yml
620: VLAN-620
621: VLAN-621 
622: VLAN-622 
623: VLAN-623  
```

The Jinja2 template did not change:
```jinja2
{%- for vlan_id, vlan_name in vlans.items() %}
vlan {{ vlan_id }}
   name {{ vlan_name }}
{%- endfor %}
```
The modified script is below. All that we changed is that we are using the `yaml.load` function to read the YML file into the vlans variable. Hence, the vlans variable is a Python dictionary object that can be used in the `get_template` method.
```python
from jinja2 import Environment
from jinja2 import FileSystemLoader
import yaml

my_template = Environment(loader=FileSystemLoader('../templates'))

vlans = yaml.load(open('vlans.yml'), Loader=yaml.SafeLoader)

template = my_template.get_template("vlans.j2")
result = template.render(vlans=vlans)
print(result)
```
This file can be found [here](https://github.com/wiwa1978/blog-hugo-netlify-code/blob/master/ACI_Python_Requests_Jinja/vlans/vlans_file_yml_file.py). 

### Loopbacks

Here is an additional example, mostly re-iterating what we learned already. Have a look at below to ensure you understand what we learned previously.

```yml
interfaces:
  Loopback2000:
    description: Description for Loopback 2000
    ipv4_addr: 200.200.200.200
    ipv4_mask: 255.255.255.255
  Loopback2001:
    description: Description for Loopback 2001
    ipv4_addr: 200.200.200.201
    ipv4_mask: 255.255.255.255
```

```jinja2
interface Loopback2000
description {{interfaces.Loopback2000.description}}
 ip address {{interfaces.Loopback2000.ipv4_addr}} {{interfaces.Loopback2000.ipv4_mask}}
!
interface Loopback2001
description {{interfaces.Loopback2001.description}}
 ip address {{interfaces.Loopback2001.ipv4_addr}} {{interfaces.Loopback2001.ipv4_mask}}
end
```

```python
import yaml
from jinja2 import Environment, FileSystemLoader

loopbacks = yaml.load(open('loopback.yml'), Loader=yaml.SafeLoader)

env = Environment(loader = FileSystemLoader('../../templates'), trim_blocks=True, lstrip_blocks=True)
template = env.get_template('loopback.j2')
loopback_config = template.render(loopbacks)

print(loopback_config)
```
This example can be found [here](https://github.com/wiwa1978/blog-hugo-netlify-code/tree/master/ACI_Python_Requests_Jinja/loopbacks/normal).

### Loopbacks (variant)
Let's look at a little variant, a little more complex. In previous example, we had multiple loopback interfaces and in the YML file we just listed them sequentially. That's doable for two interfaces but what if you have many more. This is where a for-loop (in Jinja2) comes in very handy. Let's have a look:

The YML file is modified a little compared to previous example. We have basically changed it so we have a list of loopbacks that we can address via it's name.
```YML
interfaces:
  - name: Loopback2001
    description: Description for Loopback 2000
    ipv4_addr: 200.200.200.200
    ipv4_mask: 255.255.255.255
  - name:  Loopback2002
    description: Description for Loopback 2001
    ipv4_addr: 200.200.200.201
    ipv4_mask: 255.255.255.255
```
We will rewrite the Jinja2 template. As mentioned above, we want to use a for loop inside the template. This makes it much more scalable in case we need more interfaces later on.

```jinja
{%- for interface in data.interfaces %}
interface {{interface.name}}
description {{interface.description}}
 ip address {{interface.ipv4_addr}} {{interface.ipv4_mask}}
{%- endfor %}
```
The Python file did not change, but I've added it here for completeness:
```python
import yaml
from jinja2 import Environment, FileSystemLoader

interfaces = yaml.load(open('loopback_variant.yml'), Loader=yaml.SafeLoader)

env = Environment(loader = FileSystemLoader('../../templates'), trim_blocks=True, lstrip_blocks=True)
template = env.get_template('loopback_variant.j2')
loopback_config = template.render(data=interfaces)

print(loopback_config)
```
Running this script will render the list of loopbacks.

```bash
WAUTERW-M-65P7:loopbacks wauterw$ python3 loopback_variant.py 
interface Loopback2001
description Description for Loopback 2000
 ip address 200.200.200.200 255.255.255.255interface Loopback2002
description Description for Loopback 2001
 ip address 200.200.200.201 255.255.255.255
```
This example can be found [here](https://github.com/wiwa1978/blog-hugo-netlify-code/tree/master/ACI_Python_Requests_Jinja/loopbacks/variant).

That'll do for now. In a next post, we will look into applying these concepts to Cisco ACI.

I mentioned the source files already throughout this article. The entire folder containing all examples can be found [here](https://github.com/wiwa1978/blog-hugo-netlify-code/tree/master/ACI_Python_Requests_Jinja).