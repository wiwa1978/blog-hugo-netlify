---
title: vSphere - Install Kubernetes using Ansible
date: 2019-12-02T14:39:50+01:00
draft: false
categories:
  - DevOps
  - Infrastructure As Code
  - Cloud Native
  - Private Cloud
tags:
  - Terraform
  - vSphere
  - Ansible
  - Kubernetes
---
### Introduction

In this post, we created a couple of virtual instances running on vSphere using Terraform. We will use that blog post as a reference to install 3 virtual servers on our vSphere environment.

Executing [this](https://github.com/wiwa1978/blog-hugo-netlify-code/tree/master/vSphere_Kubernetes_Ansible/Terraform) code, will result in three instances on vSphere. Obviously you will need to modify the variables file for your environment. Use it as a guideline.

![vSphere](/images/2019-12-02-1.png)

### Preparing the servers
As I'm using Ubuntu 16.04 as the underlying template, I found that the installation I did as part of the Ansible playbook did not work. So I decided for now to use Terraform to provision the three servers (see previous step) and then to manually install Docker. To do so, follow the next steps. Note that this should not be necessary on Ubuntu 18.04.

```bash
cisco@wauterw-k8s-master:~$ sudo add-get update
cisco@wauterw-k8s-master:~$ sudo add-get upgrade
cisco@wauterw-k8s-master:~$ sudo swapoff -a
cisco@wauterw-k8s-master:~$ sudo sed -i '/ swap / s/^\(.*\)$/#\1/g' /etc/fstab
cisco@wauterw-k8s-master:~$ sudo su
root@wauterw-k8s-master:~$ curl -fsSL https://download.docker.com/linux/ubuntu/gpg | apt-key add -
root@wauterw-k8s-master:~$ apt-get update && apt-get install apt-transport-https ca-certificates curl software-properties-common
root@wauterw-k8s-master:~$ add-apt-repository "deb [arch=amd64] https://download.docker.com/linux/ubuntu $(lsb_release -cs) stable"
root@wauterw-k8s-master:~$ apt update
root@wauterw-k8s-master:~$ apt install docker-ce
root@wauterw-k8s-master:~$ su cisco
cisco@wauterw-k8s-master:~$ usermod -aG docker ${USER}
cisco@wauterw-k8s-master:~$ cat > /etc/docker/daemon.json < {
>   "exec-opts": ["native.cgroupdriver=systemd"],
>   "log-driver": "json-file",
>   "log-opts": {
>     "max-size": "100m"
>   },
>   "storage-driver": "overlay2"
> }
> EOF
```
In case the above does not work, you can also simply create a file called `daemon.json` in the `/etc/docker` folder and cut and paste the following content.

```bash
{
  "exec-opts": ["native.cgroupdriver=systemd"],
  "log-driver": "json-file",
  "log-opts": {
    "max-size": "100m"
  },
  "storage-driver": "overlay2"
}
cisco@wauterw-k8s-master:~$ sudo su
root@wauterw-k8s:/home/cisco# mkdir -p /etc/systemd/system/docker.service.d
root@wauterw-k8s:/home/cisco# systemctl daemon-reload
root@wauterw-k8s:/home/cisco# systemctl restart docker
```
### Using Ansible to install Kubernetes
Next, let's install Kubernetes using Ansible. We did this already on AWS instances and DigitalOcean instances, so it should be pretty familiar in the meantime.

Let's begin with creating a hosts file for our Ansible scripts. We will define 1 master and 2 workers. The IP addresses are the same as the ones in the screenshot above. Obviously, adapt the hosts file to your own environment.
```bash
#hosts
[masters]
master ansible_host=10.16.2.236 ansible_user=cisco ansible_password=****

[workers]
worker1 ansible_host=10.16.2.237 ansible_user=cisco ansible_password=****
worker2 ansible_host=10.16.2.238 ansible_user=cisco ansible_password=****

[all:vars]
ansible_python_interpreter=/usr/bin/python3

```
Next, we will take care of some initial configuration steps. We call this file the `initial.yml` file and it essentially will create the ubuntu user, ensure the ubuntu user has sudo rights and setup the keys for us to be able to login via SSH.

```bash
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
        - /home/cisco/software/vSphere_Kubernetes_Ansible/lab_cisco_vsphere.pub
```
Next up, we will create a different file that installs the Kubernetes specific dependencies on the 3 nodes. In the other posts (installing Kubernetes on AWS and DigitalOcean) we would have first a task to install Docker. As mentioned above, we are running Ubuntu 16.04 and this requires some other steps, hence we installed Docker manually. This results in not having to define a 'install docker' tasks in the below `kube-dependencies.yml` file. All the other steps remain the same: add Kubernetes repository, install kubelet and kubeadm toolset...

```bash
#kube-dependencies.yml
- hosts: all
  become: yes
  tasks:
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
```bash
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
Last step is to create the workers.yml file. In that file, we will first retrieve the join command from the master node and then we will join worker nodes to the cluster.
```bash
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
That's it in terms of the ansible related files. Let's know continue with executing these playbooks.

### Executing Ansible playbooks
First, we will perform the initial configuration as described above. Simply run the Ansible playbook called `initial.yml`.
```bash
cisco@wauterw-ubuntu-desktop:~/software/vSphere_Kubernetes_Ansible/Ansible$ ansible-playbook -i hosts initial.yml -K
BECOME password: 

PLAY [all] ***********************************************************************************************************

TASK [Gathering Facts] ***********************************************************************************************
ok: [worker1]
ok: [master]
ok: [worker2]

TASK [create the 'ubuntu' user] **************************************************************************************
changed: [master]
changed: [worker2]
changed: [worker1]

TASK [allow 'ubuntu' to have passwordless sudo] **********************************************************************
changed: [worker1]
changed: [master]
changed: [worker2]

TASK [set up authorized keys for the ubuntu user] ********************************************************************
changed: [worker2] => (item=ssh-rsa AAAAB3NzaC1yBdB1WW0n73NlxKvIHE/pZk+oKk825QmRCTZy+yez5h30U= wauterw@WAUTERW-M-65P7)
changed: [worker1] => (item=ssh-rsa AAAAB3NzaC1yanyYlPB1WW0n73NlxKvIHE/pZk+oKk825QmRCyez5h30U= wauterw@WAUTERW-M-65P7)
changed: [master] => (item=ssh-rsa AAAAB3NzaC1yc2EAA8N0n73NlxKvIHE/pZk+oKk825QmRCTZy+yez5h30U= wauterw@WAUTERW-M-65P7)

PLAY RECAP ***********************************************************************************************************
master                     : ok=4    changed=3    unreachable=0    failed=0    skipped=0    rescued=0    ignored=0   
worker1                    : ok=4    changed=3    unreachable=0    failed=0    skipped=0    rescued=0    ignored=0   
worker2                    : ok=4    changed=3    unreachable=0    failed=0    skipped=0    rescued=0    ignored=0 
```
Next, we will install the Kubernetes dependencies:
```bash
cisco@wauterw-ubuntu-desktop:~/software/vSphere_Kubernetes_Ansible/Ansible$ ansible-playbook -i hosts kube-dependencies.yml -K
BECOME password: 

PLAY [all] ***********************************************************************************************************

TASK [Gathering Facts] ***********************************************************************************************
ok: [worker2]
ok: [worker1]
ok: [master]

TASK [install APT Transport HTTPS] ***********************************************************************************
ok: [worker2]
ok: [master]
ok: [worker1]

TASK [add Kubernetes apt-key] ****************************************************************************************
changed: [worker2]
changed: [worker1]
changed: [master]

TASK [add Kubernetes' APT repository] ********************************************************************************
changed: [worker2]
changed: [worker1]
changed: [master]

TASK [install kubelet] ***********************************************************************************************
changed: [worker1]
changed: [worker2]
changed: [master]

TASK [install kubeadm] ***********************************************************************************************
changed: [worker2]
changed: [master]
changed: [worker1]

PLAY [master] ********************************************************************************************************

TASK [Gathering Facts] ***********************************************************************************************
ok: [master]

TASK [install kubectl] ***********************************************************************************************
ok: [master]

PLAY RECAP ***********************************************************************************************************
master                     : ok=8    changed=4    unreachable=0    failed=0    skipped=0    rescued=0    ignored=0   
worker1                    : ok=6    changed=4    unreachable=0    failed=0    skipped=0    rescued=0    ignored=0   
worker2                    : ok=6    changed=4    unreachable=0    failed=0    skipped=0    rescued=0    ignored=0   
```
When the dependencies are installed, it’s time to configure the master.
```bash
cisco@wauterw-ubuntu-desktop:~/software/vSphere_Kubernetes_Ansible/Ansible$ ansible-playbook -i hosts master.yml -K
BECOME password: 

PLAY [master] ********************************************************************************************************

TASK [Gathering Facts] ***********************************************************************************************
ok: [master]

TASK [initialize the cluster] ****************************************************************************************
changed: [master]

TASK [create .kube directory] ****************************************************************************************
changed: [master]

TASK [copy admin.conf to user's kube config] *************************************************************************
changed: [master]

TASK [install Pod network] *******************************************************************************************
changed: [master]

PLAY RECAP ***********************************************************************************************************
master                     : ok=5    changed=4    unreachable=0    failed=0    skipped=0    rescued=0    ignored=0   
```
And finally, we will have the workers join our Kubernetes cluster by executing the workers playbook.
```bash
cisco@wauterw-ubuntu-desktop:~/software/vSphere_Kubernetes_Ansible/Ansible$ ansible-playbook -i hosts workers.yml -K
BECOME password: 

PLAY [master] ********************************************************************************************************

TASK [get join command] **********************************************************************************************
changed: [master]

TASK [set join command] **********************************************************************************************
ok: [master]

PLAY [workers] *******************************************************************************************************

TASK [Gathering Facts] ***********************************************************************************************
ok: [worker2]
ok: [worker1]

TASK [join cluster] **************************************************************************************************
changed: [worker2]
changed: [worker1]

PLAY RECAP ***********************************************************************************************************
master                     : ok=2    changed=1    unreachable=0    failed=0    skipped=0    rescued=0    ignored=0   
worker1                    : ok=2    changed=1    unreachable=0    failed=0    skipped=0    rescued=0    ignored=0   
worker2                    : ok=2    changed=1    unreachable=0    failed=0    skipped=0    rescued=0    ignored=0    
```
### Result
The verify that everything went fine and according to plan, we can do a quick check by logging into the master node and check if the workers have joined successfully.
```
ubuntu@wauterw-k8s-master:/etc/docker$ kubectl get nodes -o wide
NAME                   STATUS   ROLES    AGE    VERSION   INTERNAL-IP   EXTERNAL-IP   OS-IMAGE             KERNEL-VERSION      CONTAINER-RUNTIME
wauterw-k8s-master     Ready    master   10m    v1.18.0   10.16.2.236   <none>        Ubuntu 16.04.6 LTS   4.4.0-154-generic   docker://19.3.8
wauterw-k8s-worker01   Ready    <none>   8m2s   v1.18.0   10.16.2.237   <none>        Ubuntu 16.04.6 LTS   4.4.0-154-generic   docker://19.3.8
wauterw-k8s-worker02   Ready    <none>   8m2s   v1.18.0   10.16.2.238   <none>        Ubuntu 16.04.6 LTS   4.4.0-154-generic   docker://19.3.8
Take a look at the files on my Github [repo](https://github.com/wiwa1978/blog-hugo-netlify-code/tree/master/vSphere_Kubernetes_Ansible).

```