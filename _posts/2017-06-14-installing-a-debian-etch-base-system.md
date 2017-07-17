---
title: Installing a Debian Etch base system
date: 2017-06-14 18:16:00 +02:00
permalink: "/debian/installing-a-debian-etch-base-system/"
categories:
- debian
layout: post
published: false
todo: link postfix and courier installations
---

The following is a mini HOWTO on setting up a base Debian Etch system.

## Table of Contents
<!-- MarkdownTOC -->

- [Page Migration](#page-migration)
    - [Issues](#issues)
- [Introduction](#introduction)
- [Requirements](#requirements)
- [Installing the base system](#installing-the-base-system)
- [References](#references)

<!-- /MarkdownTOC -->

## Page Migration
This page first appeared on the [original Debian Wiki][history] which was created over a decade ago.

 - Originally Published: 5 February 2007
 - Migrated: 14 June 2017
 - Last Updated: 14 June 2017

### Issues
In the [Howto History][history] page it is important to note that this particular Howto has been migrated from a legacy online backup. The original Howto used a total of 30 screenshots to demonstrate, visually, how to go about installing a Debian Etch base system. 

Sadly, I have no access to any of the original screenshots and I fear this page may now be defunct. 

That said, I've marked where the images used to be with `Image:Filename.gif` and left the original text instructions in tact. The text instructions remain valid however the screenshot that supported it no longer renders. If I ever am able to find these screenshots I will update this page accordingly. 

---

## Introduction
I used this installation procedure when setting up my Etch server and this installation serves as the foundation for all other Debian Etch Server HOWTOs contained in this wiki. There are other HOWTOs in the wiki that detail setting up various other components for the server however this one only covers the Debian base installation.

## Requirements
As a starting point you'll need to download and get the latest Debian Etch version from the Debian website. Bandwidth in South Africa is very expensive so I decided to download the [Network Install version][network-install] as the total download size is only around 150MB. 

The Network Install version provides only a small portion of Debian required to start the installation process and then you can install whatever else you want from within the installation program via the Internet. The network install is perfect for setting up a Debian Server as you want a light-weight system without any unnecessary stuff.

This is the [particular version of Etch][etch-download] that I downloaded.
You will also need a dedicated Internet connection as certain items will be downloaded from the Internet during install time.

## Installing the base system
Insert your Etch Network Install CD into your system and boot from it. The installation starts, and first you have to choose your language:
`Image:Etch1.gif`
Select your country:
`Image:Etch2.gif`
Choose a keyboard layout:
`Image:Etch3.gif`
The hardware detection starts:
`Image:Etch4.gif`
Enter the hostname of your system. In this example, the system will be called server1.example.com, so enter server1:
`Image:Etch5.gif`
Enter your domain name. In this example, this is example.com:
`Image:Etch6.gif`
Now you have to partition your hard disk. I will create one big partition (with the mount point /) and a little swap partition:
`Image:Etch7.gif`
`Image:Etch8.gif`
`Image:Etch9.gif`
`Image:Etch10.gif`
Now the base system is being installed:
`Image:Etch11.gif`
Install the GRUB boot loader to the master boot record:
`Image:Etch12.gif`
Afterwards remove the Etch CD from your system and reboot it:
`Image:Etch13.gif`
Configure your time zone:
`Image:Etch14.gif`
Enter a password for root:
`Image:Etch16.gif`
Create a second user admin:
`Image:Etch17.gif`
`Image:Etch18.gif`
`Image:Etch19.gif`
Choose your installation method. It's best to select http here:
`Image:Etch20.gif`
Select a mirror for your installation. Depending on where your server is hosted chose a location that is closest to the server:
`Image:Etch21.gif`
`Image:Etch22.gif`
Enter a proxy for the installation (if necessary). Normally you can leave this field empty.
`Image:Etch23.gif`
Under Debian software selection only choose Mail server. In the other wiki HOWTOs I go into detail on installing all the other services manually after this base installation.
`Image:Etch24.gif`
The network installation starts:
`Image:Etch25.gif`
Continue installing libc-client without Maildir support. If you want to use Maildir you can install Courier-POP3/Courier-IMAP later on in this [wiki HOWTO][postfix-url].
`Image:Etch29.gif`
Do not configure Exim at this stage unless you actually plan to use Exim as a Mail server. Instead there is a [wiki HOWTO][postfix-url] on setting up Postfix as the preferred Mail server which is why I left Exim un-configured here. I recommend Postfix as it's easy to use and set-up.
`Image:Etch26.gif`
`Image:Etch27.gif`
`Image:Etch28.gif`
Congratulations! Your base system is now finished installing:
`Image:Etch30.gif`

## References
[The Perfect Setup - Debian Sarge (3.1)][falko-setup] by Falko Timme

[network-install]: http://www.debian.com/distrib/netinst
[etch-download]: http://cdimage.debian.org/cdimage/etch_di_rc1/i386/iso-cd/debian-testing-i386-netinst.iso
[history]: /howto-history/
[falko-setup]: http://www.howtoforge.com/perfect_setup_debian_sarge
[postfix-url]: //