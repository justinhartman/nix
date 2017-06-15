---
title: Installing a second hard drive
date: 2017-06-14 20:53:00 +02:00
layout: post
published: true
categories:
    debian
    server
permalink: "/debian/server/installing-a-second-hard-drive/"
---

<!-- MarkdownTOC -->

- [Page Migration](#page-migration)
- [Introduction](#introduction)
- [Assumptions](#assumptions)
- [Installation](#installation)
- [Partitioning your new drive](#partitioning-your-new-drive)
- [Format the new disk](#format-the-new-disk)
- [Mount your new drive](#mount-your-new-drive)
- [Adding to fstab](#adding-to-fstab)

<!-- /MarkdownTOC -->

## Page Migration
This page first appeared on the [original Debian Wiki][history] which was created over a decade ago.

 - Originally Published: 29 January 2007
 - Migrated: 14 June 2017
 - Last Updated: 14 June 2017

---

## Introduction
Installing a second hard drive is easy if you know how. I needed to do this on my server and below is the steps I followed.

## Assumptions
- I assume that you have installed the hard drive in your PC, and
- That you have rebooted your machine
- In this example our primary hard drive is located at `/dev/hda` and our second, new hard drive is located at `/dev/hdc` (replace according to your system)

## Installation
The first step is to check to see that your new hard drive is installed and picked up by Debian. You do this by running the following command in a shell:
```bash
fdisk -l
```
This will give you an output that should look something like this:
```bash
Disk /dev/hda: 80.0 GB, 80026361856 bytes
255 heads, 63 sectors/track, 9729 cylinders
Units = cylinders of 16065 * 512 = 8225280 bytes

   Device Boot      Start         End      Blocks   Id  System
/dev/hda1   *           1        9354    75135973   83  Linux
/dev/hda2            9355        9729     3012187    5  Extended
/dev/hda5            9355        9729     3012156   82  Linux swap / Solaris

Disk /dev/hdc: 500.1 GB, 500107862016 bytes
255 heads, 63 sectors/track, 60801 cylinders
Units = cylinders of 16065 * 512 = 8225280 bytes
```
You can see that the 500GB disk at `/dev/hdc` is being picked up but there are no partitions yet for this drive.

## Partitioning your new drive
Next up we need to partition our drive with the following:
```bash
cfdisk /dev/hdc
```
The cfdisk controller will load up and here you can create a new partition on your drive. From the menus at the bottom I selected the following:
```bash
1. New >> Primary >> Size in MB
2. Once done select Write
3. Select Quit
Your new partition has been created at /dev/hdc1
```

## Format the new disk
Now that we have a new partition at `/dev/hdc1` we need to format it for usage by the system. From a Linux shell type:
```bash
mkfs.ext3 /dev/hdc1
```
This will now format our partition with the ext3 filesystem which should work fine for your Debian system.

## Mount your new drive
Now that we have partitioned the drive and formatted it we can now mount the drive to begin using it. From a shell run:
```bash
mkdir /new-disk
mount -t ext3 /dev/hdc1 /new-disk
```
The above commands create a new directory for the drive to be mounted in and then we mount the drive to this directory. To check that the drive has been mounted run the following:
```bash
ls -lsa /new-disk
```
You should see the following:
```bash
root@debian:/# ls -lsa /new-disk
total 24
 4 drwxrwxrwt  3 root root  4096 2007-01-29 01:57 .
 4 drwxr-xr-x 22 root root  4096 2007-01-29 01:58 ..
16 drwx------  2 root root 16384 2007-01-29 01:57 lost found
```

## Adding to fstab
Everything is now up and running however we need to add our new drive to `/etc/fstab` so that it will be mounted automatically when we reboot the machine.

First let's edit fstab:
```bash
vim /etc/fstab
```
At the end of the file add the following line:
```bash
/dev/hdc1       /new-disk            ext3    defaults,errors=remount-ro 0       1 
```
Save the file and you're done.


[history]: /howto-history/