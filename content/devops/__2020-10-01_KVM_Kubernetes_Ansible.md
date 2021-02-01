---
title: Install KVM on Ubuntu
date: 2020-04-10T14:39:50+01:00
draft: True
categories:
  - DevOps
  - Private Cloud
  - All
tags:
  - KVM
  - Ubuntu
---




### Preparing the servers

cisco@wauterw-k8s-master:~$ sudo apt-get remove docker docker-engine docker.io containerd runc

cisco@wauterw-k8s-master:~$ sudo swapoff -a

Note: this needs to be done permanently so a better approach would be to comment out the swap line in your /etc/fstab file and reboot your VM.



cisco@wauterw-k8s-master:~$ sudo apt-get update

OK cisco@wauterw-k8s-master:~$ sudo apt-get install \
    apt-transport-https \
    ca-certificates \
    curl \
    gnupg-agent \
    software-properties-common

OK cisco@wauterw-k8s-master:~$ curl -fsSL https://download.docker.com/linux/ubuntu/gpg | sudo apt-key add -

OK cisco@wauterw-k8s-master:~$ sudo add-apt-repository \
   "deb [arch=amd64] https://download.docker.com/linux/ubuntu \
   $(lsb_release -cs) \
   stable"

OK cisco@wauterw-k8s-master:~$ sudo apt-get update
OK cisco@wauterw-k8s-master:~$ sudo apt-get install docker-ce docker-ce-cli containerd.io

cisco@wauterw-k8s-master:~$ sudo nano /etc/docker/daemon.json

 {
  "exec-opts": ["native.cgroupdriver=systemd"],
  "log-driver": "json-file",
  "log-opts": {
    "max-size": "100m"
  },
  "storage-driver": "overlay2"
}
cisco@wauterw-k8s-master:~$ sudo usermod -aG docker wim
cisco@wauterw-k8s-master:~$ sudo su
root@wauterw-k8s:/home/cisco# mkdir -p /etc/systemd/system/docker.service.d
root@wauterw-k8s:/home/cisco# systemctl daemon-reload
root@wauterw-k8s:/home/cisco# systemctl restart docker








wauterw@WAUTERW-M-65P7 Keys_and_Certificates % ssh-copy-id -i homelab.pub wim@192.168.80.122
/usr/bin/ssh-copy-id: INFO: Source of key(s) to be installed: "homelab.pub"
/usr/bin/ssh-copy-id: INFO: attempting to log in with the new key(s), to filter out any that are already installed
/usr/bin/ssh-copy-id: INFO: 1 key(s) remain to be installed -- if you are prompted now it is to install the new keys
wim@192.168.80.122's password:

Number of key(s) added:        1

Now try logging into the machine, with:   "ssh 'wim@192.168.80.122'"
and check to make sure that only the key(s) you wanted were added.






wauterw@WAUTERW-M-65P7 Ansible % ansible-playbook -i hosts initial.yml -k -K
➜  Ansible git:(master) ✗ ansible-playbook -i hosts -i hosts initial.yml -k -K 
SSH password: 
BECOME password[defaults to SSH password]: 

PLAY [all] ***********************************************************************************************************************************************************************************************************

TASK [Gathering Facts] ***********************************************************************************************************************************************************************************************
ok: [worker1]
ok: [worker2]
ok: [master]

TASK [create the 'wim' user] *****************************************************************************************************************************************************************************************
ok: [master]
ok: [worker2]
ok: [worker1]

TASK [allow 'wim' to have passwordless sudo] *************************************************************************************************************************************************************************
changed: [worker2]
changed: [master]
changed: [worker1]

