---
title: Ansible Vault with Cisco ACI
date: 2021-01-23T14:39:50+01:00
draft: false
categories:
  - DevOps
  - Network Programming
  - All
tags:
  - Private Cloud
  - Ansible
  - ACI
---

### Introduction
In [this](https://blog.wimwauters.com/devops/2020-01-16-aci_ansible/) post we created a simple ACI network using Ansible. The issue is that our passwords are unencrypted in all of our files. Obviously something we'd like to avoid as much as possible. Luckily, Ansible comes with Ansible Vault to protect sensible information. We will explore Ansible Vault further in this post.

### Ansible playbook (not secure version)

Let's first recap our Ansible playbook. You could also simply re-read [this](https://blog.wimwauters.com/devops/2020-01-16-aci_ansible/) post but I thought it would be good to review everything in this post (it's been a while since that post).

I will use Cisco Devnet APIC sandbox for this playbook. The sandbox can be found [here](https://sandboxapicdc.cisco.com/). 

Our host file looks as follows:

```yaml
[apic]
sandboxapicdc.cisco.com

[apic:vars]
ansible_connection = local
ansible_python_interpreter = /usr/bin/python3
username = admin
password = ---
```
As you can see, the password is in the clear in this file. Note that I have put a placeholder here as I don't like to expose passwords (even though it's freely available when you log into the Cisco Devnet Sandbox).

Below is our playbook. As you can see, we merely create a tenant, a VRF, a BD, a subnet within the BD, an application profile and an EPG
```yaml
- name: Limited ACI configuration
  hosts: apic
  connection: local
  gather_facts: False
  vars_files:
    - vars/variables_aci.yml
  vars:
    aci_login: &aci_login
      hostname: '{{ inventory_hostname }}'
      username: '{{ username }}'
      password: '{{ password }}' 
      validate_certs: False     
  
  tasks:
    - name: Task 01 - Create Tenant
      aci_tenant:
        <<: *aci_login
        state: "present"
        tenant: "{{ tenant }}"
        descr: "Tenant Created Using Ansible"
      tags: tenant, vrf, bd, filter, contract, app, epg

    - name: Task 02 - Create VRF
      aci_vrf:
        <<: *aci_login
        tenant: "{{ tenant }}"
        vrf: "{{ vrf }}"
        descr: "VRF Created Using Ansible"
      tags: vrf, bd

    - name: Task 03 - Create BD
      aci_bd:
        <<: *aci_login
        state: "present"
        tenant: "{{ tenant }}"
        bd: "{{ item.bd | default('prod_bd') }}"
        vrf: "{{ vrf }}"
      with_items: "{{ bridge_domains }}"
      tags: bd
    
    - name: Task 04 - Create subnet in BD
      aci_bd_subnet:
        <<: *aci_login
        state: "present"
        tenant: "{{ tenant }}"
        bd: "{{ item.bd }}"
        gateway: "{{ item.gateway }}"
        mask: "{{ item.mask }}"
        scope: ["public", "shared"]
      with_items: "{{ bridge_domains }}"

    - name: Task 05 - Add a new Application Profile
      aci_ap:
        <<: *aci_login
        tenant: '{{ tenant }}' 
        ap: '{{ ap }}'
        state: present

    - name: Task 06 - Create EPG's and add to the Application Profile
      aci_epg:
        <<: *aci_login
        tenant: '{{ tenant }}' 
        ap: '{{ ap }}'
        epg: '{{ item.epg }}'
        bd: '{{ item.bd }}'  
        preferred_group: yes
        state: present
      with_items: "{{ epgs }}"
```
I then also have a small variable file that looks as follows:

```yaml
---
tenant: "Tenant_Ansible"
vrf: "VRF_Ansible"
bridge_domains:
  - bd: "BD_Ansible"
    gateway: "10.16.100.1"
    mask: "24"
    scope: "shared"
ap: "AP_Ansible"
epgs:
  - epg: "EPG_Ansible"
    bd: "BD_Ansible"
```

### Executing the playbook

To execute this playbook, just use the following command:

