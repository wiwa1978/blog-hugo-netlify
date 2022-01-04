---
title: Boot Raspberry Pi 4 from USB
date: 2021-08-10T14:39:50+01:00
draft: True
categories:
  - DevOps
  - Private Cloud
  - All
tags:
  - KVM
  - Fedora
---

[link](https://www.tomshardware.com/how-to/boot-raspberry-pi-4-usb)

### Introduction

### Download Raspberry Pi Imager

Download Raspberry Pi images from the Raspberry Pi website
![USB_RPi](/images/2021-08-10-1.png)

### Configure the bootloader

Next, we will configure that our Raspberry Pi will boot from USB if available. If not, it will boot from the SD card.

Insert a microSD card and go to

![USB_RPi](/images/2021-08-10-2.png)

![USB_RPi](/images/2021-08-10-3.png)

![USB_RPi](/images/2021-08-10-4.png)

Click on Write to download and write a configuration image to the micro SD card. When done remove the card from your computer.

Insert the micro SD card into your Raspberry Pi 4 and power on. The green activity light will blink a steady pattern once the update has been completed. If you have an HDMI monitor attached, the screen will go green once the update is complete. Allow 10 seconds or more for the update to complete, do not remove the micro SD card until the update is complete.

### Install Raspbian OS

Insert the microSD card again and open the Raspberry Pi imager tool once again. Now, install the Raspbian OS onto the smartcard.

![USB_RPi](/images/2021-08-10-5.png)

![USB_RPi](/images/2021-08-10-6.png)

![USB_RPi](/images/2021-08-10-7.png)

Insert the SD card into the Raspberry Pi 4 and boot into the Rasbian OS system.

### Copy SD card content to USB

Next, ensure your USB drive is attached to an USB 3.0 port (blue) of your Raspberry Pi 4. On the Raspbian OS home screen, go to the `Accessories` section of the start menu and select `SD Card Copier`.

![USB_RPi](/images/2021-08-10-8.png)

![USB_RPi](/images/2021-08-10-9.png)

Finally, after couple of minutes, it will mention `Copy complete`. You now have copied the content from the SD card (containing the Raspbian OS) to the USB drive. Together with the modifiction to boot from the USB drive, we should now be able to boot our Raspberry Pi from our USB drive.

Let's go ahead and do that. Remove the SD Card from your Raspberry Pi and leave the USB drive inserted. Reboot your Rasberry Pi. You will see that the Raspbian OS loads from your USB drive.

![USB_RPi](/images/2021-08-10-10.png)
