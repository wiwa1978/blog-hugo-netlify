---
title: Install KVM on Centos8
date: 2020-04-10T14:39:50+01:00
draft: True
categories:
  - DevOps
  - Private Cloud
  - All
tags:
  - KVM
  - Centos
---

### Introduction



###
 

```
[wim@localhost ~]$ sudo yum update
[sudo] password for wim:
Last metadata expiration check: 0:17:26 ago on Sun 29 Mar 2020 12:40:52 PM CEST.
Dependencies resolved.
Nothing to do.
Complete!
```

### Check Virtualization extensions
```
[wim@localhost ~]$ lscpu | grep Virtualization
Virtualization:      VT-x
```

### Install KVM virtualization
```
[root@intelnuc wim]# dnf module install virt
[root@intelnuc wim]# dnf install virt-install virt-viewer
```

```
[root@intelnuc wim]# lsmod | grep kvm
kvm_intel             290816  0
kvm                   753664  1 kvm_intel
irqbypass              16384  1 kvm
```

```
[root@intelnuc wim]# virt-host-validate
setlocale: No such file or directory
  QEMU: Checking for hardware virtualization                                 : PASS
  QEMU: Checking if device /dev/kvm exists                                   : PASS
  QEMU: Checking if device /dev/kvm is accessible                            : PASS
  QEMU: Checking if device /dev/vhost-net exists                             : PASS
  QEMU: Checking if device /dev/net/tun exists                               : PASS
  QEMU: Checking for cgroup 'cpu' controller support                         : PASS
  QEMU: Checking for cgroup 'cpuacct' controller support                     : PASS
  QEMU: Checking for cgroup 'cpuset' controller support                      : PASS
  QEMU: Checking for cgroup 'memory' controller support                      : PASS
  QEMU: Checking for cgroup 'devices' controller support                     : PASS
  QEMU: Checking for cgroup 'blkio' controller support                       : PASS
  QEMU: Checking for device assignment IOMMU support                         : PASS
  QEMU: Checking if IOMMU is enabled by kernel                               : WARN (IOMMU appears to be disabled in kernel. Add intel_iommu=on to kernel cmdline arguments)
```

```
[root@intelnuc wim]# sudo usermod -aG libvirt wim
[root@intelnuc wim]# systemctl start libvirtd.service
[root@intelnuc wim]# systemctl enable libvirtd.service
[root@intelnuc wim]# systemctl status libvirtd.service
● libvirtd.service - Virtualization daemon
   Loaded: loaded (/usr/lib/systemd/system/libvirtd.service; enabled; vendor preset: enabled)
   Active: active (running) since Tue 2020-03-31 11:29:40 CEST; 11min ago
     Docs: man:libvirtd(8)
           https://libvirt.org
 Main PID: 19900 (libvirtd)
    Tasks: 19 (limit: 32768)
   Memory: 42.4M
   CGroup: /system.slice/libvirtd.service
           ├─ 3454 /usr/sbin/dnsmasq --conf-file=/var/lib/libvirt/dnsmasq/default.conf --leasefile-ro --dhcp-script=/usr/libexec/libvirt_leaseshelper
           ├─ 3456 /usr/sbin/dnsmasq --conf-file=/var/lib/libvirt/dnsmasq/default.conf --leasefile-ro --dhcp-script=/usr/libexec/libvirt_leaseshelper
           └─19900 /usr/sbin/libvirtd

Mar 31 11:29:40 intelnuc.wymedia systemd[1]: Starting Virtualization daemon...
Mar 31 11:29:40 intelnuc.wymedia systemd[1]: Started Virtualization daemon.
Mar 31 11:29:41 intelnuc.wymedia dnsmasq[3454]: read /etc/hosts - 2 addresses
Mar 31 11:29:41 intelnuc.wymedia dnsmasq[3454]: read /var/lib/libvirt/dnsmasq/default.addnhosts - 0 addresses
Mar 31 11:29:41 intelnuc.wymedia dnsmasq-dhcp[3454]: read /var/lib/libvirt/dnsmasq/default.hostsfile
```
### Install Virtual Machine Manager UI

```
[root@intelnuc wim]# sudo yum install virt-manager
```

### Install Cockpit

```
[root@intelnuc wim]# systemctl start cockpit.socket
[root@intelnuc wim]# systemctl enable cockpit.socket
Created symlink /etc/systemd/system/sockets.target.wants/cockpit.socket → /usr/lib/systemd/system/cockpit.socket.
[root@intelnuc wim]# systemctl status cockpit.socket
● cockpit.socket - Cockpit Web Service Socket
   Loaded: loaded (/usr/lib/systemd/system/cockpit.socket; enabled; vendor preset: disabled)
   Active: active (listening) since Tue 2020-03-31 11:41:53 CEST; 8s ago
     Docs: man:cockpit-ws(8)
   Listen: [::]:9090 (Stream)
    Tasks: 0 (limit: 26213)
   Memory: 2.0M
   CGroup: /system.slice/cockpit.socket

Mar 31 11:41:53 intelnuc.wymedia systemd[1]: Starting Cockpit Web Service Socket.
Mar 31 11:41:53 intelnuc.wymedia systemd[1]: Listening on Cockpit Web Service Socket.
```

### Create a networking bridge

```
[wim@localhost ~]$ sudo nmcli connection show
NAME    UUID                                  TYPE      DEVICE
eno1    3b2c1338-90d7-4f3d-b421-74b6d76e0619  ethernet  eno1
Virus   390e0e74-74c0-4f0e-8980-a459020243d0  wifi      wlp3s0
virbr0  43fd5ce0-abb4-4d54-909c-534b996cb6e6  bridge    virbr0
```


![VirtualBridge](/images/2020-04-10-1.png)
![VirtualBridge](/images/2020-04-10-2.png)
![VirtualBridge](/images/2020-04-10-3.png)
![VirtualBridge](/images/2020-04-10-4.png)
![VirtualBridge](/images/2020-04-10-5.png)
![VirtualBridge](/images/2020-04-10-6.png)