```bash
~ACI_Ansible/Vault master ?1 ❯ ansible-playbook -i hosts_aci.yml playbook_aci_create.yml

PLAY [Limited ACI configuration] ***********************************************************************************************************************************************************************

TASK [Task 01 - Create Tenant] *************************************************************************************************************************************************************************
changed: [sandboxapicdc.cisco.com]

TASK [Task 02 - Create VRF] ****************************************************************************************************************************************************************************
changed: [sandboxapicdc.cisco.com]

TASK [Task 03 - Create BD] *****************************************************************************************************************************************************************************
changed: [sandboxapicdc.cisco.com] => (item={'bd': 'BD_Ansible', 'gateway': '10.16.100.1', 'mask': '24', 'scope': 'shared'})

TASK [Task 04 - Create subnet in BD] *******************************************************************************************************************************************************************
changed: [sandboxapicdc.cisco.com] => (item={'bd': 'BD_Ansible', 'gateway': '10.16.100.1', 'mask': '24', 'scope': 'shared'})

TASK [Task 05 - Add a new Application Profile] *********************************************************************************************************************************************************
changed: [sandboxapicdc.cisco.com]

TASK [Task 06 - Create EPG's and add to the Application Profile] ***************************************************************************************************************************************
changed: [sandboxapicdc.cisco.com] => (item={'epg': 'EPG_Ansible', 'bd': 'BD_Ansible'})

PLAY RECAP *********************************************************************************************************************************************************************************************
sandboxapicdc.cisco.com    : ok=6    changed=6    unreachable=0    failed=0    skipped=0    rescued=0    ignored=0   
```

![ansiblevault](/images/2021-01-23.png)

### Ansible Vault basics
Vault is a mechanism that allows encrypted content to be incorporated transparently into Ansible workflows. A utility called ansible-vault secures confidential data by encrypting it on disk. 

In order to encrypt the variables file, use the following command:

```bash
~ACI_Ansible/Vault master ?1 ❯ ansible-vault encrypt vars/variables_aci.yml                                                                  14:54:06
New Vault password: 
Confirm New Vault password: 
Encryption successful
```
Our variables file will be changed to:
```
$ANSIBLE_VAULT;1.1;AES256
38653237343337383762316165303937323036643539646335383565373462613834353739663033
6465653863303761643662366662306266363266353031640a303766613030366530356434373766
38633861663466336261383961663763396433323937356238623136393836333238646662643432
3139316564616264330a366166333165306561303465623661623863353962663937303633326436
61613935373162303430326630393731353962393763623636643266616662313434366162323130
66323636656135363236376461666533663539623234313263663535373931343065653732623931
30336437303134306666636335386266326138313166643335643264306462643536306337383839
39373261313465656562366464313734356165633034326138653061323966303133306261396533
61393763656262326663303334373437643636323330353161666161616136393461306437383638
38643366343135343836326535346335363162643135323536373536306331326365623163363036
61653231613231336165653466653934313231313265386130633630646539303834356332663532
61626461306134333234336163306437313533663932656434356430326334333263636132653961
61373331343432643266616166363264363830353166636163323065616634663962646366373335
34396436656163653437623134383266663861316161373535633733646132316463316131383632
303461396164386438373138613433323830
```
If you want to decrypt the file again, use the `decrypt` keyword:

```
~ACI_Ansible/Vault master ?1 ❯ ansible-vault decrypt vars/variables_aci.yml                                                              12s 14:55:17
Vault password: 
Decryption successful
```
You will notice you get back the original variables file.

### Executing Ansible Playbook with encrypted variables files

If we know want to execute this playbook, we can do so as follows:

```bash
~ACI_Ansible/Vault master ?1 ❯ ansible-playbook -i hosts_aci.yml playbook_aci_create.yml                                                  9s 15:00:14
ERROR! Attempting to decrypt but no vault secrets found
```
You will obviously get a message that Ansible does not know how to decrypt the content as it does not have the key we used to encrypt the file. There's multiple ways to achieve this. By far the easiest is to use the `--ask-vault-pass` option.

```bash
~ACI_Ansible/Vault master ?1 ❯ ansible-playbook -i hosts_aci.yml --ask-vault-pass  playbook_aci_create.yml
Vault password: 

PLAY [Limited ACI configuration] ***********************************************************************************************************************************************************************

TASK [Task 01 - Create Tenant] *************************************************************************************************************************************************************************
changed: [sandboxapicdc.cisco.com]

TASK [Task 02 - Create VRF] ****************************************************************************************************************************************************************************
changed: [sandboxapicdc.cisco.com]

TASK [Task 03 - Create BD] *****************************************************************************************************************************************************************************
changed: [sandboxapicdc.cisco.com] => (item={'bd': 'BD_Ansible', 'gateway': '10.16.100.1', 'mask': '24', 'scope': 'shared'})

TASK [Task 04 - Create subnet in BD] *******************************************************************************************************************************************************************
changed: [sandboxapicdc.cisco.com] => (item={'bd': 'BD_Ansible', 'gateway': '10.16.100.1', 'mask': '24', 'scope': 'shared'})

TASK [Task 05 - Add a new Application Profile] *********************************************************************************************************************************************************
changed: [sandboxapicdc.cisco.com]

TASK [Task 06 - Create EPG's and add to the Application Profile] ***************************************************************************************************************************************
changed: [sandboxapicdc.cisco.com] => (item={'epg': 'EPG_Ansible', 'bd': 'BD_Ansible'})

PLAY RECAP *********************************************************************************************************************************************************************************************
sandboxapicdc.cisco.com    : ok=6    changed=6    unreachable=0    failed=0    skipped=0    rescued=0    ignored=0   
```    