TASK [set up authorized keys for the ubuntu user] ********************************************************************************************************************************************************************
ok: [worker1] => (item=ssh-rsa AAAAB3NzaC1yc2EAAAADAQABAAACAQDC9FjfJgM87InjdJe6hWncxjafmC0+5Yo68bgnMwheV131MF7z6u1RiePSYTWD8ySttz488CVhh3EE9+tu3E2PFu1RZLdfofRSssRuPo5hG4npHPn9BCO8x2as/XEVaLzvGIpUodyXN82a+wjitJkJI3LRoc3tB1ltQdPlmF3VHLa2goz5yt/A0gc1anax2Z4Z4q4ga9DCaBIx9RcNOuHD5yAl/prWumBgGJ3bth4QRsxo2nIsAkT+v5pkH/Phdpg5crCfEknyYy/BER95UUoJpeetg8dZVGTeGan4ZUcpANpBPXnFiDwkcRIvMFgRMG0zaIPPfG7qSI9+RUhDm9AgBL3wcyXCjsI75Lyi0mhgh8h/7ee/HEg1FojL3FF656jQ0NXdkc44K0wDpN/TBw1pNbxN/CZCL7vafZ7v5C1VftvGvpQSjtO4zNRiiYjUpcEvkoGeljR25DHRpfZbv0Byjv9NFIBL3RUAy/s0+da3y9jg0f/N6bDBs6izJMrzCVMGfDRUOTGKVhYiXTJF6G7afcZpYl5t6oL6mao62sOUV+9AkrYMCegGRWDYXvzxcU9wFYsIOOYUNJ5cO09vQCGzn+wGJ6sMJHc6NDTwXIEzyFYk+RPeC6Nero1EeKkPEj1U2qwWFgAA+d1FSOazHQt0x9x6VBuEYIeu5ta25rWTJQ== Homelab_Wim)
ok: [master] => (item=ssh-rsa AAAAB3NzaC1yc2EAAAADAQABAAACAQDC9FjfJgM87InjdJe6hWncxjafmC0+5Yo68bgnMwheV131MF7z6u1RiePSYTWD8ySttz488CVhh3EE9+tu3E2PFu1RZLdfofRSssRuPo5hG4npHPn9BCO8x2as/XEVaLzvGIpUodyXN82a+wjitJkJI3LRoc3tB1ltQdPlmF3VHLa2goz5yt/A0gc1anax2Z4Z4q4ga9DCaBIx9RcNOuHD5yAl/prWumBgGJ3bth4QRsxo2nIsAkT+v5pkH/Phdpg5crCfEknyYy/BER95UUoJpeetg8dZVGTeGan4ZUcpANpBPXnFiDwkcRIvMFgRMG0zaIPPfG7qSI9+RUhDm9AgBL3wcyXCjsI75Lyi0mhgh8h/7ee/HEg1FojL3FF656jQ0NXdkc44K0wDpN/TBw1pNbxN/CZCL7vafZ7v5C1VftvGvpQSjtO4zNRiiYjUpcEvkoGeljR25DHRpfZbv0Byjv9NFIBL3RUAy/s0+da3y9jg0f/N6bDBs6izJMrzCVMGfDRUOTGKVhYiXTJF6G7afcZpYl5t6oL6mao62sOUV+9AkrYMCegGRWDYXvzxcU9wFYsIOOYUNJ5cO09vQCGzn+wGJ6sMJHc6NDTwXIEzyFYk+RPeC6Nero1EeKkPEj1U2qwWFgAA+d1FSOazHQt0x9x6VBuEYIeu5ta25rWTJQ== Homelab_Wim)
ok: [worker2] => (item=ssh-rsa AAAAB3NzaC1yc2EAAAADAQABAAACAQDC9FjfJgM87InjdJe6hWncxjafmC0+5Yo68bgnMwheV131MF7z6u1RiePSYTWD8ySttz488CVhh3EE9+tu3E2PFu1RZLdfofRSssRuPo5hG4npHPn9BCO8x2as/XEVaLzvGIpUodyXN82a+wjitJkJI3LRoc3tB1ltQdPlmF3VHLa2goz5yt/A0gc1anax2Z4Z4q4ga9DCaBIx9RcNOuHD5yAl/prWumBgGJ3bth4QRsxo2nIsAkT+v5pkH/Phdpg5crCfEknyYy/BER95UUoJpeetg8dZVGTeGan4ZUcpANpBPXnFiDwkcRIvMFgRMG0zaIPPfG7qSI9+RUhDm9AgBL3wcyXCjsI75Lyi0mhgh8h/7ee/HEg1FojL3FF656jQ0NXdkc44K0wDpN/TBw1pNbxN/CZCL7vafZ7v5C1VftvGvpQSjtO4zNRiiYjUpcEvkoGeljR25DHRpfZbv0Byjv9NFIBL3RUAy/s0+da3y9jg0f/N6bDBs6izJMrzCVMGfDRUOTGKVhYiXTJF6G7afcZpYl5t6oL6mao62sOUV+9AkrYMCegGRWDYXvzxcU9wFYsIOOYUNJ5cO09vQCGzn+wGJ6sMJHc6NDTwXIEzyFYk+RPeC6Nero1EeKkPEj1U2qwWFgAA+d1FSOazHQt0x9x6VBuEYIeu5ta25rWTJQ== Homelab_Wim)

