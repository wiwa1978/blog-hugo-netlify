---
title: AWS - Install Kubernetes using Ansible
date: 2019-11-25T14:39:50+01:00
draft: false
categories:
  - DevOps
  - Infrastructure As Code
  - Cloud Native
  - Public Cloud
  - All
tags:
  - Terraform
  - AWS
  - Ansible
  - Kubernetes
---
> Quick note: the original post dates from 25-11-2019 but got updated at 01-04-2020 with latest Kubernetes version.
### Introduction

This blog post is a follow up of a post we wrote couple of days ago where we created 3 EC2 instances on AWS. That blog post can be found here. If you want to follow along with this guide, then use that post to create 3 droplets.

Executing the code from that post, will result in three EC2 instances on AWS:

![AWS](/images/2019-11-25-1.png)

The output of the Terraform script gives me back the following IP addresses to be used in our Ansible hosts file.

```bash
Outputs:

instance_ip_addr = [
  "172.31.13.140",
  "172.31.5.24",
  "172.31.14.2",
]
instance_ips = [
  "18.203.68.253",
  "3.249.4.142",
  "3.249.141.242",
]
```

Note: ensure to select instance types with 2CPU's or more (t2.medium as a minimim in AWS), otherwise you will get an error message later:
> The number of available CPUs 1 is less than the required 2

### Using Ansible to install Kubernetes