You will see that the playbook just executes perfectly well. However having to specify the password through the CLI might not be our most preferred way of working. So let's evaluate some other options as well. If you do not wish to type in the Vault password each time you execute a task, you can add your Vault password to a file and reference the file during execution.

For example, you could put your password in a .vault_pass file like this:

```bash
echo 'your_password' > .vault_pass
```
If you are using version control, make sure to add the password file to your version control software’s ignore file to avoid accidentally committing it:

```bash
echo '.vault_pass' >> .gitignore
```
We can then use it as follows:
```
~ACI_Ansible/Vault master ?1 ❯ ansible-playbook -i hosts_aci.yml --vault-password-file=.vault_pass playbook_aci_create.yml
```
Again, it might be kind of annoying to each time specify the `--vault-password-file` option, so we could also set an environment variable.
```
~ACI_Ansible/Vault master ?1 ❯ export ANSIBLE_VAULT_PASSWORD_FILE=./.vault_pass             
```
You will then be able to execute the initial command without having to specify the options:
```
~ACI_Ansible/Vault master ?1 ❯ ansible-playbook -i hosts_aci.yml  playbook_aci_create.yml   
```

### Executing Ansible Playbook with environment variables

Mission accomplished you say? Well...no.

An issue with all of the above methods is that we don't want to take the risk to check in the Vault password file into our repository. I know that we added the `.vault_pass` file in our .gitignore file so it should not be committed to our repository but a mistake is easily made, right? 

So if we ever want to have a CICD pipeline execute our Ansible playbook and we don't want to commit the password file, the question pops up how the CICD tool will get to know the secret password in order to be able to execute the Ansible playbook on our behalf. I hear you say we should use environment variables? Well yeah, but unfortunately Ansible does not have an environment variable for setting the password (e.g. similar to ANSIBLE_VAULT_PASSWORD_FILE)

So we need a workaround. The best way I have found in various sources is to create a small Python script that reads an environment variable. So let's explore that further.

Change the .vault_pass as follows:

```python
#!/usr/bin/env python

import os
print os.environ['VAULT_PASSWORD']
```
Next, as the Python script reads the `VAULT_PASSWORD`, we need to set it:

```bash
~ACI_Ansible/Vault master ?1 ❯ export VAULT_PASSWORD=*******
```

And lastly, let's execute the playbook again:

```bash
~ACI_Ansible/Vault master ?1 ❯ ansible-playbook -i hosts_aci.yml  playbook_aci_create.yml

PLAY [Limited ACI configuration] ***********************************************************************************************************************************************************************

TASK [Task 01 - Create Tenant] *************************************************************************************************************************************************************************
changed: [sandboxapicdc.cisco.com]

TASK [Task 02 - Create VRF] ****************************************************************************************************************************************************************************
changed: [sandboxapicdc.cisco.com]

TASK [Task 03 - Create BD] *****************************************************************************************************************************************************************************
changed: [sandboxapicdc.cisco.com] => (item={'bd': 'BD_Ansible', 'gateway': '10.16.100.1', 'mask': '24', 'scope': 'shared'})

TASK [Task 04 - Create subnet in BD] *******************************************************************************************************************************************************************
changed: [sandboxapicdc.cisco.com] => (item={'bd': 'BD_Ansible', 'gateway': '10.16.100.1', 'mask': '24', 'scope': 'shared'})

TASK [Task 05 - Add a new Application Profile] *********************************************************************************************************************************************************
changed: [sandboxapicdc.cisco.com]

TASK [Task 06 - Create EPG's and add to the Application Profile] ***************************************************************************************************************************************
changed: [sandboxapicdc.cisco.com] => (item={'epg': 'EPG_Ansible', 'bd': 'BD_Ansible'})

PLAY RECAP *********************************************************************************************************************************************************************************************
sandboxapicdc.cisco.com    : ok=6    changed=6    unreachable=0    failed=0    skipped=0    rescued=0    ignored=0  
```
And as expected, the playbook just runs normal and the ACI structures get created successfully.

That's it folks. Nothing world shocking but I find myself in my day job needing this quite often so I decided to dedicate a small post to it. Keep reading!
