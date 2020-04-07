---
title: Create Cisco ACI network with Ansible
date: 2020-01-16T14:39:50+01:00
draft: True
categories:
  - DevOps
  - Infrastructure As Code
tags:
  - Private Cloud
  - Ansible
  - ACI
---

### Introduction


### Ansible code

```yaml
- name: Limited ACI configuration
  hosts: apic
  connection: local
  gather_facts: False
  vars_files:
    - vars/variables_aci.yml
  
  tasks:
    - name: Task 01 - Ensure Tenant Exists
      aci_tenant:
        host: "{{ inventory_hostname }}"
        username: "{{ username }}"
        password: "{{ password }}"
        state: "present"
        validate_certs: False
        tenant: "{{ tenant }}"
        descr: "Tenant Created Using Ansible"
      tags: tenant, vrf, bd, filter, contract, app, epg

    - name: Task 02 - Ensure VRF Exists
      aci_vrf:
        host: "{{ inventory_hostname }}"
        username: "{{ username }}"
        password: "{{ password }}"
        state: "present"
        validate_certs: False
        tenant: "{{ tenant }}"
        vrf: "{{ vrf }}"
        descr: "VRF Created Using Ansible"
      tags: vrf, bd

    - name: Task 03 - Ensure Bridge Domains and Exist
      aci_bd:
        host: "{{ inventory_hostname }}"
        username: "{{ username }}"
        password: "{{ password }}"
        state: "present"
        validate_certs: False
        tenant: "{{ tenant }}"
        bd: "{{ item.bd | default('prod_bd') }}"
        vrf: "{{ vrf }}"
      with_items: "{{ bridge_domains }}"
      tags: bd
    
    - name: Task 03A - Attach L3Out to BD
      aci_bd_to_l3out:
        host: "{{ inventory_hostname }}"
        username: "{{ username }}"
        password: "{{ password }}"
        state: "present"
        validate_certs: False
        bd: "{{ item.bd }}"
        tenant: "{{ tenant }}"
        l3out: "labinfra-l3out-ro"
      with_items: "{{ bridge_domains }}"
      tags: bd

    - name: Task 04 - Ensure Bridge Domains Have Subnets
      aci_bd_subnet:
        host: "{{ inventory_hostname }}"
        username: "{{ username }}"
        password: "{{ password }}"
        validate_certs: False
        state: "present"
        tenant: "{{ tenant }}"
        bd: "{{ item.bd }}"
        gateway: "{{ item.gateway }}"
        mask: "{{ item.mask }}"
        scope: ["public", "shared"]
      with_items: "{{ bridge_domains }}"
```
Variables

```yaml
---
tenant: "Tenant_Demo"
vrf: "VRF_Demo"
bridge_domains:
  - bd: "BD_Demo"
    gateway: "10.16.100.1"
    mask: "24"
    scope: "shared"
```

### Deploying the infrastructure

```bash
WAUTERW-M-65P7:ACI_Ansible wauterw$ ansible-playbook -i hosts_aci.yml playbook_aci_create.yml 

PLAY [Limited ACI configuration] *************************************************************************

TASK [Task 01 - Ensure Tenant Exists] ********************************************************************
changed: [10.48.109.10]

TASK [Task 02 - Ensure VRF Exists] ***********************************************************************
changed: [10.48.109.10]

TASK [Task 03 - Ensure Bridge Domains and Exist] *********************************************************
changed: [10.48.109.10] => (item={'bd': 'BD_Demo', 'gateway': '10.16.100.1', 'mask': '24', 'scope': 'shared'})

TASK [Task 03A - Attach L3Out to BD] *********************************************************************
changed: [10.48.109.10] => (item={'bd': 'BD_Demo', 'gateway': '10.16.100.1', 'mask': '24', 'scope': 'shared'})

TASK [Task 04 - Ensure Bridge Domains Have Subnets] ******************************************************
changed: [10.48.109.10] => (item={'bd': 'BD_Demo', 'gateway': '10.16.100.1', 'mask': '24', 'scope': 'shared'})

PLAY RECAP ***********************************************************************************************
10.48.109.10               : ok=5    changed=5    unreachable=0    failed=0    skipped=0    rescued=0    ignored=0   
```

### Verify the infrastructure
![ACIAnsible](/images/2020-01-16.1.png)

![ACIAnsible](/images/2020-01-16.2.png)

### Destroy the infrastructure

```yaml
- name: Cleanup - Removing Tenants
  hosts: apic
  connection: local
  gather_facts: False
  vars:
    tenants:
      - "TenantDemo"

  tasks:
    - name: Delete Tenants
      aci_tenant:
        host: "{{ ansible_host }}"
        username: "{{ username }}"
        password: "{{ password }}"
        state: "absent"
        validate_certs: False
        tenant: "{{ item }}"
      with_items: "{{ tenants }}"
```

```bash
WAUTERW-M-65P7:ACI_Ansible wauterw$ ansible-playbook -i hosts_aci.yml playbook_aci_delete.yml 

PLAY [Cleanup - Removing Tenants] ************************************************************************

TASK [Delete Tenants] ************************************************************************************
ok: [10.48.109.10] => (item=TenantDemo)

PLAY RECAP ***********************************************************************************************
10.48.109.10               : ok=1    changed=0    unreachable=0    failed=0    skipped=0    rescued=0    i
gnored=0   
```

[Github](https://github.com/wiwa1978/blog-hugo-netlify-code/tree/master/ACI_Ansible)
[![Private](/images/whitebox.png)](https://github.com/wiwa1978/blog-hugo-netlify-code-private/tree/master/ACI_Ansible_Complete)