PLAY RECAP ***********************************************************************************************************************************************************************************************************
master                     : ok=4    changed=1    unreachable=0    failed=0    skipped=0    rescued=0    ignored=0   
worker1                    : ok=4    changed=1    unreachable=0    failed=0    skipped=0    rescued=0    ignored=0   
worker2                    : ok=4    changed=1    unreachable=0    failed=0    skipped=0    rescued=0    ignored=0   












wauterw@WAUTERW-M-65P7 Ansible % ansible-playbook -i hosts kube-dependencies.yml -k -K 
➜  Ansible git:(master) ✗ 
➜  Ansible git:(master) ✗ 
➜  Ansible git:(master) ✗ ansible-playbook -i hosts kube-dependencies.yml -k -K 
SSH password: 
BECOME password[defaults to SSH password]: 

PLAY [all] ***********************************************************************************************************************************************************************************************************

TASK [Gathering Facts] ***********************************************************************************************************************************************************************************************
ok: [worker1]
ok: [worker2]
ok: [master]

TASK [install APT Transport HTTPS] ***********************************************************************************************************************************************************************************
ok: [worker2]
ok: [worker1]
ok: [master]

TASK [add Kubernetes apt-key] ****************************************************************************************************************************************************************************************
changed: [master]
changed: [worker1]
changed: [worker2]

