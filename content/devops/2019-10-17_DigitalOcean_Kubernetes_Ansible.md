---
title: DigitalOcean - Install Kubernetes using Ansible
date: 2019-10-17T14:39:50+01:00
draft: false
categories:
  - Infrastructure As Code
  - Cloud Native
  - Public Cloud
tags:
  - Terraform
  - DigitalOcean
  - Ansible
  - Kubernetes
---
> Quick note: the original post dates from 17-10-2019 but got updated at 01-04-2020 with latest Kubernetes version.

### Introduction
Couple of weeks ago, we have created a post where we created some servers on DigitalOcean. This post can be found here. If you want to follow along with this guide, then use that post to create 3 droplets.

If all went well, you will see the following screen in DigitalOcean.

![DigitalOcean](/images/2019-10-17-1.png)

Note: just for completeness sake, in the [script](https://github.com/wiwa1978/blog-hugo-netlify-code/blob/master/DigitalOcean_Kubernetes_Ansible/Terraform/create_server.tf) we used to create these 3 servers, we added also some output to print the IP addresses of all droplets to our terminal. Also, we added a variable to indicate the `type` of server we want to build. Reason for adding that type variable is that we need to select instance types with 2CPU's or more (s-2vcpu-2gb as a minimim in DigitalOcean), otherwise you will get an error message later:
> The number of available CPUs 1 is less than the required 2

```bash
Outputs:

IP_addresses = [
  "142.93.225.252",
  "134.209.92.116",
  "165.22.202.133",
]
```
### Using Ansible to install Kubernetes

In this post, we will focus on using Ansible to install Kubernetes. In fact, we will be implementing [this](https://www.digitalocean.com/community/tutorials/how-to-create-a-kubernetes-1-10-cluster-using-kubeadm-on-ubuntu-16-04) guide, be it on Ubuntu 18.04 with Kubernetes 1.19 and latest Calico release. In other words, this post is just a little more up to date but the general principles apply. 

First of all, let’s create a hosts file for our Ansible scripts. We will define 1 master and 2 workers. The IP addresses are the same as the ones in the DigitalOcean screenshot (obviously).

```yaml
#hosts
[masters]
master ansible_host=142.93.225.252 ansible_user=root

[workers]
worker1 ansible_host=134.209.92.116 ansible_user=root
worker2 ansible_host=165.22.202.133 ansible_user=root

[all:vars]
ansible_python_interpreter=/usr/bin/python3
ansible_ssh_private_key_file=~/.ssh/keypair_digitalocean
```
Next, we will  create a file to install all the updates, to create the ubuntu user and ensure the ubuntu user has sudo rights.

```yaml
#initial.yml
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
When the above is finished, we will create a specific file for the master node. This file will take care of the initialization of the Kubernetes cluster, will create the .kube directory and will copy the admin.conf file to the user’s kube.config file and also install the Calico network. 
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
      shell: kubectl apply -f https://docs.projectcalico.org/manifests/calico.yaml >> pod_network_setup.txt
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

Last file we will create is the workers.yml file. In that file, we will first retrieve the join command from the master node and then we will join worker nodes to the cluster.
```yaml
#workers.yml
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
If you followed along, we have a number of yml files. These are:

- hosts
- initial.yml
- kube-dependencies.yml
- master.yml
- workers.yml

### Executing Ansible playbooks

Once we have all the files, we can execute the Ansible playbooks. We will start with the initial.yml file.
```bash
WAUTERW-M-65P7:Ansible wauterw$ ansible-playbook -i hosts initial.yml

PLAY [all] ******************************************************************************************************************************************

TASK [Gathering Facts] ******************************************************************************************************************************
ok: [worker1]
ok: [worker2]
ok: [master]

TASK [create the 'ubuntu' user] *********************************************************************************************************************
changed: [worker1]
changed: [worker2]
changed: [master]

TASK [allow 'ubuntu' to have passwordless sudo] *****************************************************************************************************
changed: [worker1]
changed: [worker2]
changed: [master]

TASK [set up authorized keys for the ubuntu user] ***************************************************************************************************
changed: [worker1] => (item=ssh-rsa AAAAB3NzaC1yc2EAAAADAQABAAABAQC6ULleEqtsN/H4mpnuuhD/ZutJjb31mARluxU/nf2FlKuLKHuImc39opoMF wauterw@WAUTERW-M-65P7)
changed: [worker2] => (item=ssh-rsa AAAAB3NzaC1yc2EAAAADAQABAAABAQC6ULleEqtsN/ABzLNyc1mARluxU/nf2FlKuLKHuImc39opoMF wauterw@WAUTERW-M-65P7)
changed: [master] => (item=ssh-rsa AAAAB3NzaC1yc2EAAAADAQABAAABAQC6ULleEqtsN/GPSShD/ZutJjb31mARluxU/nf2FlKuLKHuImc39opoMF wauterw@WAUTERW-M-65P7)

PLAY RECAP ******************************************************************************************************************************************
master                     : ok=4    changed=3    unreachable=0    failed=0    skipped=0    rescued=0    ignored=0   
worker1                    : ok=4    changed=3    unreachable=0    failed=0    skipped=0    rescued=0    ignored=0   
worker2                    : ok=4    changed=3    unreachable=0    failed=0    skipped=0    rescued=0    ignored=0 
```
Next, we will install the Kubernetes dependencies:
```bash
WAUTERW-M-65P7:Ansible wauterw$ ansible-playbook -i hosts kube-dependencies.yml

PLAY [all] ******************************************************************************************************************************************

TASK [Gathering Facts] ******************************************************************************************************************************
ok: [master]
ok: [worker2]
ok: [worker1]

TASK [install Docker] *******************************************************************************************************************************
 [WARNING]: Could not find aptitude. Using apt-get instead

changed: [worker1]
changed: [worker2]
changed: [master]

TASK [install APT Transport HTTPS] ******************************************************************************************************************
changed: [worker2]
changed: [worker1]
changed: [master]

TASK [add Kubernetes apt-key] ***********************************************************************************************************************
changed: [worker2]
changed: [worker1]
changed: [master]

TASK [add Kubernetes' APT repository] ***************************************************************************************************************
changed: [worker1]
changed: [worker2]
changed: [master]

TASK [install kubelet] ******************************************************************************************************************************
changed: [master]
changed: [worker1]
changed: [worker2]

TASK [install kubeadm] ******************************************************************************************************************************
changed: [master]
changed: [worker1]
changed: [worker2]

PLAY [master] ***************************************************************************************************************************************

TASK [Gathering Facts] ******************************************************************************************************************************
ok: [master]

TASK [install kubectl] ******************************************************************************************************************************
ok: [master]

PLAY RECAP ******************************************************************************************************************************************
master                     : ok=9    changed=6    unreachable=0    failed=0    skipped=0    rescued=0    ignored=0   
worker1                    : ok=7    changed=6    unreachable=0    failed=0    skipped=0    rescued=0    ignored=0   
worker2                    : ok=7    changed=6    unreachable=0    failed=0    skipped=0    rescued=0    ignored=0   
```
When the dependencies are installed, it’s time to configure the master.

```bash
WAUTERW-M-65P7:Ansible wauterw$ ansible-playbook -i hosts master.yml

PLAY [master] ***************************************************************************************************************************************

TASK [Gathering Facts] ******************************************************************************************************************************
ok: [master]

TASK [initialize the cluster] ***********************************************************************************************************************
changed: [master]

TASK [create .kube directory] ***********************************************************************************************************************
 [WARNING]: Module remote_tmp /home/ubuntu/.ansible/tmp did not exist and was created with a mode of 0700, this may cause issues when running as
another user. To avoid this, create the remote_tmp dir with the correct permissions manually

changed: [master]

TASK [copy admin.conf to user's kube config] ********************************************************************************************************
changed: [master]

TASK [install Pod network] **************************************************************************************************************************
changed: [master]

PLAY RECAP ******************************************************************************************************************************************
master                     : ok=5    changed=4    unreachable=0    failed=0    skipped=0    rescued=0    ignored=0   

```
And finally, we will have the workers join our Kubernetes cluster by executing the workers playbook.
```bash
WAUTERW-M-65P7:Ansible wauterw$ ansible-playbook -i hosts workers.yml

PLAY [master] ***************************************************************************************************************************************

TASK [get join command] *****************************************************************************************************************************
changed: [master]

TASK [set join command] *****************************************************************************************************************************
ok: [master]

PLAY [workers] **************************************************************************************************************************************

TASK [Gathering Facts] ******************************************************************************************************************************
ok: [worker1]
ok: [worker2]

TASK [join cluster] *********************************************************************************************************************************
changed: [worker2]
changed: [worker1]

PLAY RECAP ******************************************************************************************************************************************
master                     : ok=2    changed=1    unreachable=0    failed=0    skipped=0    rescued=0    ignored=0   
worker1                    : ok=2    changed=1    unreachable=0    failed=0    skipped=0    rescued=0    ignored=0   
worker2                    : ok=2    changed=1    unreachable=0    failed=0    skipped=0    rescued=0    ignored=0  
```
The verify that everything went fine and according to plan, we can do a quick check by logging into the master node and check if the workers have joined successfully.
```bash
WAUTERW-M-65P7:Keys wauterw$ ssh -i keypair_digitalocean root@142.93.225.252
Welcome to Ubuntu 18.04.3 LTS (GNU/Linux 4.15.0-66-generic x86_64)

 * Documentation:  https://help.ubuntu.com
 * Management:     https://landscape.canonical.com
 * Support:        https://ubuntu.com/advantage

  System information as of Wed Apr  1 13:42:41 UTC 2020

  System load:  0.07              Users logged in:        0
  Usage of /:   5.4% of 57.98GB   IP address for eth0:    142.93.225.252
  Memory usage: 41%               IP address for docker0: 172.17.0.1
  Swap usage:   0%                IP address for tunl0:   10.244.130.64
  Processes:    145

0 packages can be updated.
0 updates are security updates.


Last login: Wed Apr  1 13:40:22 2020 from 173.38.220.54
root@kubernetes-0:~# su ubuntu
ubuntu@kubernetes-0:/root$ kubectl get nodes -o  wide
NAME           STATUS   ROLES    AGE     VERSION   INTERNAL-IP      EXTERNAL-IP   OS-IMAGE             KERNEL-VERSION      CONTAINER-RUNTIME
kubernetes-0   Ready    master   4m52s   v1.18.0   142.93.225.252   <none>        Ubuntu 18.04.3 LTS   4.15.0-66-generic   docker://19.3.6
kubernetes-1   Ready    <none>   3m53s   v1.18.0   134.209.92.116   <none>        Ubuntu 18.04.3 LTS   4.15.0-66-generic   docker://19.3.6
kubernetes-2   Ready    <none>   3m53s   v1.18.0   165.22.202.133   <none>        Ubuntu 18.04.3 LTS   4.15.0-66-generic   docker://19.3.6
```
As you can see the status indicates ready which means everything is just fine and also the networking CNI plugin was installed correctly.

That was it for today folks, hope you enjoyed this post. The code can be found at my [Github repo](https://github.com/wiwa1978/blog-hugo-netlify-code/tree/master/DigitalOcean_Kubernetes_Ansible).