---
title: Ansible and IOSXE - RESTCONF
date: 2020-05-07T12:32:50+01:00
draft: false
categories:
  - Network Programming
  - Programming
  - All
tags:
  - Ansible
  - IOSXE
  - RESTCONF
---
### Introduction


### General preparation

```ini
[all:vars]
ansible_python_interpreter="/usr/bin/python3"
ansible_network_os=default

[iosxe]
ios-xe-mgmt.cisco.com ansible_port=8181
```


group_vars

```yaml
ansible_user: 'developer'
ansible_password: 'C1sco12345'
ansible_connection: "httpapi"
ansible_network_os: "restconf"
ansible_httpapi_use_ssl: true
ansible_httpapi_port: 9443
ansible_httpapi_validate_certs: false
ansible_httpapi_restconf_root: "/restconf"
```

### Ansible RESTCONF: retrieve IETF interfaces

```yaml
---
- name: Restconf - GET
  hosts: iosxe
  connection: local

  tasks:
  - name: Get interfaces from IETF YANG model
    restconf_get:
      content: config
      output: json
      path: /data/ietf-interfaces:interfaces/interface=GigabitEthernet1
    register: output

  - name: display all
    debug:
      var: output
```
In case you want to retrieve all interface, use the followinf path `path: /data/ietf-interfaces:interfaces`

```bash
wauterw@WAUTERW-M-65P7 restconf % ansible-playbook -i hosts restconf_get.yml
➜  Method1 git:(master) ✗ ansible-playbook -i hosts get_interfaces_restconf.yml 

PLAY [Restconf - GET] **********************************************************************************************************************************************************

TASK [Gathering Facts] *********************************************************************************************************************************************************
ok: [ios-xe-mgmt.cisco.com]

TASK [Get interfaces from IETF YANG model] *************************************************************************************************************************************
ok: [ios-xe-mgmt.cisco.com]

TASK [display all] *************************************************************************************************************************************************************
ok: [ios-xe-mgmt.cisco.com] => {
    "output": {
        "changed": false,
        "failed": false,
        "response": {
            "ietf-interfaces:interface": {
                "description": "CONF WITH MAT BY MATI",
                "enabled": true,
                "ietf-ip:ipv4": {
                    "address": [
                        {
                            "ip": "10.10.20.48",
                            "netmask": "255.255.255.0"
                        }
                    ]
                },
                "ietf-ip:ipv6": {},
                "name": "GigabitEthernet1",
                "type": "iana-if-type:ethernetCsmacd"
            }
        }
    }
}

PLAY RECAP *********************************************************************************************************************************************************************
ios-xe-mgmt.cisco.com      : ok=3    changed=0    unreachable=0    failed=0    skipped=0    rescued=0    ignored=0   
```

### Ansible RESTCONF: create IETF interfaces

variables file
```json
{
    "interface": [
       {
          "name": "Loopback200010",
          "description": "Loopback via Ansible Restconf",
          "type": "iana-if-type:softwareLoopback",
          "enabled": true,
          "ietf-ip:ipv4": {
             "address": [
                {
                   "ip": "10.0.12.22",
                   "netmask": "255.255.255.255"
                }
             ]
          }
       },
       {
          "name": "Loopback200011",
          "description": "Loopback via Ansible Restconf",
          "type": "iana-if-type:softwareLoopback",
          "enabled": true,
          "ietf-ip:ipv4": {
             "address": [
                {
                   "ip": "10.0.12.23",
                   "netmask": "255.255.255.255"
                }
             ]
          }
       },
       {
          "name": "Loopback200012",
          "description": "Loopback via Ansible Restconf",
          "type": "iana-if-type:softwareLoopback",
          "enabled": true,
          "ietf-ip:ipv4": {
             "address": [
                {
                   "ip": "10.0.12.24",
                   "netmask": "255.255.255.255"
                }
             ]
          }
       }
    ]
 }
```

```yaml
---
- name: Restconf - Config
  hosts: iosxe

  tasks:
  - name:  Create interfaces with IETF YANG model
    restconf_config:
      method: post
      format: json
      path: /data/ietf-interfaces:interfaces
      content: "{{ lookup('file','./loopback.json') | string }}"
    register: output
    ignore_errors: true

  - name: display all
    debug:
      var: output
```


### Ansible RESTCONF: create IETF interfaces -- variant

Add the following to the group_vars file.
```yaml

loopback_interfaces: 
    interface:
        - name: "Loopback200001"
          description: "Loopback via Ansible Restconf"
          type: "iana-if-type:softwareLoopback"
          enabled: true
          ietf-ip:ipv4:
            address:
              - ip: "10.0.12.12"
                netmask: "255.255.255.255"
        - name: "Loopback200002"
          description: "Loopback via Ansible Restconf"
          type: "iana-if-type:softwareLoopback"
          enabled: true
          ietf-ip:ipv4:
            address:
              - ip: "10.0.12.13"
                netmask: "255.255.255.255"
        - name: "Loopback200003"
          description: "Loopback via Ansible Restconf"
          type: "iana-if-type:softwareLoopback"
          enabled: true
          ietf-ip:ipv4:
            address:
              - ip: "10.0.12.14"
                netmask: "255.255.255.255"
```

```yaml
---
- name: Restconf - Config
  hosts: iosxe

  tasks:
  - name:  Create interfaces with IETF YANG model
    restconf_config:
      path: /data/ietf-interfaces:interfaces
      content: "{{ loopback_interfaces | to_json }}"
```

### Ansible RESTCONF: delete IETF interfaces 
```yaml
---
- name: Restconf - Config
  hosts: iosxe

  tasks:
  - name:  Delete interfaces with IETF YANG model
    restconf_config:
      method: delete
      path: /data/ietf-interfaces:interfaces/interface={{ item['name'] }}
    with_items: "{{ loopback_interfaces['interface'] }}"
```