TASK [add Kubernetes' APT repository] ********************************************************************************************************************************************************************************
changed: [master]
changed: [worker1]
changed: [worker2]

TASK [install kubelet] ***********************************************************************************************************************************************************************************************
changed: [master]
changed: [worker2]
changed: [worker1]

TASK [install kubeadm] ***********************************************************************************************************************************************************************************************
changed: [worker1]
changed: [master]
changed: [worker2]

PLAY [master] ********************************************************************************************************************************************************************************************************

TASK [Gathering Facts] ***********************************************************************************************************************************************************************************************
ok: [master]

TASK [install kubectl] ***********************************************************************************************************************************************************************************************
ok: [master]

PLAY RECAP ***********************************************************************************************************************************************************************************************************
master                     : ok=8    changed=4    unreachable=0    failed=0    skipped=0    rescued=0    ignored=0   
worker1                    : ok=6    changed=4    unreachable=0    failed=0    skipped=0    rescued=0    ignored=0   
worker2                    : ok=6    changed=4    unreachable=0    failed=0    skipped=0    rescued=0    ignored=0   
















➜  Ansible git:(master) ✗ ansible-playbook -i hosts master.yml -k -K
SSH password: 
BECOME password[defaults to SSH password]: 

PLAY [master] ************************************************************************************************************************************************************************

TASK [Gathering Facts] ***************************************************************************************************************************************************************
ok: [master]

TASK [initialize the cluster] ********************************************************************************************************************************************************
ok: [master]

TASK [create .kube directory] ********************************************************************************************************************************************************
ok: [master]

TASK [copy admin.conf to user's kube config] *****************************************************************************************************************************************
changed: [master]

TASK [install Pod network] ***********************************************************************************************************************************************************
changed: [master]

PLAY RECAP ***************************************************************************************************************************************************************************
master                     : ok=5    changed=2    unreachable=0    failed=0    skipped=0    rescued=0    ignored=0   










wauterw@WAUTERW-M-65P7 Ansible % ansible-playbook -i hosts workers.yml -k -K 
➜  Ansible git:(master) ✗ ansible-playbook -i hosts workers.yml -k -K 
SSH password: 
BECOME password[defaults to SSH password]: 

PLAY [master] ************************************************************************************************************************************************************************

TASK [get join command] **************************************************************************************************************************************************************
changed: [master]

TASK [set join command] **************************************************************************************************************************************************************
ok: [master]

PLAY [workers] ***********************************************************************************************************************************************************************

TASK [Gathering Facts] ***************************************************************************************************************************************************************
ok: [worker1]
ok: [worker2]

TASK [join cluster] ******************************************************************************************************************************************************************
changed: [worker1]
changed: [worker2]

PLAY RECAP ***************************************************************************************************************************************************************************
master                     : ok=2    changed=1    unreachable=0    failed=0    skipped=0    rescued=0    ignored=0   
worker1                    : ok=2    changed=1    unreachable=0    failed=0    skipped=0    rescued=0    ignored=0   
worker2                    : ok=2    changed=1    unreachable=0    failed=0    skipped=0    rescued=0    ignored=0 










wim@k8s-master:/etc/kubernetes$ kubectl get nodes
NAME          STATUS   ROLES    AGE     VERSION
k8s-master    Ready    master   5m23s   v1.18.8
k8s-worker1   Ready    <none>   89s     v1.18.8
k8s-worker2   Ready    <none>   89s     v1.18.8





In case of problems:

1) wim@k8s-master:~$ kubectl get nodes => The connection to the server 192.168.80.121:6443 was refused - did you specify the right host or port?

Solution:



wim@k8s-master:~$ docker ps
CONTAINER ID        IMAGE               COMMAND             CREATED             STATUS              PORTS               NAMES
wim@k8s-master:~$ systemctl enable kubelet
==== AUTHENTICATING FOR org.freedesktop.systemd1.manage-unit-files ===
Authentication is required to manage system service or unit files.
Authenticating as: Wim (wim)
Password:
==== AUTHENTICATION COMPLETE ===
==== AUTHENTICATING FOR org.freedesktop.systemd1.reload-daemon ===
Authentication is required to reload the systemd state.
Authenticating as: Wim (wim)
Password:
==== AUTHENTICATION COMPLETE ===


Then you will see that all docker containers are up again and kubectl get nodes will work



2) The worker nodes are metnionin Not Healthy

wim@k8s-master:~$ kubectl get nodes
NAME           STATUS     ROLES    AGE   VERSION
k8s-master     Ready      master   27m   v1.18.3
k8s-worker-1   NotReady   <none>   25m   v1.18.3
k8s-worker-2   Ready      <none>   25m   v1.18.3


Edit /etc/fstab and comment out the swap line:

wim@k8s-master:~$ cat /etc/fstab
# /etc/fstab: static file system information.
#
# Use 'blkid' to print the universally unique identifier for a
# device; this may be used with UUID= as a more robust way to name devices
# that works even if disks are added and removed. See fstab(5).
#
# <file system> <mount point>   <type>  <options>       <dump>  <pass>
# / was on /dev/vda2 during curtin installation
/dev/disk/by-uuid/178f4902-1fb6-4991-b605-8e4621c147fd / ext4 defaults 0 0
#/swap.img	none	swap	sw	0	0

Then reboot and it should work:

wim@k8s-master:~$ kubectl get nodes
NAME           STATUS   ROLES    AGE   VERSION
k8s-master     Ready    master   30m   v1.18.3
k8s-worker-1   Ready    <none>   28m   v1.18.3
k8s-worker-2   Ready    <none>   28m   v1.18.3