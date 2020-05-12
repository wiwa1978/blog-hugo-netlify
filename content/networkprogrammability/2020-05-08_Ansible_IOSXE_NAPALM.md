---
title: Ansible and IOSXE - NAPALM
date: 2020-05-08T10:32:50+01:00
draft: false
categories:
  - Network Programming
  - Programming
  - All
tags:
  - Ansible
  - IOSXE
  - NAPALM
---
### Introduction

In [this](https://blog.wimwauters.com/networkprogrammability/2020-04-29_ansible_iosxe_iosmodules/) blog post, we used Ansible to interact with our IOS XE devices through a module called `ios-modules`. We followed up with using the `netconf` module for Ansible, check [this](https://blog.wimwauters.com/networkprogrammability/2020-04-29_ansible_iosxe_netconf/) post. In this post, we will use Ansible with NAPALM. Have a look at [this](https://github.com/napalm-automation/napalm-ansible) Github project.

> For all the examples, we will use a Cisco sandbox environment delivered by [Cisco Devnet](https://developer.cisco.com). To get a list of all sandboxes, check out [this](https://devnetsandbox.cisco.com/RM/Topology) link. For this blog post, we will use the IOS XE sandbox.

### Installation

Installation is pretty simple. 

```bash
wauterw@WAUTERW-M-65P7 napalm % pip3 install napalm-ansible
wauterw@WAUTERW-M-65P7 napalm % pip3 install napalm
```

I got some issues after installing the napalm-ansible, as my Python installation could not find the proper library in the site-packages. This was mainly because I run multiple versions of Python on my laptop and it kind of messed up a little bit. Therefore, it's import you know the sites-packages directory that Napalm and Napalm-Ansible got installed. 

It's important to follow the `Configuring Ansible` instructions on the napalm-ansible Github repo. It mentions there that you need to create an ansible.cfg file and specify the `library` and `action_plugins` variables. Here's what my file looks like.

```yaml
[defaults]
library = /usr/local/lib/python3.7/site-packages/napalm_ansible/modules
action_plugins = /usr/local/lib/python3.7/site-packages/napalm_ansible/plugins/action
hostfile = hosts
host_key_checking = False
deprecation_warnings=False
stdout_callback = skippy

[persistent_connection]
connect_timeout = 100
command_timeout = 80
```

### Retrieve information from IOSXE device

Have a look at the [information](https://napalm.readthedocs.io/en/latest/integrations/ansible/modules/napalm_get_facts/) for `napalm_get_facts`. It allows you to gather facts from a network device through NAPALM. Here's an example:

```yaml
---
- name: Retrieve interfaces
  hosts: iosxe
  gather_facts: no

  tasks:
    - name: get facts from device
      napalm_get_facts:
        username: "{{ ansible_user }}"
        password: "{{ ansible_ssh_pass }}"
        filter: facts
        optional_args:
          port: 8181
      register: result

    - name: print data
      debug: var=result
```
When we run the above script, we see the following output. 

```bash
wauterw@WAUTERW-M-65P7 napalm % ansible-playbook -i hosts get_facts.yml      

PLAY [Retrieve interfaces] *********************************************************************************************

TASK [get facts from device] *******************************************************************************************
ok: [ios-xe-mgmt-latest.cisco.com]

TASK [print data] ******************************************************************************************************
ok: [ios-xe-mgmt-latest.cisco.com] => {
    "result": {
        "ansible_facts": {
            "napalm_facts": {
                "fqdn": "csr1000v-1.abc.inc",
                "hostname": "csr1000v-1",
                "interface_list": [
                    "GigabitEthernet1",
                    "GigabitEthernet2",
                    "GigabitEthernet3",
                    "Loopback21",
                    "Loopback1001"
                ],
                "model": "CSR1000V",
                "os_version": "Virtual XE Software (X86_64_LINUX_IOSD-UNIVERSALK9-M), Version 16.11.1a, RELEASE SOFTWARE (fc1)",
                "serial_number": "9EFJDZV5LZF",
                "uptime": 59520,
                "vendor": "Cisco"
            },
            "napalm_fqdn": "csr1000v-1.abc.inc",
            "napalm_hostname": "csr1000v-1",
            "napalm_interface_list": [
                "GigabitEthernet1",
                "GigabitEthernet2",
                "GigabitEthernet3",
                "Loopback21",
                "Loopback1001"
            ],
            "napalm_model": "CSR1000V",
            "napalm_os_version": "Virtual XE Software (X86_64_LINUX_IOSD-UNIVERSALK9-M), Version 16.11.1a, RELEASE SOFTWARE (fc1)",
            "napalm_serial_number": "9EFJDZV5LZF",
            "napalm_uptime": 59520,
            "napalm_vendor": "Cisco"
        },
        "changed": false,
        "failed": false
    }
}

PLAY RECAP *************************************************************************************************************
ios-xe-mgmt-latest.cisco.com : ok=2    changed=0    unreachable=0    failed=0    skipped=0    rescued=0    ignored=0   

```
We get a nice overview of the basic information of our IOS device but this time through the NAPALM module. In case you want to retrieve a list of interfaces, change the following line:
```
filter: facts
```
to
```
filter: facts, interfaces
```
As such, you will receive a list of the configured interfaces as well.

Other paramets you could use for filter are: mac_address_table, arp_table, bgp_config, bgp_neighbors, bgp_neighbors_detail, interfaces_counters, interfaces_ip, lldp_neighbors, lldp_neighbors_detail, network_instances, ntp_servers, ntp_stats, users...and some more.

### Execute commands

Have a look at the [information](https://napalm.readthedocs.io/en/latest/integrations/ansible/modules/napalm_cli/index.html) for `napalm_cli` module. It allows you to execute commands on your IOS XE device. Have a look at the below script which executes two simple commands.

```yaml
---
- name: Execute commands
  hosts: iosxe
  gather_facts: no

  tasks:
    - name: Execute commands
      napalm_cli:
        hostname: "{{ inventory_hostname }}"
        username: "{{ ansible_user }}"
        password: "{{ ansible_ssh_pass }}"
        dev_os: "ios"
        optional_args:
          port: 8181
        args:
          commands:
            - show version
            - show ip interface brief
      register: result

    - name: print data
      debug: var=result
```
And you will get the following output. Note that I truncated the output quite a bit.

```bash
wauterw@WAUTERW-M-65P7 napalm % ansible-playbook -i hosts show_command.yml

PLAY [Configure loopback using ios_interface] **************************************************************************

TASK [get facts from device] *******************************************************************************************
ok: [ios-xe-mgmt-latest.cisco.com]

TASK [print data] ******************************************************************************************************
ok: [ios-xe-mgmt-latest.cisco.com] => {
    "result": {
        "changed": false,
        "cli_results": {
            "show ip interface brief": "Interface              IP-Address      OK? Method Status                Protocol\nGigabitEthernet1       10.10.20.48     YES NVRAM  up                    up      \nGigabitEthernet2       172.16.31.202   YES other  administratively down down    \nGigabitEthernet3       unassigned      YES NVRAM  administratively down down    \nLoopback1001           10.111.100.3    YES other  up                    up",
            
            "show version": "Cisco IOS XE Software,
            ***Truncated***"
        },
        "failed": false
    }
}

PLAY RECAP *************************************************************************************************************
ios-xe-mgmt-latest.cisco.com : ok=2    changed=0    unreachable=0    failed=0    skipped=0    rescued=0    ignored=0   
```

### Create loopback interfaces

In this section, we will focus on adding some Loopback interfaces. We will use NAPALM's merge function for that. Again, a similar example was discussed in an earlier [post](http://localhost:1313/networkprogrammability/2020-04-07_napalm_introduction_part2/) although this was done through Python. Here we will use the ansbible-napalm module.

Let's create a loopbacks.txt file. This file contains the configuration for the interfaces:

```
interface Loopback100
description LOOPBACK 100
 ip address 4.4.4.101 255.255.255.255
!
interface Loopback200
description LOOPBACK 100
 ip address 1.1.1.20 255.255.255.255
end
```
Next, let's have a look at the Ansible playbook. Again, Napalm-Ansible module makes it fairly straightforward for us.

```yaml
---
- name: Configure loopback using ios_interface
  hosts: iosxe
  gather_facts: no

  tasks:
    - name: create loopback interface
      napalm_install_config:
        hostname: "{{ inventory_hostname }}"
        username: "{{ ansible_user }}"
        password: "{{ ansible_ssh_pass }}"
        dev_os: "ios"
        optional_args:
          port: 8181
        config_file: 'loopbacks.txt'
        commit_changes: true
        replace_config: false  
```
The important part is that we point to our loopbacks.txt file we created earlier. The `commit_changes` will allow us to merge/replace the configuration. If set to true, we will merge the loopbacks information with the existing configuration. The `replace_config` is set to false, which means we will not replace the entire configuration. The `get_diffs` will indicate that we want a diff to be generated between the existing and the new configuration. The `diff_file` will let you select the path where the diff file will be stored.

Before we continue, let's see the current overview of configured interfaces on our device.

```bash
csr1000v-1#show ip interface brief
Interface              IP-Address      OK? Method Status                Protocol
GigabitEthernet1       10.10.20.48     YES NVRAM  up                    up
GigabitEthernet2       unassigned      YES NVRAM  administratively down down
GigabitEthernet3       unassigned      YES NVRAM  administratively down down
```

When we execute the ansible playbook, we get the following:

```bash
wauterw@WAUTERW-M-65P7 napalm % ansible-playbook -i hosts create_loopback.yml

PLAY [Configure loopback using ios_interface] **************************************************************************

TASK [create loopback interface] ***************************************************************************************
changed: [ios-xe-mgmt-latest.cisco.com]

PLAY RECAP *************************************************************************************************************
ios-xe-mgmt-latest.cisco.com : ok=1    changed=1    unreachable=0    failed=0    skipped=0    rescued=0    ignored=0   
```
When we look in our device, we will see the following:

```bash
csr1000v-1#show ip interface brief
Interface              IP-Address      OK? Method Status                Protocol
GigabitEthernet1       10.10.20.48     YES NVRAM  up                    up
GigabitEthernet2       unassigned      YES NVRAM  administratively down down
GigabitEthernet3       unassigned      YES NVRAM  administratively down down
Loopback100            4.4.4.101       YES TFTP   up                    up
Loopback200            1.1.1.20        YES TFTP   up                    up
```
And also, you'll notice a diff file has been created (in the current directory) with the following contents:

```
+interface Loopback100
+description LOOPBACK 100
+ ip address 4.4.4.101 255.255.255.255
+interface Loopback200
+description LOOPBACK 100
+ ip address 1.1.1.20 255.255.255.255
```

### NAPALM Validation

We discussed NAPALM validation already in [this](http://localhost:1313/networkprogrammability/2020-04-07_napalm_introduction_part2/) post. The idea is to check or validate the configuration on our device. We learned that we need to create a validation file which contains the proper values. In our case, we would want to validate whether the device is running a particular software version and we would like to validate the IP address of the loopback interface. Let's do this first, the validation file looks as follows:

```yaml
---
- get_facts:
    os_version: 16.11.1a
    vendor: Cisco

- get_interfaces_ip:
    Loopback100:
      ipv4:
        4.4.4.101:
          prefix_length: 32
```
In this post, we would obviously like to use Napalm-ansible module. The playbook is in fact very straightforward:

```yaml
---
- name: Validate command
  hosts: iosxe
  gather_facts: no

  tasks:
    - name: NaPalm Validation
      napalm_validate:
        hostname: "{{ inventory_hostname }}"
        username: "{{ ansible_user }}"
        password: "{{ ansible_ssh_pass }}"
        dev_os: "ios"
        optional_args:
          port: 8181
        validation_file: "{{ validate_file }}"
```
You'll notice that we reference the `{{ validate_file }}` variable so do not forget to add this variable in the group_vars/iosxe.yaml file.

```yaml
host_key_checking : False
ansible_user: ***
ansible_ssh_pass: ***
ansible_port: 8181
ansible_python_interpreter: /usr/local/bin/python3
ansible_network_os: ios
ansible_connection: network_cli
validate_file: "validation.yml"
```
Let's execute this playbook next.

```bash
wauterw@WAUTERW-M-65P7 napalm % ansible-playbook -i hosts validate_command.yml

PLAY [Validate command] ************************************************************************************************

TASK [NaPalm Validation] ***********************************************************************************************
ok: [ios-xe-mgmt-latest.cisco.com]

PLAY RECAP *************************************************************************************************************
ios-xe-mgmt-latest.cisco.com : ok=1    changed=0    unreachable=0    failed=0    skipped=0    rescued=0    ignored=0   
 
```
You will notice the validation was successful. Let's change the value of the IP address inside the validation.yml file. We will change it from 4.4.4.101 to 4.4.4.102 so we expect the validation to fail. Let's have a look:

```bash
wauterw@WAUTERW-M-65P7 napalm % ansible-playbook -i hosts validate_command.yml

PLAY [Validate command] ************************************************************************************************

TASK [NaPalm Validation] ***********************************************************************************************
fatal: [ios-xe-mgmt-latest.cisco.com]: FAILED! => {"changed": false, "compliance_report": {"complies": false, "get_facts": {"complies": true, "extra": [], "missing": [], "present": {"os_version": {"complies": true, "nested": false}, "vendor": {"complies": true, "nested": false}}}, "get_interfaces_ip": {"complies": false, "extra": [], "missing": [], "present": {"Loopback100": {"complies": false, "diff": {"complies": false, "extra": [], "missing": [], "present": {"ipv4": {"complies": false, "diff": {"complies": false, "extra": [], "missing": ["4.4.4.102"], "present": {}}, "nested": true}}}, "nested": true}}}, "skipped": []}, "msg": "Device does not comply with policy"}

PLAY RECAP *************************************************************************************************************
ios-xe-mgmt-latest.cisco.com : ok=0    changed=0    unreachable=0    failed=1    skipped=0    rescued=0    ignored=0   

```
In the above output, you will notice that the compliance status is false as expected.

Check out my Github [repo](https://github.com/wiwa1978/blog-hugo-netlify-code/tree/master/Ansible_IOSXE/napalm) for these examples. Hope to see you back soon.