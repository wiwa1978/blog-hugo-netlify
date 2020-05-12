---
title: Ansible and IOSXE - NETCONF
date: 2020-05-05T12:32:50+01:00
draft: false
categories:
  - Network Programming
  - Programming
  - All
tags:
  - Ansible
  - IOSXE
  - NETCONF
---
### Introduction

In [this](https://blog.wimwauters.com/networkprogrammability/2020-04-29_ansible_iosxe_iosmodules/) blog post, we used Ansible to interact with our IOS XE devices through a module called `ios-modules`. However, it's also possible to interact with our IOS XE devices through Ansible and Netconf, through the `netconf` module. Check the documentation [here](https://docs.ansible.com/ansible/latest/modules/list_of_network_modules.html#netconfsh).

> For all the examples, we will use a Cisco sandbox environment delivered by [Cisco Devnet](https://developer.cisco.com). To get a list of all sandboxes, check out [this](https://devnetsandbox.cisco.com/RM/Topology) link. For this blog post, we will use the IOS XE sandbox.

### Create interface

Similar to what we did in [this](https://blog.wimwauters.com/networkprogrammability/2020-04-29_ansible_iosxe_iosmodules/) post, we will add an interface to our IOSXE device but using the `netconf_config` [module](https://docs.ansible.com/ansible/latest/modules/netconf_config_module.html).

The key part in below snippet is the `content` part. This contains the YANG filter to create the interface on the IOS XE device. In case you wonder where this XML comes from, I would refer you to an earlier post on this topic, see [here](https://blog.wimwauters.com/networkprogrammability/2020-03-31_netconf_python_part2/). The XML we are using in below Ansible playbook is exactly the same as what you would use to configure an interface using plain netconf. 

```yaml
---
- name: Add interface to IOSXE
  hosts: iosxe
  connection: local
  gather_facts: False

  tasks:
    - set_fact:
        ansible_connection: local

    - name: Create a loopback Loopback100 with NETCONF
      netconf_config:
        host: "{{inventory_hostname}}"
        port: "{{netconf_port}}"
        username: "{{ansible_user}}"
        password: "{{ansible_password}}"
        hostkey_verify: False
        content: |
            <config>
              <interfaces xmlns="urn:ietf:params:xml:ns:yang:ietf-interfaces">
                <interface>
                  <name>Loopback1001</name>
                  <description>Pod Number 100</description>
                  <type xmlns:ianaift="urn:ietf:params:xml:ns:yang:iana-if-type">
                    ianaift:softwareLoopback
                  </type>
                  <enabled>true</enabled>
                  <ipv4 xmlns="urn:ietf:params:xml:ns:yang:ietf-ip">
                    <address>
                      <ip>10.111.100.3</ip>
                      <netmask>255.255.255.255</netmask>
                    </address>
                  </ipv4>
                </interface>
              </interfaces>
            </config>
```
Short post this time. Have a look at the Github [repo](https://github.com/wiwa1978/blog-hugo-netlify-code/tree/master/Ansible_IOSXE/netconf) to see the example.