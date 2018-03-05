---
layout: default
title: Mounting LVM partitions
license: not needed
---

## Introduction

There are a number of advantages to using *Logical Volume Management (LVM)*
instead of a standard partition table when configuring your disk drives.
In particuar, the flexibility to add new drives and resize partitions,
even when the system is still running. However, the process of mounting
LVM partitions is a little more complicated. This tutorial gives a brief
introduction.

## Installing the required software

You'll need `kpartx`, which is used to create device maps from partition
tables, and `lvm2`, which is used for LVM management. You can install them
in the usual way:

   ```sh
   $ sudo apt-get install kpartx lvm2
   ```
   {: .code}

## Mounting LVM partitions from a disk image

1. First, create a loop device associated with your disk image file. This
allows the disk image file to be treated as a block device. Check that
the association has been made correctly, e.g.

     ```sh
     $ sudo losetup -f disk.img
     $ sudo losetup -a
     /dev/loop0: [2054]:7383067 (/home/cgdk2/disk.img)
     ```
     {: .code}

1. Now read the partition table from the newly created block device and
create mappings for the partitions. Check the mappings, e.g.

     ```sh
     $ sudo kpartx -a -v /dev/loop0
     add map loop0p1 (253:0): 0 997376 linear 7:0 2048
     add map loop0p2 (253:1): 0 2 linear 7:0 1001470
     add map loop0p5 (253:2): 0 15773696 linear 7:0 1001472
     $ ls -l /dev/mapper
     total 0
     crw------- 1 root root 10, 236 Jan 23 13:41 control
     lrwxrwxrwx 1 root root       7 Mar  3 11:23 loop0p1 -> ../dm-0
     lrwxrwxrwx 1 root root       7 Mar  3 11:23 loop0p2 -> ../dm-1
     lrwxrwxrwx 1 root root       7 Mar  3 11:23 loop0p5 -> ../dm-2
     ```
     {: .code}

1. Ensure that the LVM partitions are recognised as new physical volumes and
display the details:

     ```
     $ sudo pvscan
       PV /dev/mapper/loop0p5   VG ubuntu-s02-vg   lvm2 [7.52 GiB / 0    free]
       Total: 1 [7.52 GiB] / in use: 1 [7.52 GiB] / in no VG: 0 [0   ]
     $ sudo pvdisplay
       --- Physical volume ---
       PV Name               /dev/mapper/loop0p5
       VG Name               ubuntu-s02-vg
       PV Size               7.52 GiB / not usable 2.00 MiB
       Allocatable           yes (but full)
       PE Size               4.00 MiB
       Total PE              1925
       Free PE               0
       Allocated PE          1925
       PV UUID               4Wx7jU-Y1O4-efiE-HwTI-S8w5-2UWN-EP5YVT
     ```
     {: .code}

1. Next, activate the LVM volume group and check the mappings again:

     ```
     $ sudo vgchange -a y
       2 logical volume(s) in volume group "ubuntu-s02-vg" now active
     $ ls -l /dev/mapper
     total 0
     crw------- 1 root root 10, 236 Jan 23 13:41 control
     lrwxrwxrwx 1 root root       7 Mar  3 11:23 loop0p1 -> ../dm-0
     lrwxrwxrwx 1 root root       7 Mar  3 11:23 loop0p2 -> ../dm-1
     lrwxrwxrwx 1 root root       7 Mar  3 11:23 loop0p5 -> ../dm-2
     lrwxrwxrwx 1 root root       7 Mar  3 11:31 ubuntu--s02--vg-root -> ../dm-3
     lrwxrwxrwx 1 root root       7 Mar  3 11:31 ubuntu--s02--vg-swap_1 -> ../dm-4

     ```
     {: .code}

1. Now, make sure you have a mount point and mount the volume that you want,
e.g.

     ```sh
     $ mkdir mnt
     $ sudo mount -o ro,noatime /dev/mapper/ubuntu--s02--vg-root mnt
     $ ls mnt
     bin   dev  home        lib    lost+found  mnt  proc  run   snap  sys  usr  vmlinuz
     boot  etc  initrd.img  lib64  media       opt  root  sbin  srv   tmp  var
     ```
     {: .code}
 
## Cleaning up afterwards

Cleaning up is essentially undoing what you've just done, in reverse order:

   ```sh
   $ sudo umount mnt
   $ sudo vgchange -a n ubuntu-s02-vg
    0 logical volume(s) in volume group "ubuntu-s02-vg" now active
   $ sudo kpartx -d /dev/loop0
   $ sudo losetup -d /dev/loop0
   ```
   {: .code}

