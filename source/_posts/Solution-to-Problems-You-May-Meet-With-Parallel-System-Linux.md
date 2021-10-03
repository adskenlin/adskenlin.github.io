---
title: Solution to Problems You May Meet With Parallel System Ubuntu
date: 2021-08-15 03:34:13
tags: 
- Ubuntu
---
This article collects the problems I met while I was installing Ubuntu and using Ubuntu at the beginning.
## Normal Steps to Install Linux(Ubuntu)
+ Download Ubuntu from [official website](https://ubuntu.com/download/desktop).
+ Burn the .msi to your U-Disk with [Rufus](http://rufus.ie/en/).
+ Prapare the Disk partition for Ubuntu System:
Windows' Disk Management -> Right Click on your available Disk -> Shrink Volume... -> Set the space -> Shrink
+ Turn off Win10 Quick Start
+ Restart the PC, stick your U-Disk, go to BIOS Setting
+ Choose "UEFI: USB Flash Disk" in start menu
+ Choose "Install Ubuntu"
+ Finish Installation and restart
+ If your PC automatically use Windows, go to BIOS Setting again and choose ubuntu to start

## Problems
+ When you switch your system, the system time will be incorrect:

```bash
// Solution: Change UTC=yes to UTC=no in Linux 
// type in cmd:
timedatectl set-local-rtc 1 --adjust-system-clock
```

+ Installation freezes at the start and show checking nvidia driver:

```txt
Solution: 
1. when you see "install Ubuntu" etc.. (GRUB) press "e" quickly! 
2. Edit: Replace "quiet splash" to "nomodeset" and press F10 to boot.
3. After installation done, and reboot.
4. At the same place(GRUB), press "e" and in the line that starts with "linux", add "nouveau.modeset=0" at the end of that line. 
5. When you successfully enter Ubuntu, use Software and Update to install Nvidia Driver.
```

+ Connect Window's Disk in Ubuntu:
```txt
Solution:
1. with `sudo fdisk -l` you could get an overview of all disk partitions.
2. mark the name of device used by your windows disk, e.g. /dev/sda2.
3. `sudo gedit /etc/fstab`
4. add a line at the end with e.g.
`/dev/sda2  /media/Data`
5. the device would be able to visited through `/media/Data` in your ubuntu system
```
