---
title: Ansible and IOSXE - RESTCONF
date: 2020-05-02T10:32:50+01:00
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


### restconf_get

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

```yaml
---
- name: Restconf - GET
  hosts: iosxe
  connection: local

  tasks:
  - name: Get Config
    restconf_get:
      content: config
      output: json
      path: /data/ietf-interfaces:interfaces
    register: output

  - name: display all
    debug:
      var: output
```

```bash
wauterw@WAUTERW-M-65P7 restconf % ansible-playbook -i hosts restconf_get.yml

PLAY [Restconf - GET] *********************************************************************************************************************************************************************

TASK [Gathering Facts] ********************************************************************************************************************************************************************
ok: [ios-xe-mgmt-latest.cisco.com]

TASK [Get Config] *************************************************************************************************************************************************************************
ok: [ios-xe-mgmt-latest.cisco.com]

TASK [display all] ************************************************************************************************************************************************************************
ok: [ios-xe-mgmt-latest.cisco.com] => {
    "output": {
        "changed": false,
        "failed": false,
        "response": {
            "ietf-interfaces:interfaces": {
                "interface": [
                    {
                        "description": "MANAGEMENT INTERFACE - DON'T TOUCH ME",
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
                    },
                    {
                        "description": "Network Interface",
                        "enabled": false,
                        "ietf-ip:ipv4": {},
                        "ietf-ip:ipv6": {},
                        "name": "GigabitEthernet2",
                        "type": "iana-if-type:ethernetCsmacd"
                    },
                    {
                        "description": "Network Interface",
                        "enabled": false,
                        "ietf-ip:ipv4": {},
                        "ietf-ip:ipv6": {},
                        "name": "GigabitEthernet3",
                        "type": "iana-if-type:ethernetCsmacd"
                    }
                ]
            }
        }
    }
}

PLAY RECAP ********************************************************************************************************************************************************************************
ios-xe-mgmt-latest.cisco.com : ok=3    changed=0    unreachable=0    failed=0    skipped=0    rescued=0    ignored=0   

```

If we only want to look at one particular interface, change the following line:

```yaml
path: /data/ietf-interfaces:interfaces
```
to
```yaml
path: /data/ietf-interfaces:interfaces/interface=GigabitEthernet1
```
And the output will show this:

```bash
wauterw@WAUTERW-M-65P7 restconf % ansible-playbook -i hosts restconf_get.yml

PLAY [Restconf - GET] *********************************************************************************************************************************************************************

TASK [Gathering Facts] ********************************************************************************************************************************************************************
ok: [ios-xe-mgmt-latest.cisco.com]

TASK [Get Config] *************************************************************************************************************************************************************************
ok: [ios-xe-mgmt-latest.cisco.com]

TASK [display all] ************************************************************************************************************************************************************************
ok: [ios-xe-mgmt-latest.cisco.com] => {
    "output": {
        "changed": false,
        "failed": false,
        "response": {
            "ietf-interfaces:interface": {
                "description": "MANAGEMENT INTERFACE - DON'T TOUCH ME",
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

PLAY RECAP ********************************************************************************************************************************************************************************
ios-xe-mgmt-latest.cisco.com : ok=3    changed=0    unreachable=0    failed=0    skipped=0    rescued=0    ignored=0   

```
