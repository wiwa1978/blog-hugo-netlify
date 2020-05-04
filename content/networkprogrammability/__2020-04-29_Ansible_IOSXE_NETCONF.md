---
title: Ansible and IOSXE - NETCONF
date: 2020-04-29T10:32:50+01:00
draft: true
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

In this blog post, we will use Ansible to interact with our IOS XE devices. Ansible has a very extensive set of Network Modules for various devices and vendors. To interact with IOS XE devices, one could use the IOS module. You can find the documentation [here](https://docs.ansible.com/ansible/latest/modules/list_of_network_modules.html#ios).

We will go through a small number of use cases to see how this module works.

> For all the examples, we will use a Cisco sandbox environment delivered by [Cisco Devnet](https://developer.cisco.com). Go check out Devnet, really brilliant. To get a list of all sandboxes, check out [this](https://devnetsandbox.cisco.com/RM/Topology) link. For this blog post, we will use the IOS XE sandbox.

### Show version and show interfaces