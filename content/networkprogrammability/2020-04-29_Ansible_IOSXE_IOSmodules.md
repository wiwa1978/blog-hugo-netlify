---
title: Ansible and IOSXE - IOS modules
date: 2020-04-29T10:32:50+01:00
draft: false
categories:
  - Network Programming
  - Programming
  - All
tags:
  - Ansible
  - IOSXE
---
### Introduction
In this blog post, we will use Ansible to interact with our IOS XE devices. Ansible has a very extensive set of Network Modules for various devices and vendors. To interact with IOS XE devices, one could use the IOS module. You can find the documentation [here](https://docs.ansible.com/ansible/latest/modules/list_of_network_modules.html#ios).

We will go through a small number of use cases to see how this module works.

> For all the examples, we will use a Cisco sandbox environment delivered by [Cisco Devnet](https://developer.cisco.com). Go check out Devnet, really brilliant. To get a list of all sandboxes, check out [this](https://devnetsandbox.cisco.com/RM/Topology) link. For this blog post, we will use the IOS XE sandbox.

### Show version and show interfaces

Let's start with a simple use case, which is showing the version and a list of interfaces on our IOS XE device. To do so, we will be using the `ios-command` network module which allows us to run arbitrary commands on IOS devices and display the response from the device. This module is documented [here](https://docs.ansible.com/ansible/latest/modules/ios_command_module.html#ios-command-module).

Let's start with defining a `hosts` file. For this example, it will just include a single entry, which points to the FQDN of the sandbox IOSXE device.

```yaml
[iosxe]
ios-xe-mgmt-latest.cisco.com
```

Next, let's define the variables in a `group_vars` file called `iosxe.yaml`

```yaml
ansible_connection: local
ansible_python_interpreter: /usr/bin/python3
host_key_checking: False
ansible_ssh_user: developer
ansible_ssh_pass: ***
ansible_port: 8181
```

Next, let's create the script. You will see that we issue the `show version` and `show ip interface brief` commands one by one and we capture the response in a variable through Ansible `register` method. We then use the `debug` method to visualize the response.

```yaml
- name: Show examples
  hosts: iosxe
  gather_facts: no

  tasks:
  - name: GATHERING FACTS
    ios_facts:
      gather_subset: hardware

  - name: run show version
    ios_command:
      commands: show version
    register: version
  
  - name: display version
    debug:
      var: version["stdout_lines"][0]

  - name: run show ip int brief
    ios_command:
      commands: show ip interface brief
    register: interfaces

  - name: display interfaces
    debug:
      var: interfaces["stdout_lines"][0]
```
The above script can be run as follows. As expected, we will get back the output of both commands.

```bash
wauterw@WAUTERW-M-65P7 ios_modules % ansible-playbook command_show.yaml -i hosts

PLAY [Show examples] ******************************************************************************************************************************************************

TASK [GATHERING FACTS] ****************************************************************************************************************************************************
ok: [ios-xe-mgmt-latest.cisco.com]

TASK [run show version] ***************************************************************************************************************************************************
ok: [ios-xe-mgmt-latest.cisco.com]

TASK [display version] ****************************************************************************************************************************************************
ok: [ios-xe-mgmt-latest.cisco.com] => {
    "version[\"stdout_lines\"][0]": [
        "Cisco IOS XE Software, Version 16.11.01a",
        ***Truncated***
        "Configuration register is 0x2102"
    ]
}

TASK [run show ip int brief] **********************************************************************************************************************************************
ok: [ios-xe-mgmt-latest.cisco.com]

TASK [display interfaces] *************************************************************************************************************************************************
ok: [ios-xe-mgmt-latest.cisco.com] => {
    "interfaces[\"stdout_lines\"][0]": [
        "Interface              IP-Address      OK? Method Status                Protocol",
        "GigabitEthernet1       10.10.20.48     YES NVRAM  up                    up      ",
        "GigabitEthernet2       unassigned      YES NVRAM  administratively down down    ",
        "GigabitEthernet3       unassigned      YES NVRAM  administratively down down    ",
        "Loopback192            192.0.1.2       YES manual up                    up"
    ]
}

PLAY RECAP ****************************************************************************************************************************************************************
ios-xe-mgmt-latest.cisco.com : ok=5    changed=0    unreachable=0    failed=0    skipped=0    rescued=0    ignored=0   
```
In previous example, we issues both commands seperately (in a seperate tasks that is). We can also write it as follows, which is a little shorter.

```yaml
- name: Getting started
  hosts: iosxe
  gather_facts: no

  tasks:
  - name: GATHERING FACTS
    ios_facts:
      gather_subset: hardware

  - name: run multiple commands
    ios_command:
      commands: 
        - show version
        - show ip interface brief
    register: output

  - name: display all
    debug:
      var: output["stdout_lines"]
```
With the above script, you will get back a list of responses. If you would only be interested to see the output of the `show ip interface brief`, you could do this by looking at the `output["stdout_lines"][1]` output.

### Add/Delete single interface
Next, we will add and delete some interfaces (instead of just reading them from our device as we did in the use case above). Let's start with adding some interfaces. 

###### A) Add single interface
In the below Ansible script, we will create a loopback interface on our IOS XE device. We will use the `ios_interface` module for that. Information can be found [here](https://docs.ansible.com/ansible/latest/modules/ios_interface_module.html#ios-interface-module). As you can see, it allows you to specify the MTU size, the duplex mode and many other things. There is no option to specify an IP address. There is another module for that, called `ios_l3_interface` (see [documentation](https://docs.ansible.com/ansible/latest/modules/ios_l3_interface_module.html#ios-l3-interface-module)). This one takes care of all layer 3 configuration on your interfaces. Hence we are using that one to set the IP address of our interface.

```yaml
---
- name: Configure loopback using ios_interface
  hosts: iosxe
  gather_facts: no

  tasks:
    - name: Creating loopback 
      ios_interface:
        name: Loopback100
        enabled: True
        description: Loopback interface 100 created with Ansible
    - name: Assign IP to loopback
      ios_l3_interface:
        name: Loopback100
        ipv4: 10.10.10.10/32
```
Let's execute this playbook:

```bash
wauterw@WAUTERW-M-65P7 ios_modules % ansible-playbook -i hosts loopback_create_single.yaml 

PLAY [Configure loopback using ios_interface] *****************************************************************************************************************************

TASK [Creating loopback] **************************************************************************************************************************************************
changed: [ios-xe-mgmt-latest.cisco.com]

TASK [Assign IP to loopback] **********************************************************************************************************************************************
changed: [ios-xe-mgmt-latest.cisco.com]

PLAY RECAP ****************************************************************************************************************************************************************
ios-xe-mgmt-latest.cisco.com : ok=2    changed=2    unreachable=0    failed=0    skipped=0    rescued=0    ignored=0   
```
When logging into the device using SSH, you can verify that the interfaces was added successfully.
```bash
csr1000v-1#show ip interface brief
Interface              IP-Address      OK? Method Status                Protocol
GigabitEthernet1       10.10.20.48     YES NVRAM  up                    up
GigabitEthernet2       unassigned      YES NVRAM  administratively down down
GigabitEthernet3       unassigned      YES NVRAM  administratively down down
Loopback100            10.10.10.10     YES manual up                    up
```

###### B) Delete interface
Next, let's delete the interface we just created through an Ansible script. It's easy as we will use the `ios_interface` module again and put the state to absent. Ansible will take care of the removal of the interface.

```yaml
---
- name: Delete interfaces
  hosts: iosxe
  gather_facts: no

  tasks:
  - name: Delete loopback 100
    ios_interface:
      name: Loopback100
      state: absent
```
Let's execute this playbook:

```bash
wauterw@WAUTERW-M-65P7 ios_modules % ansible-playbook -i hosts loopback_remove_single.yaml 

PLAY [Delete interfaces] **************************************************************************************************************************************************

TASK [Delete loopback 100] ************************************************************************************************************************************************
changed: [ios-xe-mgmt-latest.cisco.com]

PLAY RECAP ****************************************************************************************************************************************************************
ios-xe-mgmt-latest.cisco.com : ok=1    changed=1    unreachable=0    failed=0    skipped=0    rescued=0    ignored=0   
```
And indeed, when we login through SSH, you will see the interface got successfully deleted.

```bash
csr1000v-1#show ip interface brief
Interface              IP-Address      OK? Method Status                Protocol
GigabitEthernet1       10.10.20.48     YES NVRAM  up                    up
GigabitEthernet2       unassigned      YES NVRAM  administratively down down
GigabitEthernet3       unassigned      YES NVRAM  administratively down down
```

### Add/Delete multiple interfaces

Next, let's make it a bit more complex. Let's add multiple interfaces to our IOSXE device. We will do so through iterating over some interfaces we list in a variables file.

If you want to follow along, create a file called `vars/loopback.yaml` with the following content:

```yaml
---
loopbacks: 
  - loopbackname: "Loopback100"
    ip: "10.16.100.100"
    mask: "32"
  - loopbackname: "Loopback101"
    ip: "10.16.100.101"
    mask: "32"
  - loopbackname: "Loopback102"
    ip: "10.16.100.102"
    mask: "32"
  - loopbackname: "Loopback103"
    ip: "10.16.100.103"
    mask: "32"
```

###### A) Add multiple interfaces
In order to add the interfaces from the above variable file, we will loop over them. We start by specifying the `vars_files`. Next, we create the loopback interfaces by looping over the `loopbacks` defined in the variables file with the `with_items` method. We repeat the same method for assigning IP addresses to the interfaces.

```yaml
---
- name: Configure loopback using ios_interface
  hosts: iosxe
  gather_facts: no
  vars_files:
    - vars/loopbacks.yaml

  tasks:
    - name: Creating loopback 
      ios_interface:
        name: "{{ item.loopbackname }}"
        enabled: True
        description: Loopback interface 100 created with Ansible
      with_items: "{{ loopbacks }}"

    - name: Assign IP to loopback
      ios_l3_interface:
        name: "{{ item.loopbackname }}"
        ipv4: "{{ item.ip }}/{{ item.mask }}"
      with_items: "{{ loopbacks }}"

    - name: Make interface down
      ios_interface:
        name: "{{ item.loopbackname }}"
        enabled: False
      with_items: "{{ loopbacks }}"
```
Let's execute the Ansible script:

```bash
wauterw@WAUTERW-M-65P7 ios_modules % ansible-playbook -i hosts loopback_create_multiple.yaml 

PLAY [Configure loopback using ios_interface] *****************************************************************************************************************************

TASK [Creating loopback] **************************************************************************************************************************************************
changed: [ios-xe-mgmt-latest.cisco.com] => (item={'loopbackname': 'Loopback100', 'ip': '10.16.100.100', 'mask': '32'})
changed: [ios-xe-mgmt-latest.cisco.com] => (item={'loopbackname': 'Loopback101', 'ip': '10.16.100.101', 'mask': '32'})
changed: [ios-xe-mgmt-latest.cisco.com] => (item={'loopbackname': 'Loopback102', 'ip': '10.16.100.102', 'mask': '32'})
changed: [ios-xe-mgmt-latest.cisco.com] => (item={'loopbackname': 'Loopback103', 'ip': '10.16.100.103', 'mask': '32'})

TASK [Assign IP to loopback] **********************************************************************************************************************************************
changed: [ios-xe-mgmt-latest.cisco.com] => (item={'loopbackname': 'Loopback100', 'ip': '10.16.100.100', 'mask': '32'})
changed: [ios-xe-mgmt-latest.cisco.com] => (item={'loopbackname': 'Loopback101', 'ip': '10.16.100.101', 'mask': '32'})
changed: [ios-xe-mgmt-latest.cisco.com] => (item={'loopbackname': 'Loopback102', 'ip': '10.16.100.102', 'mask': '32'})
changed: [ios-xe-mgmt-latest.cisco.com] => (item={'loopbackname': 'Loopback103', 'ip': '10.16.100.103', 'mask': '32'})

TASK [Make interface down] ************************************************************************************************************************************************
changed: [ios-xe-mgmt-latest.cisco.com] => (item={'loopbackname': 'Loopback100', 'ip': '10.16.100.100', 'mask': '32'})
changed: [ios-xe-mgmt-latest.cisco.com] => (item={'loopbackname': 'Loopback101', 'ip': '10.16.100.101', 'mask': '32'})
changed: [ios-xe-mgmt-latest.cisco.com] => (item={'loopbackname': 'Loopback102', 'ip': '10.16.100.102', 'mask': '32'})
changed: [ios-xe-mgmt-latest.cisco.com] => (item={'loopbackname': 'Loopback103', 'ip': '10.16.100.103', 'mask': '32'})

PLAY RECAP ****************************************************************************************************************************************************************
ios-xe-mgmt-latest.cisco.com : ok=3    changed=3    unreachable=0    failed=0    skipped=0    rescued=0    ignored=0  
```

And let's verify on the device itself. You will see indeed our interfaces got added successfully and the correct IP addresses were configured on them.

```bash
csr1000v-1#show ip interface brief
Interface              IP-Address      OK? Method Status                Protocol
GigabitEthernet1       10.10.20.48     YES NVRAM  up                    up
GigabitEthernet2       unassigned      YES NVRAM  administratively down down
GigabitEthernet3       unassigned      YES NVRAM  administratively down down
Loopback100            10.16.100.100   YES manual administratively down down
Loopback101            10.16.100.101   YES manual administratively down down
Loopback102            10.16.100.102   YES manual administratively down down
Loopback103            10.16.100.103   YES manual administratively down down
```

###### B) Remove multiple interfaces
Next, let's quickly remove these interface as well. Pretty straightforward, we loop again over the interfaces defined in the variables file and we delete them (cfr state: absent).

```yaml
---
- name: Delete interfaces
  hosts: iosxe
  gather_facts: no
  vars_files:
    - vars/loopbacks.yaml
  
  tasks:
  - name: Delete loopback interfaces
    ios_interface:
      name: "{{ item.loopbackname }}"
      state: absent
    with_items: "{{ loopbacks }}"
```
Let's execute the playbook:

```bash
wauterw@WAUTERW-M-65P7 ios_modules % ansible-playbook -i hosts loopback_remove_multiple.yaml

PLAY [Delete interfaces] **************************************************************************************************************************************************

TASK [Delete loopback interfaces] *****************************************************************************************************************************************
changed: [ios-xe-mgmt-latest.cisco.com] => (item={'loopbackname': 'Loopback100', 'ip': '10.16.100.100', 'mask': '32'})
changed: [ios-xe-mgmt-latest.cisco.com] => (item={'loopbackname': 'Loopback101', 'ip': '10.16.100.101', 'mask': '32'})
changed: [ios-xe-mgmt-latest.cisco.com] => (item={'loopbackname': 'Loopback102', 'ip': '10.16.100.102', 'mask': '32'})
changed: [ios-xe-mgmt-latest.cisco.com] => (item={'loopbackname': 'Loopback103', 'ip': '10.16.100.103', 'mask': '32'})

PLAY RECAP ****************************************************************************************************************************************************************
ios-xe-mgmt-latest.cisco.com : ok=1    changed=1    unreachable=0    failed=0    skipped=0    rescued=0    ignored=0 
```
And let's verify that the interfaces are really removed.

```bash
csr1000v-1#show ip interface brief
Interface              IP-Address      OK? Method Status                Protocol
GigabitEthernet1       10.10.20.48     YES NVRAM  up                    up
GigabitEthernet2       unassigned      YES NVRAM  administratively down down
GigabitEthernet3       unassigned      YES NVRAM  administratively down down
```

### Change interface description
Next, let's change the description of an interface. I'm merely showing this example to get familiar with the `ios_config` module for which you can find the information [here](https://docs.ansible.com/ansible/latest/modules/ios_config_module.html#ios-config-module).

In below script, we will change the description of an existing interface.

```yaml
---
- name: Change interface description
  hosts: iosxe
  gather_facts: no

  tasks:
    - name: configure interface settings
      ios_config:
        lines:
          - description Changed through Ansible
          - ip address 172.31.1.1 255.255.255.0
        parents: interface GigabitEthernet3
```
Let's execute the playbook:

```bash
wauterw@WAUTERW-M-65P7 ios_modules % ansible-playbook -i hosts change_description.yaml

PLAY [Change interface description] ***************************************************************************************************************************************

TASK [configure interface settings] ***************************************************************************************************************************************
changed: [ios-xe-mgmt-latest.cisco.com]

PLAY RECAP ****************************************************************************************************************************************************************
ios-xe-mgmt-latest.cisco.com : ok=1    changed=1    unreachable=0    failed=0    skipped=0    rescued=0    ignored=0   
```
And indeed, you will see the interface got changed.

```bash
csr1000v-1#show inter descr
Interface                      Status         Protocol Description
Gi1                            up             up       MANAGEMENT INTERFACE - DON'T TOUCH ME
Gi2                            admin down     down     Network Interface
Gi3                            admin down     down     Changed through Ansible

```

### Change multiple interface descriptions

Lastly, let's change multiple interfaces simultaneously. We first create the interfaces by defining them in a variable file (see use case above where this has been covered already) and next, we will change the description. To achieve that, we define a list of interface commands under the `with_items` section and we use that information in the `parents` attribute.

```yaml
---
- name: Configure loopback using ios_interface
  hosts: iosxe
  gather_facts: no
  vars_files:
    - vars/loopbacks.yaml

  tasks:
    - name: Creating loopback 
      ios_interface:
        name: "{{ item.loopbackname }}"
        enabled: True
        description: Loopback interface created with Ansible
      with_items: "{{ loopbacks }}"

    - name: Change description
      ios_config:
        lines:
          - description Loopback interface changed through Ansible
        parents: "{{item}}"
      with_items:
        - interface Loopback100
        - interface Loopback101
        - interface Loopback102
        - interface Loopback103
```
Execute the playbook:

```bash
wauterw@WAUTERW-M-65P7 ios_modules % ansible-playbook -i hosts change_description_multipleinterfaces.yaml

```
And look on the device through SSH. The interface description got changed indeed.

```
csr1000v-1#show interface description
Interface                      Status         Protocol Description
Gi1                            up             up       MANAGEMENT INTERFACE - DON'T TOUCH ME
Gi2                            admin down     down     Network Interface
Gi3                            admin down     down     Network Interface
Lo100                          up             up       Loopback interface changed through Ansible
Lo101                          up             up       Loopback interface changed through Ansible
Lo102                          up             up       Loopback interface changed through Ansible
Lo103                          up             up       Loopback interface changed through Ansible
```

### Change multiple interface descriptions -- variant

The above example listed all the interfaces individually under the `with_items` section. That was a bit clumsy. So I created also a small variant which handles this a bit more elegant. I won't go over the script as with the information you learned above it should be pretty straigthforward.

```yaml
---
- name: Configure loopback using ios_interface
  hosts: iosxe
  gather_facts: no
  vars_files:
    - vars/loopbacks.yaml
    - vars/loopbacks_interfaces.yaml

  tasks:
    - name: Creating loopback 
      ios_interface:
        name: "{{ item.loopbackname }}"
        enabled: True
        description: Loopback interface created with Ansible
      with_items: "{{ loopbacks }}"

    - name: Change description
      ios_config:
        lines:
          - description Loopback interface changed through Ansible
        parents: "{{item.loopbackinterface}}"
      with_items: "{{ loopbacks_interfaces }}"
```

Hope you enjoyed working with Ansible. The scripts I have used in this post, can be found on my Github [repo](https://github.com/wiwa1978/blog-hugo-netlify-code/tree/master/Ansible_IOSXE/ios_modules).
