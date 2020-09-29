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

Follow this guide: 

https://fabianlee.org/2019/04/01/kvm-creating-a-bridged-network-with-netplan-on-ubuntu-bionic/

### Introduction



### Check Virtualization extensions

wim@wim-nuc:~$ sudo apt install cpu-checker
wim@wim-nuc:~$ kvm-ok
INFO: /dev/kvm exists
KVM acceleration can be used



### Install KVM virtualization
wim@wim-nuc:~$ sudo apt install qemu-kvm libvirt-daemon-system libvirt-clients bridge-utils virtinst virt-manager

qemu-kvm - software that provides hardware emulation for the KVM hypervisor.
libvirt-daemon-system - configuration files to run the libvirt daemon as a system service.
libvirt-clients - software for managing virtualization platforms.
bridge-utils - a set of command-line tools for configuring ethernet bridges.
virtinst - a set of command-line tools for creating virtual machines.
virt-manager - an easy-to-use GUI interface and supporting command-line utilities for managing virtual machines through libvirt.

wim@wim-nuc:~$ sudo usermod -aG libvirt $USER
wim@wim-nuc:~$ sudo usermod -aG kvm $USER

A bridge named virbr0 is created:
wim@wim-nuc:~$ brctl show
bridge name	bridge id		STP enabled	interfaces
virbr0		8000.525400638110	yes		virbr0-nic

### Netplan configuration

```
wim@wim-nuc:~$ cd /etc/netplan
wim@wim-nuc:/etc/netplan$ ls
01-network-manager-all.yaml
wim@wim-nuc:/etc/netplan$ cat 01-network-manager-all.yaml 
# Let NetworkManager manage all devices on this system
network:
  version: 2
  renderer: NetworkManager

wim@wim-nuc:/etc/netplan$ sudo cp 01-network-manager-all.yaml 01-network-manager-all.yaml_bu
network:
  version: 2
  renderer: networkd

  ethernets:
    eno1:
      dhcp4: false 
      dhcp6: false 

  bridges:
    br0:
      interfaces: [eno1]
      addresses: [192.168.80.7/24]
      gateway4: 192.168.80.1
      mtu: 1500
      nameservers:
        addresses: [8.8.8.8]
      parameters:
        stp: true
        forward-delay: 4
      dhcp4: no
      dhcp6: no

wim@wim-nuc:/etc/netplan$ sudo netplan generate
wim@wim-nuc:/etc/netplan$ sudo netplan --debug apply
```

You will notice there is a new bridge called br0

```
wim@wim-nuc:/etc/netplan$ brctl show
bridge name	bridge id		STP enabled	interfaces
br0		8000.001fc69ccdc7	yes		eno1
virbr0		8000.525400638110	yes		virbr0-nic

wim@wim-nuc:/etc/netplan$ ifconfig
br0: flags=4163<UP,BROADCAST,RUNNING,MULTICAST>  mtu 1500
        inet 192.168.80.7  netmask 255.255.255.0  broadcast 192.168.80.255
        inet6 fe80::21f:c6ff:fe9c:cdc7  prefixlen 64  scopeid 0x20<link>
        ether 00:1f:c6:9c:cd:c7  txqueuelen 1000  (Ethernet)
        RX packets 3507  bytes 253945 (253.9 KB)
        RX errors 0  dropped 1  overruns 0  frame 0
        TX packets 2518  bytes 4502574 (4.5 MB)
        TX errors 0  dropped 0 overruns 0  carrier 0  collisions 0

eno1: flags=4163<UP,BROADCAST,RUNNING,MULTICAST>  mtu 1500
        ether 00:1f:c6:9c:cd:c7  txqueuelen 1000  (Ethernet)
        RX packets 57875  bytes 34239410 (34.2 MB)
        RX errors 0  dropped 2  overruns 0  frame 0
        TX packets 59493  bytes 45671055 (45.6 MB)
        TX errors 0  dropped 0 overruns 0  carrier 0  collisions 0
        device interrupt 16  memory 0xdf300000-df320000  

lo: flags=73<UP,LOOPBACK,RUNNING>  mtu 65536
        inet 127.0.0.1  netmask 255.0.0.0
        inet6 ::1  prefixlen 128  scopeid 0x10<host>
        loop  txqueuelen 1000  (Local Loopback)
        RX packets 522  bytes 50718 (50.7 KB)
        RX errors 0  dropped 0  overruns 0  frame 0
        TX packets 522  bytes 50718 (50.7 KB)
        TX errors 0  dropped 0 overruns 0  carrier 0  collisions 0

virbr0: flags=4099<UP,BROADCAST,MULTICAST>  mtu 1500
        inet 192.168.122.1  netmask 255.255.255.0  broadcast 192.168.122.255
        ether 52:54:00:63:81:10  txqueuelen 1000  (Ethernet)
        RX packets 0  bytes 0 (0.0 B)
        RX errors 0  dropped 0  overruns 0  frame 0
        TX packets 0  bytes 0 (0.0 B)
        TX errors 0  dropped 0 overruns 0  carrier 0  collisions 0

wlp3s0: flags=4163<UP,BROADCAST,RUNNING,MULTICAST>  mtu 1500
        inet 192.168.80.70  netmask 255.255.255.0  broadcast 192.168.80.255
        inet6 fe80::a018:4dca:aec3:4683  prefixlen 64  scopeid 0x20<link>
        ether a0:c5:89:86:9c:50  txqueuelen 1000  (Ethernet)
        RX packets 1811  bytes 319847 (319.8 KB)
        RX errors 0  dropped 0  overruns 0  frame 0
        TX packets 203  bytes 28759 (28.7 KB)
        TX errors 0  dropped 0 overruns 0  carrier 0  collisions 0


```

Configure libvrt to use the newly created bridge

```
wim@wim-nuc:~$ sudo nano host-bridge.xml

<network>
  <name>host-bridge</name>
  <forward mode="bridge"/>
  <bridge name="br0"/>
</network>

wim@wim-nuc:~$ sudo virsh net-define host-bridge.xml
Network host-bridge defined from host-bridge.xml

wim@wim-nuc:~$ virsh net-start host-bridge
error: failed to connect to the hypervisor
error: Failed to connect socket to '/var/run/libvirt/libvirt-sock': Permission denied

wim@wim-nuc:~$ sudo virsh net-start host-bridge
Network host-bridge started

wim@wim-nuc:~$ sudo virsh net-autostart host-bridge
Network host-bridge marked as autostarted

wim@wim-nuc:~$ virsh net-list --all
error: failed to connect to the hypervisor
error: Failed to connect socket to '/var/run/libvirt/libvirt-sock': Permission denied

wim@wim-nuc:~$ sudo virsh net-list --all
 Name          State    Autostart   Persistent
------------------------------------------------
 default       active   yes         yes
 host-bridge   active   yes         yes

```
