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

### Vlans: template inside code

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
### Vlans: template inside code with for-loop

What is we have more than one vlan. How would we address this? 

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
### Vlans: template in seperate file

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

my_template = Environment(loader=FileSystemLoader('templates'))

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

### Vlans: template in seperate file, input from YAML file

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

my_template = Environment(loader=FileSystemLoader('templates'))

vlans = yaml.load(open('vlans.yml'), Loader=yaml.SafeLoader)

template = my_template.get_template("vlans.j2")
result = template.render(vlans=vlans)
print(result)
```
That'll do for now. In a next post, we will look into applying these concepts to Cisco ACI.

The code can be found at [Github](https://github.com/wiwa1978/blog-hugo-netlify-code/tree/master/ACI_Python_Requests_Jinja).