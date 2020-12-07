---
title: Packer Introduction with vSphere
date: 2021-10-18T14:39:50+01:00
draft: True
categories:
  - DevOps
  - All
tags:
  - Packer
  - AWS
---
### Introduction

https://medium.com/@gmusumeci/how-to-use-packer-to-build-an-ubuntu-18-04-template-for-vmware-vsphere-da5bbf1be92f

http://cdimage.ubuntu.com/releases/18.04/release/

http://cdimage.ubuntu.com/releases/18.04/release/SHA256SUMS

https://help.ubuntu.com/lts/installation-guide/example-preseed.txt

https://www.packer.io/guides/automatic-operating-system-installs/preseed_ubuntu

https://www.thehumblelab.com/automating-ubuntu-18-packer/#_

### Code

```
{
    "builders": [
      {
        "CPUs": "{{user `vm-cpu-num`}}",
        "RAM": "{{user `vm-mem-size`}}",
        "RAM_reserve_all": true,
        "boot_command": [
          "<enter><wait><f6><wait><esc><wait>",
          "<bs><bs><bs><bs><bs><bs><bs><bs><bs><bs>",
          "<bs><bs><bs><bs><bs><bs><bs><bs><bs><bs>",
          "<bs><bs><bs><bs><bs><bs><bs><bs><bs><bs>",
          "<bs><bs><bs><bs><bs><bs><bs><bs><bs><bs>",
          "<bs><bs><bs><bs><bs><bs><bs><bs><bs><bs>",
          "<bs><bs><bs><bs><bs><bs><bs><bs><bs><bs>",
          "<bs><bs><bs><bs><bs><bs><bs><bs><bs><bs>",
          "<bs><bs><bs><bs><bs><bs><bs><bs><bs><bs>",
          "<bs><bs><bs>",
          "/install/vmlinuz",
          " initrd=/install/initrd.gz",
          " priority=critical",
          " locale=en_US",
          " file=/media/preseed.cfg",
          "<enter>"
        ],
        "boot_order": "disk,cdrom",
        "cluster": "{{user `cluster`}}",
        "convert_to_template": "true",
        "datacenter": "{{user `datacenter`}}",
        "datastore": "{{user `datastore`}}",
        "disk_controller_type": "pvscsi",
        "floppy_files": [
          "./preseed.cfg"
        ],
        "folder": "{{user `folder`}}",
        "guest_os_type": "ubuntu64Guest",
        "host": "{{user `host`}}",
        "insecure_connection": "true",
        "iso_checksum": "{{user `iso-checksum-type`}}:{{user `iso-checksum`}}",
        "iso_urls": "{{user `iso-url`}}",
        "network_adapters": [
          {
            "network": "{{user `network`}}",
            "network_card": "vmxnet3"
          }
        ],
        "password": "{{user `vcenter-password`}}",
        "ssh_password": "{{user `ssh-password`}}",
        "ssh_username": "{{user `ssh-username`}}",
        "storage": [
          {
            "disk_size": "{{user `vm-disk-size`}}",
            "disk_thin_provisioned": true
          }
        ],
        "type": "vsphere-iso",
        "username": "{{user `vcenter-username`}}",
        "vcenter_server": "{{user `vcenter-server`}}",
        "vm_name": "{{user `vm-name`}}"
      }
    ],
    "provisioners": [
      {
        "inline": [
          "echo 'Packer Template Build -- Complete'"
        ],
        "type": "shell"
      }
    ]
  }
```

```
{
    "vm-name":            "Ubuntu-Packer",
    "vcenter-server":     "vcenter-amslab.cisco.com",
    "vcenter-username":   "administrator@vsphere.local",
    "vcenter-password":   "C!sco12345",
    "datacenter":         "Amsterdam",
    "datastore":          "HX-CONTENT-ACI",
    "folder":             "wauterw",
    "cluster":            "HX-ACI",
    "network":            "vm-network-98",
    "vm-cpu-num":         "1",
    "vm-mem-size":        "1024",
    "vm-disk-size":       "10240",
    "ssh-username":       "cisco",
    "ssh-password":       "C!sco12345",
    "iso-url":            "http://cdimage.ubuntu.com/releases/18.04/release/ubuntu-18.04.5-server-amd64.iso",
    "iso-checksum":       "8c5fc24894394035402f66f3824beb7234b757dd2b5531379cb310cedfdf0996",
    "iso-checksum-type":  "sha256"
  }
```