In this post, we will focus on using Ansible to install Kubernetes. We will base ourselves on an excellent [post](https://www.digitalocean.com/community/tutorials/how-to-create-a-kubernetes-cluster-using-kubeadm-on-ubuntu-18-04) written by the DigitalOcean community but finetune it for our AWS endeavour. Also, what we do here with Ansible are essentially the same steps as I did in this post where I configured Kubernetes manually.

First of all, let’s create a hosts file for our Ansible scripts. We will define 1 master and 2 workers. The IP addresses are the same as the ones in the DigitalOcean screenshot (obviously).

```yaml
#hosts
[masters]
master ansible_host=18.203.68.253 ansible_user=ubuntu

[workers]
worker1 ansible_host=3.249.4.142 ansible_user=ubuntu
worker2 ansible_host=3.249.141.242 ansible_user=ubuntu

[all:vars]
ansible_python_interpreter=/usr/bin/python3
ansible_ssh_private_key_file=~/.ssh/AWS-Cisco.pem

```
Next, we will define create a file to install all the updates, to create the ubuntu user and ensure the ubuntu user has sudo rights.
```yaml
- hosts: all
  become: yes
  tasks:
    - name: create the 'ubuntu' user
      user: name=ubuntu append=yes state=present createhome=yes shell=/bin/bash

    - name: allow 'ubuntu' to have passwordless sudo
      lineinfile:
        dest: /etc/sudoers
        line: 'ubuntu ALL=(ALL) NOPASSWD: ALL'
        validate: 'visudo -cf %s'

    - name: set up authorized keys for the ubuntu user
      authorized_key: user=ubuntu key="{{item}}"
      with_file:
        - ~/.ssh/id_rsa.pub
```
Next up, we will create a different file that installs the Kubernetes specific dependencies on the 3 nodes. First of all, we start with the installation of Docker, then we add Kubernetes as well as the kubelet and kubeadm toolset.
```yaml
#kube-dependencies.yml
- hosts: all
  become: yes
  tasks:
   - name: install Docker
     apt:
       name: docker.io
       state: present
       update_cache: true

   - name: install APT Transport HTTPS
     apt:
       name: apt-transport-https
       state: present

   - name: add Kubernetes apt-key
     apt_key:
       url: https://packages.cloud.google.com/apt/doc/apt-key.gpg
       state: present

   - name: add Kubernetes' APT repository
     apt_repository:
      repo: deb http://apt.kubernetes.io/ kubernetes-xenial main
      state: present
      filename: 'kubernetes'

   - name: install kubelet
     apt:
       name: kubelet
       state: present
       update_cache: true

   - name: install kubeadm
     apt:
       name: kubeadm
       state: present

- hosts: master
  become: yes
  tasks:
   - name: install kubectl
     apt:
       name: kubectl
       state: present
```
When the above is finished, we will create a specific file for the master node. This file will take care of the initialization of the Kubernetes cluster, will create the .kube directory and will copy the admin.conf file to the user’s kube.config file and also install the Calico network. Just similar as what we did in the manual process.

```yaml
#master.yml
- hosts: master
  become: yes
  tasks:
    - name: initialize the cluster
      shell: kubeadm init --pod-network-cidr=10.244.0.0/16 >> cluster_initialized.txt
      args:
        chdir: $HOME
        creates: cluster_initialized.txt

    - name: create .kube directory
      become: yes
      become_user: ubuntu
      file:
        path: $HOME/.kube
        state: directory
        mode: 0755

    - name: copy admin.conf to user's kube config
      copy:
        src: /etc/kubernetes/admin.conf
        dest: /home/ubuntu/.kube/config
        remote_src: yes
        owner: ubuntu

    - name: install Pod network
      become: yes
      become_user: ubuntu
      shell: kubectl apply -f https://docs.projectcalico.org/v3.9/manifests/calico.yaml >> pod_network_setup.txt
      args:
        chdir: $HOME
        creates: pod_network_setup.txt

```
Note: if you prefer to use Flannel CNI plugin instead of Calico, it suffices to replace that part with:
```yaml
   - name: install Pod network
      become: yes
      become_user: ubuntu
      shell: kubectl apply -f https://raw.githubusercontent.com/coreos/flannel/master/Documentation/kube-flannel.yml >> pod_network_setup.txt
      args:
        chdir: $HOME
        creates: pod_network_setup.txt

```
Last file we create is the workers.yml file. In that file, we will first retrieve the join command from the master node and then we will join worker nodes to the cluster.
```yaml
- hosts: master
  become: yes
  gather_facts: false
  tasks:
    - name: get join command
      shell: kubeadm token create --print-join-command
      register: join_command_raw

    - name: set join command
      set_fact:
        join_command: "{{ join_command_raw.stdout_lines[0] }}"


- hosts: workers
  become: yes
  tasks:
    - name: join cluster
      shell: "{{ hostvars['master'].join_command }} >> node_joined.txt"
      args:
        chdir: $HOME
        creates: node_joined.txt

```
### Executing the Ansible playbooks

First, we will perform the initial configuration as described above.

```
WAUTERW-M-65P7:Ansible wauterw$ ansible-playbook -i hosts initial.yml

PLAY [all] ************************************************************************************************************************************

TASK [Gathering Facts] ************************************************************************************************************************
ok: [worker2]
ok: [master]
ok: [worker1]

TASK [create the 'ubuntu' user] ***************************************************************************************************************
ok: [master]
ok: [worker2]
ok: [worker1]

TASK [allow 'ubuntu' to have passwordless sudo] ***********************************************************************************************
changed: [worker1]
changed: [worker2]
changed: [master]

TASK [set up authorized keys for the ubuntu user] *********************************************************************************************
changed: [worker2] => (item=ssh-rsa AAAAB3NzaC1yc2EAAAADAQABAAABAQC6ULleEqtsN/OoaAmQbZCuHjNgDrRr3Gur33n6WfpuHCDNAiqTo5kr0jTKlhWPiBTtuokbONvyJGGL9qkpSmM5isr/fK88WpUOqEc28wYENHCQXsdRMZUYAdxPrtdQgAXSMJEHye9ea8e7E/U275c9D1kKNcHV/Xf06RNSqbHTFvbX79WqK3F6I6DG8rcogw+WGw0pH7lqg/8k1aWjH+apNXjl8NO3ABzLNyc3I6ytUvFOHIkRZeEPNuH4mpb122VAPTYiSpU+F6E8PQUomnGPSSMwfU+T0Sq33H4TwXkgRnuuhD/ZutJjb31mARluxU/nf2FlKuLKHuImc39opoMF wauterw@WAUTERW-M-65P7)
changed: [worker1] => (item=ssh-rsa AAAAB3NzaC1yc2EAAAADAQABAAABAQC6ULleEqtsN/OoaAmQbZCuHjNgDrRr3Gur33n6WfpuHCDNAiqTo5kr0jTKlhWPiBTtuokbONvyJGGL9qkpSmM5isr/fK88WpUOqEc28wYENHCQXsdRMZUYAdxPrtdQgAXSMJEHye9ea8e7E/U275c9D1kKNcHV/Xf06RNSqbHTFvbX79WqK3F6I6DG8rcogw+WGw0pH7lqg/8k1aWjH+apNXjl8NO3ABzLNyc3I6ytUvFOHIkRZeEPNuH4mpb122VAPTYiSpU+F6E8PQUomnGPSSMwfU+T0Sq33H4TwXkgRnuuhD/ZutJjb31mARluxU/nf2FlKuLKHuImc39opoMF wauterw@WAUTERW-M-65P7)
changed: [master] => (item=ssh-rsa AAAAB3NzaC1yc2EAAAADAQABAAABAQC6ULleEqtsN/OoaAmQbZCuHjNgDrRr3Gur33n6WfpuHCDNAiqTo5kr0jTKlhWPiBTtuokbONvyJGGL9qkpSmM5isr/fK88WpUOqEc28wYENHCQXsdRMZUYAdxPrtdQgAXSMJEHye9ea8e7E/U275c9D1kKNcHV/Xf06RNSqbHTFvbX79WqK3F6I6DG8rcogw+WGw0pH7lqg/8k1aWjH+apNXjl8NO3ABzLNyc3I6ytUvFOHIkRZeEPNuH4mpb122VAPTYiSpU+F6E8PQUomnGPSSMwfU+T0Sq33H4TwXkgRnuuhD/ZutJjb31mARluxU/nf2FlKuLKHuImc39opoMF wauterw@WAUTERW-M-65P7)

PLAY RECAP ************************************************************************************************************************************
master                     : ok=4    changed=2    unreachable=0    failed=0    skipped=0    rescued=0    ignored=0   
worker1                    : ok=4    changed=2    unreachable=0    failed=0    skipped=0    rescued=0    ignored=0   
worker2                    : ok=4    changed=2    unreachable=0    failed=0    skipped=0    rescued=0    ignored=0   
 
```
Next, we will install the Kubernetes dependencies:
```
WAUTERW-M-65P7:Ansible wauterw$ ansible-playbook -i hosts kube-dependencies.yml 

PLAY [all] ************************************************************************************************************************************

TASK [Gathering Facts] ************************************************************************************************************************
ok: [worker1]
ok: [worker2]
ok: [master]

TASK [install Docker] *************************************************************************************************************************
 [WARNING]: Could not find aptitude. Using apt-get instead

changed: [master]
changed: [worker2]
changed: [worker1]

TASK [install APT Transport HTTPS] ************************************************************************************************************
changed: [master]
changed: [worker2]
changed: [worker1]

TASK [add Kubernetes apt-key] *****************************************************************************************************************
changed: [worker2]
changed: [master]
changed: [worker1]

TASK [add Kubernetes' APT repository] *********************************************************************************************************
changed: [worker2]
changed: [worker1]
changed: [master]

TASK [install kubelet] ************************************************************************************************************************
changed: [worker2]
changed: [master]
changed: [worker1]

TASK [install kubeadm] ************************************************************************************************************************
changed: [worker1]
changed: [worker2]
changed: [master]

PLAY [master] *********************************************************************************************************************************

TASK [Gathering Facts] ************************************************************************************************************************
ok: [master]

TASK [install kubectl] ************************************************************************************************************************
ok: [master]

PLAY RECAP ************************************************************************************************************************************
master                     : ok=9    changed=6    unreachable=0    failed=0    skipped=0    rescued=0    ignored=0   
worker1                    : ok=7    changed=6    unreachable=0    failed=0    skipped=0    rescued=0    ignored=0   
worker2                    : ok=7    changed=6    unreachable=0    failed=0    skipped=0    rescued=0    ignored=0  
```
When the dependencies are installed, it’s time to configure the master.
```
WAUTERW-M-65P7:Ansible wauterw$ ansible-playbook -i hosts master.yml 

PLAY [master] *********************************************************************************************************************************

TASK [Gathering Facts] ************************************************************************************************************************
ok: [master]

TASK [initialize the cluster] *****************************************************************************************************************
changed: [master]

TASK [create .kube directory] *****************************************************************************************************************
changed: [master]

TASK [copy admin.conf to user's kube config] **************************************************************************************************
changed: [master]

TASK [install Pod network] ********************************************************************************************************************
changed: [master]

PLAY RECAP ************************************************************************************************************************************
master                     : ok=5    changed=4    unreachable=0    failed=0    skipped=0    rescued=0    ignored=0   
```
Next up, we are going to configure the workers.
```
WAUTERW-M-65P7:Ansible wauterw$ ansible-playbook -i hosts workers.yml 

PLAY [master] *********************************************************************************************************************************

TASK [get join command] ***********************************************************************************************************************
changed: [master]

TASK [set join command] ***********************************************************************************************************************
ok: [master]

PLAY [workers] ********************************************************************************************************************************

TASK [Gathering Facts] ************************************************************************************************************************
ok: [worker2]
ok: [worker1]

TASK [join cluster] ***************************************************************************************************************************
changed: [worker2]
changed: [worker1]

PLAY RECAP ************************************************************************************************************************************
master                     : ok=2    changed=1    unreachable=0    failed=0    skipped=0    rescued=0    ignored=0   
worker1                    : ok=2    changed=1    unreachable=0    failed=0    skipped=0    rescued=0    ignored=0   
worker2                    : ok=2    changed=1    unreachable=0    failed=0    skipped=0    rescued=0    ignored=0   
```
### Validate the K8S cluster

First, we need to SSH into the K8S master. In our case, this is available on ‘IP address’.
```
ssh ubuntu@18.203.68.253
```
This works, since we have configured the EC2 instances with our public key that is stored in our ~/.ssh directory (id_rsa,pub). Since we are also using our security key from AWS, we can also login as follows.
```
ssh -i AWS.pem ubuntu@18.203.68.253
```
Next, let’s see if our nodes are up:
```bash
ubuntu@ip-172-31-13-140:~$ kubectl get nodes -o  wide
NAME               STATUS   ROLES    AGE     VERSION   INTERNAL-IP     EXTERNAL-IP   OS-IMAGE             KERNEL-VERSION    CONTAINER-RUNTIME
ip-172-31-13-140   Ready    master   11m     v1.18.0   172.31.13.140   <none>        Ubuntu 18.04.1 LTS   4.15.0-1021-aws   docker://19.3.6
ip-172-31-14-2     Ready    <none>   4m29s   v1.18.0   172.31.14.2     <none>        Ubuntu 18.04.1 LTS   4.15.0-1021-aws   docker://19.3.6
ip-172-31-5-24     Ready    <none>   4m29s   v1.18.0   172.31.5.24     <none>        Ubuntu 18.04.1 LTS   4.15.0-1021-aws   docker://19.3.6
```
And our namespaces:
```bash
ubuntu@ip-172-31-13-140:~$ kubectl get namespaces
NAME              STATUS   AGE
default           Active   12m
kube-node-lease   Active   12m
kube-public       Active   12m
kube-system       Active   12m
```
We are now ready to deploy some applications onto our cluster. We'll blog about that at some later point in time. First wanted to get my head around the installation of the K8S clusters onto several plartforms. 

See you next time!



