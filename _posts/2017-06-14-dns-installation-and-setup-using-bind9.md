---
title: DNS Installation and Setup using BIND9
date: 2017-06-14 18:45:00 +02:00
permalink: "/debian/mail-server/dns-installationand-setup-using-bind9/"
categories:
  mail-server
  debian
layout: post
published: true
---

This HOWTO will assist you in getting a Domain Name Server (DNS) up and running using BIND9 on Debian Etch.

## Table of Contents
<!-- MarkdownTOC -->

- [Page Migration](#page-migration)
- [Introduction](#introduction)
- [Requirements](#requirements)
- [Pre-Installation](#pre-installation)
- [Installing lsb-base and BIND9](#installing-lsb-base-and-bind9)
- [Configure the Master DNS Server](#configure-the-master-dns-server)
- [Setting up the example.com domain](#setting-up-the-examplecom-domain)
  - [Creating the zone files](#creating-the-zone-files)
  - [Making sure all is OK](#making-sure-all-is-ok)
  - [Adding zone files to BIND9](#adding-zone-files-to-bind9)
  - [Testing](#testing)
- [Troubleshooting](#troubleshooting)
- [References](#references)

<!-- /MarkdownTOC -->

## Page Migration
This page first appeared on the [original Debian Wiki][history] which was created over a decade ago.

 - Originally Published: 15 January 2007
 - Migrated: 14 June 2017
 - Last Updated: 14 June 2017

---

## Introduction
This HOWTO will assist you in getting a Domain Name Server (DNS) up and running using BIND9 on Debian Etch. When setting up a DNS server it is common practise to use two separate DNS servers for a domain as you are required to have at least two DNS servers running for DNS to work correctly. If one breaks, the other can continue to serve your domain.

However, when I setup my DNS system I did not have the resources on hand to use two different servers for DNS so the setup below will configure one server to run both nameservers. It's not an ideal solution and is definitely not a best-practise solution but one can only work with what you have.

In this HOWTO I will use the fictional domain `example.com`. The nameservers will use `192.168.254.1` and `192.168.254.2` as their IP addresses. Both the domain and namerserver IPs need to be changed to reflect your server.

## Requirements
- A Debian Etch base installation - [Installation HOWTO here][debian-etch-install].
- At least two static IP addresses that you can use to setup the nameserver information.
- Root access to your server.

## Pre-Installation
Before proceeding to install, update the necessary packages in Debian with this command.
```bash
apt-get update
apt-get upgrade
```
## Installing lsb-base and BIND9
To continue we need some Debian building tools since we have to download source packages:
```bash
apt-get install devscripts
```
BIND9 depends on lsb-base from testing. Lets grab it: (Syntax explanation: the `-y` tells apt to say yes to all questions, `build-dep` installs all packages required for `-testing_packageX-` from the Etch repository and with `-b` the sour
```bash
mkdir /usr/local/lsb-base/
cd /usr/local/lsb-base/
apt-get -y build-dep lsb-base
apt-get source lsb-base -b
dpkg-i lsb-base*.deb
```
Next is BIND9:
```bash
mkdir /usr/local/bind9
cd /usr/local/bind9
apt-get -y build-dep bind9
apt-get source bind9 -b
dpkg -i *.deb
```
## Configure the Master DNS Server
First we need to stop BIND9:
```bash
/etc/init.d/bind9 stop
```
In order to chroot bind we need to set an option in `/etc/default/bind9`.
Locate this in `/etc/default/bind9`:
```bash
OPTIONS="-u bind"
```
Replace it with this:
```bash
OPTIONS="-u bind -t /var/lib/named"
```
It will now run as user 'bind' chrooted in `/var/lib/named`.
These steps are required for the chroot jail:
```bash
mkdir -p /var/lib/named/etc
mkdir /var/lib/named/dev
mkdir -p /var/lib/named/var/cache/bind
mkdir -p /var/lib/named/var/run/bind/run
mv /etc/bind /var/lib/named/etc
ln -s /var/lib/named/etc/bind /etc/bind
mknod /var/lib/named/dev/null c 1 3
mknod /var/lib/named/dev/random c 1 8
chmod 666 /var/lib/named/dev/*
chown -R bind:bind /var/lib/named/var/*
chown -R bind:bind /var/lib/named/etc/bind
```
Bind now has its own dir with space for .pid files and config files. In order to keep things clear we made a symlink back to /etc/.
Now edit `/etc/init.d/sysklogd` to allow logging of bind activity. Replace this:
```bash
SYSLOGD=""
```
With this:
```bash
SYSLOGD="-a /var/lib/named/dev/log"
```
Now restart `sysklogd` and BIND9:
```bash
/etc/init.d/sysklogd restart
/etc/init.d/bind9 start
```
And test:
```bash
ping www.google.com
```
If you get a reply, then your DNS master server is working and ready to use. We will now complete and use the example.com domain with our new master server.

## Setting up the example.com domain
The new master DNS server is currently just forwarding requests to the server of your ISP. So, we will now install and configure our own domain and let our new server handle all request regarding that domain.

`example.com` has been chosen for illustrative purposes as per the [RFC 2606][rfc-link] - see this [Wikipedia Example.com article][wikipedia-article] for more information.

### Creating the zone files
Lets start with creating the directory where we will store the zone file. This file contains all info about the domain.
```bash
mkdir /etc/bind/zones/master/
```
Next we will create the zones file:
```bash
vim /etc/bind/zones/master/example.com.db
```
Add the following (obviously replacing `example.com` and `192.168.254.1` with your own details):

    ;
    ; BIND data file for example.com
    ;
    $TTL    604800
    @       IN      SOA     ns1.example.com. info.example.com. (
                                2007011501         ; Serial
                                      7200         ; Refresh
                                       120         ; Retry
                                   2419200         ; Expire
                                    604800)        ; Default TTL
    ;
    @       IN      NS      ns1.example.com.
    @       IN      NS      ns2.example.com.
    example.com.    IN      MX      10      mail.example.com.
    example.com.    IN      A       192.168.254.1
    ns1                     IN      A       192.168.254.1
    ns2                     IN      A       192.168.254.2
    www                     IN      CNAME   example.com.
    mail                    IN      A       192.168.254.1
    ftp                     IN      CNAME   example.com.
    example.com.            IN      TXT     "v=spf1 ip4:192.168.254.1 a mx ~all"
    mail                    IN      TXT     "v=spf1 a -all"

Here we have created a DNS zone file with both nameservers as well as records for the mail and ftp server for the domain `example.com`. Trying to go into more detail about what each item reflects above is beyond the scope of this HOWTO and you should do your own research into what each item means.

In South Africa registering domain names with the .co.za extension requires that Reverse DNS (RDNS) is setup correctly. Other TLD's don't necessarily require RDNS but either way it's good practise to setup RDNS for your DNS server so we'll do so now.

Create a new file called `192.168.254.rev` which follows the convention of the first three IP ranges in your IP address.
```bash
vim /etc/bind/zones/master/192.168.254.rev
```
Add the following:
```bash
$TTL 1d ;
$ORIGIN 254.168.192.IN-ADDR.ARPA.
@       IN      SOA     ns1.example.com.   info.example.com. (
                                       2007011501
                                       7200
                                       120
                                       2419200
                                       604800
)
        IN      NS      ns1.example.com.
        IN      NS      ns2.example.com.
1       IN      PTR     ns1.example.com.
2       IN      PTR     ns2.example.com.
```
The reverse lookup files are almost identical to the domain zone files with only minor changes. The first section of this file is exactly the same as the first section of the domain zone file. The bottom section is where it is different. This time we are listing the last part of the IP address first and then the hostname last.

There are two things you must notice here. You have to use the fully qualified domain name here and you must put a `.` at the end of it. These 2 things are important to the file and weird things will happen if you don't do it this way.

You must also change the `$ORIGIN` section at the top of the RDNS file to reflect the reverse IP address of your server. In this example our IP address ranges are `192.168.254.1/2` and the reverse of this would be `254.168.192.IN-ADDR.ARPA`. In the PTR records at the bottom we assign the final IP range to reflect our two nameservers - i.e. 1 & 2.

### Making sure all is OK
Now that we've created both zone and reverse files we need to check that our main zone file is good to go. BIND9 breaks very easily so it's best to run this check before committing your changes.
```bash
cd /etc/bind/zones/master/
named-checkzone example.com example.com.db
```
You should get an OK status when doing this. If not you need to double-check your zone file and make changes until you get an OK status.

### Adding zone files to BIND9
We now need to add the zone file data to the `named.conf.local` file:
```bash
vim /etc/bind/named.conf.local
And add the following to the file:
zone "example.com" {
       type master;
       file "/etc/bind/zones/master/example.com.db";
};

zone "254.168.192.IN-ADDR.ARPA" {
       type master;
       file "/etc/bind/zones/master/192.168.254.rev";
};
```
### Testing
We can now restart bind and check if it works:
```bash
/etc/init.d/bind9 restart
ping ns1.example.com
```
This should bring bring up a ping result resolving to `192.168.254.1`

Try another test:
```bash
nslookup
ns1.example.com
```
Should give you `192.168.254.1`

Finally run this one:
```bash
dig @localhost example.com
```
If all is OK then you'll be presented with the zone file information.
At this stage you now have a working and usable DNS server.

## Troubleshooting
If you're wondering why updates to the zone file on your master seem to fail, check the serial number inside the zone file. Each time you make a change to the zone file you will need to increase the Serial number in the zone file to ensure that your latest changes are updated.

The serial is setup and structured as follows:

    2007011501 = (2007)(01)(15)(01)

- First 4 digits of the serial indicate the year - i.e. 2007
- Next 2 digits of the serial indicate the month - i.e. 01 (January)
- Next 2 digits of the serial indicate the date - i.e. 15
- The final 2 digits of the serial indicate the revision number for that day - i.e. 01

If you are updating your Serial number but your changes are not being reflected I recommend that you reload your BIND data by executing the following command in a Linux shell:
```bash
rndc reload
```
If you are running BIND on two different servers you will need to install `ntpdate` on both servers to ensure that zone transfers happen correctly. Both master and slave servers need to have the exact same time setting for zone transfers to take place:
```bash
apt-get -y install ntpdate
```

## References
 - [Debian GNU/Linux Network Administrator's Manual - Chapter 8 DNS/BIND][ref1]
 - [Debian Sarge Installing A Bind9 Master/Slave DNS System][ref2] - by harm
 - [Two-in-one DNS server with BIND9][ref3] - by pupeno
 - [Building A Debian DNS System][ref4] - by joe

[history]: /howto-history/
[debian-etch-install]: /debian/installing-a-debian-etch-base-system/
[rfc-link]: http://tools.ietf.org/html/rfc2606
[wikipedia-article]: http://en.wikipedia.org/wiki/Example.com
[ref1]: http://www.debian.org/doc/manuals/network-administrator/ch-bind.html
[ref2]: http://www.howtoforge.com/debian_bind9_master_slave_system
[ref3]: http://www.howtoforge.com/two_in_one_dns_bind9_views
[ref4]: http://www.howtoforge.com/debian_dns