```
# Setting the locales, country
# Supported locales available in /usr/share/i18n/SUPPORTED
d-i debian-installer/language string en
d-i debian-installer/country string us
d-i debian-installer/locale string en_US.UTF-8

# Keyboard setting
d-i console-setup/ask_detect boolean false
d-i keyboard-configuration/layoutcode string us
d-i keyboard-configuration/xkb-keymap us
d-i keyboard-configuration/modelcode string pc105

# User creation
d-i passwd/user-fullname string cisco
d-i passwd/username string cisco
d-i passwd/user-password password C!sco12345
d-i passwd/user-password-again password C!sco12345
d-i user-setup/allow-password-weak boolean true

# Disk and Partitioning setup
d-i partman-auto/disk string /dev/sda
d-i partman-auto/method string regular
d-i partman-partitioning/confirm_write_new_label boolean true
d-i partman/choose_partition select finish
d-i partman/confirm boolean true
d-i partman/confirm_nooverwrite boolean true

# Set root password
d-i passwd/root-login boolean true
d-i passwd/root-password password kopicloud
d-i passwd/root-password-again password kopicloud

# Package installations
d-i pkgsel/include string open-vm-tools openssh-server

d-i grub-installer/only_debian boolean true

d-i preseed/late_command string \
    echo 'kopicloud ALL=(ALL) NOPASSWD: ALL' > /target/etc/sudoers.d/kopicloud ; \
    in-target chmod 440 /etc/sudoers.d/kopicloud ;

d-i finish-install/reboot_in_progress note

```


```
~/SynologyDrive/Programming/Packer/vSphere ❯ packer build -var-file variables.json server.json                                        6m 23s 20:34:19
vsphere-iso: output will be in this color.

==> vsphere-iso: Retrieving ISO
==> vsphere-iso: Trying http://cdimage.ubuntu.com/releases/18.04/release/ubuntu-18.04.5-server-amd64.iso
==> vsphere-iso: Trying http://cdimage.ubuntu.com/releases/18.04/release/ubuntu-18.04.5-server-amd64.iso?checksum=sha256%3A8c5fc24894394035402f66f3824beb7234b757dd2b5531379cb310cedfdf0996
==> vsphere-iso: http://cdimage.ubuntu.com/releases/18.04/release/ubuntu-18.04.5-server-amd64.iso?checksum=sha256%3A8c5fc24894394035402f66f3824beb7234b757dd2b5531379cb310cedfdf0996 => /Users/wauterw/SynologyDrive/Programming/Packer/vSphere/packer_cache/a37af95ab12e665ba168128cde2f3662740b21a2.iso
==> vsphere-iso: Uploading a37af95ab12e665ba168128cde2f3662740b21a2.iso to packer_cache/a37af95ab12e665ba168128cde2f3662740b21a2.iso
==> vsphere-iso: Creating VM...
==> vsphere-iso: Customizing hardware...
==> vsphere-iso: Mounting ISO images...
==> vsphere-iso: Adding configuration parameters...
==> vsphere-iso: Creating floppy disk...
    vsphere-iso: Copying files flatly from floppy_files
    vsphere-iso: Copying file: ./preseed.cfg
    vsphere-iso: Done copying files from floppy_files
    vsphere-iso: Collecting paths from floppy_dirs
    vsphere-iso: Resulting paths from floppy_dirs : []
    vsphere-iso: Done copying paths from floppy_dirs
==> vsphere-iso: Uploading created floppy image
==> vsphere-iso: Adding generated Floppy...
==> vsphere-iso: Set boot order...
==> vsphere-iso: Power on VM...
==> vsphere-iso: Waiting 10s for boot...
==> vsphere-iso: Typing boot command...
==> vsphere-iso: Waiting for IP...
==> vsphere-iso: IP address: 10.61.124.228
==> vsphere-iso: Using ssh communicator to connect: 10.61.124.228
==> vsphere-iso: Waiting for SSH to become available...
==> vsphere-iso: Connected to SSH!
==> vsphere-iso: Provisioning with shell script: /var/folders/27/hzk_ld5j6gb516mtp077gv600000gq/T/packer-shell686057022
    vsphere-iso: Packer Template Build -- Complete
==> vsphere-iso: Shutting down VM...
==> vsphere-iso: Deleting Floppy drives...
==> vsphere-iso: Deleting Floppy image...
==> vsphere-iso: Eject CD-ROM drives...
==> vsphere-iso: Convert VM into template...
Build 'vsphere-iso' finished after 32 minutes 43 seconds.

==> Wait completed after 32 minutes 43 seconds

==> Builds finished. The artifacts of successful builds are:
--> vsphere-iso: Ubuntu-Packer

```