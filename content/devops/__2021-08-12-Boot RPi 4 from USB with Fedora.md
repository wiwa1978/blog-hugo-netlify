---
title: Boot Raspberry Pi 4 from USB with Fedora
date: 2021-08-12T14:39:50+01:00
draft: True
categories:
  - DevOps
  - Private Cloud
  - All
tags:
  - KVM
  - Fedora
---

Inspired by [this](https://budai.cz/posts/2021-03-07-booting-fedora-server-from-usb-on-raspberry-pi-4/) post

### Introduction

In the previous guide, we have ensured we can boot our Raspberry Pi from a USB disk. We ended up with installing Raspbian OS onto the USB drive. However, since the ultimate goal of all this is to run virtual machines through KVM, we will install Fedora onto the USB drive.

### Download Fedora

Download Fedora from this website.

![USB_RPi](/images/2021-08-12-1.png)

The file you will get is a raw file, with extension .aarch64.raw.xz. In my case, filename is `Fedora-Workstation-35-1.2.aarch64.raw.xz`.

### Copy Fedora to the USB drive

Find the device name for your USB drive. You can use `fdisk -l` for that. In my case, you will see that the USB drive is `/dev/sdb` disk.

![USB_RPi](/images/2021-08-12-2.png)

Remember that our USB disk contains two partitions: one is the bootloader (/dev/sdb1), the other is the Raspbian OS (/dev/sdb2).

Next, copy the Fedora image to the USB drive

```bash
wim@ubuntu-nuc:~/Downloads$ xzcat Fedora-Workstation-35-1.2.aarch64.raw.xz | sudo dd status=progress bs=4M of=/dev/sdb
```

This will take a couple of minutes.

```bash
wim@ubuntu-nuc:~/Downloads$ xzcat Fedora-Workstation-35-1.2.aarch64.raw.xz | sudo dd status=progress bs=4M of=/dev/sdb
12837330944 bytes (13 GB, 12 GiB) copied, 167 s, 76,9 MB/s
0+1560603 records in
0+1560603 records out
12884901888 bytes (13 GB, 12 GiB) copied, 175,643 s, 73,4 MB/s
```

### Change the bootloader

When you insert the USB disk into the Raspberry Pi, you will notice that it won't boot. So we need to change the bootloader.
First, download the UEFI firmware from [here](https://github.com/pftf/RPi4/releases).

```
wim@ubuntu-nuc:/home/wim/Downloads# sudo mkdir /mnt/rpi
wim@ubuntu-nuc:/home/wim/Downloads# sudo mount /dev/sdb1 /mnt/rpi
wim@ubuntu-nuc:/home/wim/Downloads# cd /mnt/rpi
wim@ubuntu-nuc:/mnt/rpi# ls
bcm2709-rpi-2-b.dtb       bootcode.bin  fixup.dat        start4.elf
bcm2710-rpi-2-b.dtb       config.txt    fixup_db.dat     start4x.elf
bcm2710-rpi-3-b.dtb       EFI           fixup_x.dat      start_cd.elf
bcm2710-rpi-3-b-plus.dtb  fixup4cd.dat  overlays         start_db.elf
bcm2710-rpi-cm3.dtb       fixup4.dat    rpi3-u-boot.bin  start.elf
bcm2711-rpi-400.dtb       fixup4db.dat  rpi4-u-boot.bin  start_x.elf
bcm2711-rpi-4-b.dtb       fixup4x.dat   start4cd.elf
bcm2711-rpi-cm4.dtb       fixup_cd.dat  start4db.elf
```

Next, unzip the content in the folder `/mnt/rpi`.

```bash
root@ubuntu-nuc:/home/wim/Downloads# mkdir /mnt/rpi
root@ubuntu-nuc:/home/wim/Downloads# mount /dev/sdb1 /mnt/rpi
root@ubuntu-nuc:/home/wim/Downloads# cd /mnt/rpi
root@ubuntu-nuc:/mnt/rpi# ls
bcm2709-rpi-2-b.dtb       bootcode.bin  fixup.dat        start4.elf
bcm2710-rpi-2-b.dtb       config.txt    fixup_db.dat     start4x.elf
bcm2710-rpi-3-b.dtb       EFI           fixup_x.dat      start_cd.elf
bcm2710-rpi-3-b-plus.dtb  fixup4cd.dat  overlays         start_db.elf
bcm2710-rpi-cm3.dtb       fixup4.dat    rpi3-u-boot.bin  start.elf
bcm2711-rpi-400.dtb       fixup4db.dat  rpi4-u-boot.bin  start_x.elf
bcm2711-rpi-4-b.dtb       fixup4x.dat   start4cd.elf
bcm2711-rpi-cm4.dtb       fixup_cd.dat  start4db.elf
```

When you insert the USB disk into the Raspberry Pi, you will notice that it nicely boots into Fedora OS.

### Configure Remote desktop

[wim@fedora ~]$ sudo dnf install xrdp -y
[wim@fedora ~]$ sudo systemctl enable xrdp
Created symlink /etc/systemd/system/multi-user.target.wants/xrdp.service → /usr/lib/systemd/system/xrdp.service.
[wim@fedora ~]$ sudo systemctl start xrdp
[wim@fedora ~]$ sudo firewall-cmd --permanent --add-port=3389/tcp
Warning: ALREADY_ENABLED: 3389:tcp
success
[wim@fedora ~]$ sudo firewall-cmd --reload
success
[wim@fedora ~]$ sudo chcon --type=bin_t /usr/sbin/xrdp
[wim@fedora ~]$ sudo chcon --type=bin_t /usr/sbin/xrdp-sesman
[wim@fedora ~]$ sudo reboot

### Extend the filesystem

You will notice that not all space is allocated to the partition. We can change this in multiple ways but I generally like to use `GParted` for that.

![USB_RPi](/images/2021-08-12-3.png)

![USB_RPi](/images/2021-08-12-4.png)
![USB_RPi](/images/2021-08-12-5.png)
![USB_RPi](/images/2021-08-12-6.png)
![USB_RPi](/images/2021-08-12-7.png)

```bash
[wim@fedora ~]$ df -h
Filesystem      Size  Used Avail Use% Mounted on
devtmpfs        1.4G     0  1.4G   0% /dev
tmpfs           1.5G     0  1.5G   0% /dev/shm
tmpfs           575M  8.8M  567M   2% /run
/dev/sda3       465G  4.3G  460G   1% /
tmpfs           1.5G   88K  1.5G   1% /tmp
/dev/sda3       465G  4.3G  460G   1% /home
/dev/sda2       974M  114M  793M  13% /boot
/dev/sda1       599M   33M  567M   6% /boot/efi
tmpfs           288M   56K  288M   1% /run/user/42
tmpfs           288M  144K  288M   1% /run/user/1000

```
