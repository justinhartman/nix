---
title: PureFTPd Installation and Setup
date: 2017-06-14 20:53:00 +02:00
permalink: "/debian/lamp/pureftpd-installation-and-setup/"
categories:
- debian
- lamp
layout: post
---

The following HOWTO is aimed at installing and setting up a FTP server, based on Pure-FTPd, which supports multiple Virtual Hosts.

## Table of Contents
<!-- MarkdownTOC -->

- [Page Migration](#page-migration)
- [Requirements](#requirements)
- [Pre-Installation](#pre-installation)
- [Install Pure-FTPd](#install-pure-ftpd)
    - [Create a new user](#create-a-new-user)
    - [User Information](#user-information)
    - [Resetting a password](#resetting-a-password)
- [Starting the FTP Server](#starting-the-ftp-server)
- [Configuring Pure-FTPd](#configuring-pure-ftpd)
- [Restart Pure-FTPd:](#restart-pure-ftpd)
- [References](#references)

<!-- /MarkdownTOC -->

## Page Migration
This page first appeared on the [original Debian Wiki][history] which was created over a decade ago.

 - Originally Published: 15 January 2007
 - Migrated: 14 June 2017
 - Last Updated: 14 June 2017

---

## Requirements

 - A Debian Etch base installation - [Installation HOWTO here][debian-etch-install].
 - Root access to your server.

## Pre-Installation

Before proceeding to install, update the necessary packages in Debian with these commands.
```bash
apt-get update
apt-get upgrade
```

## Install Pure-FTPd
In a Linux shell run the following:
```bash
apt-get install pure-ftpd-common pure-ftpd
```
Now we need to create a new system group for pureftpd:
```bash
groupadd ftpgroup
```
Now we add a user for the group and give that user no permission to a home directory or a shell:
```bash
useradd -g ftpgroup -d /dev/null -s /etc ftpuser
```
### Create a new user
Lets create our first FTP user. In this example our user will be `justin`:
```bash
pure-pw useradd justin -u ftpuser -g ftpgroup -d /home/pubftp/justin -N 10
```
In the above command we gave him a limit of 10 MB disk space with the option `-N 10`. Now you have to enter justin's new password twice.

By default your users will be saved in `/etc/pure-ftpd/pureftpd.passwd`, but first we have to update the pureftpd Database:
```bash
pure-pw mkdb
```
The _Database_ here is simply a binary file but it is ordered and has an index for quick access.

### User Information
To get some user details enter the following to get a complete list of all pureftpd users:
```bash
pure-pw list
```
If you want to show information about a specific user:
```bash
pure-pw show justin
```
This will show you detailed information about the user `justin`.
You will notice that the line `Directory: /home/pubftp/justin/./` has a trailing `./` but you shouldn't worry as this is simply the chroot for the user, which means he can't go _above_ his directory.

### Resetting a password
If you forget the password for a user, you can reset it as follows:
```bash
pure-pw passwd justin
```
After a password reset update your database:
```bash
pure-pw mkdb
```

## Starting the FTP Server
To test the server let's start it:
```bash
/usr/sbin/pure-ftpd -S 127.0.0.1,21 -c 30 -C 1 -l puredb:/etc/pureftpd.pdb -x -E -j -R
```
The shell will open up a new `pure-ftpd` session and you should be able to connect to your FTP server. Use an FTP client to test whether or not you are able to login with your user details you created.
Once you are happy close the session off:
```bash
ctrl   z
```

## Configuring Pure-FTPd
Right so by now you have created a user and been able to connect to your FTP server. We now want to setup a few things so that we can run Pure-FTPd as a daemon.

First you need to set Pure-FTPd as a standalone server:
```bash
vim /etc/default/pure-ftpd-common
```
Replace this:
```bash
STANDALONE_OR_INETD=inetd
```
With this:
```bash
STANDALONE_OR_INETD=standalone
```
Now we want to ensure that the standalone server checks our user-names and passwords against the `pureftpd` database file:
```bash
cd /etc/pure-ftpd/conf
vim PureDB
```
Add the following to that file (if it doesn't exist):
```bash
/etc/pure-ftpd/pureftpd.pdb
```
Now we need to create a symbolic link to the PureDB file:
```bash
cd /etc/pure-ftpd/auth
ln -s /etc/pure-ftpd/conf/PureDB 50pure
ls -ls
```
You should now see a new file `50pure` linking to `../conf/PureDB`.

## Restart Pure-FTPd:
```bash
/etc/init.d/pure-ftpd restart
```
Pure-FTPd should now startup in daemon mode and everything should be up and running.

## References

 - [Howto install pureftpd on a debian machine][ref1] - by remofritzsche
 - [Pure-FTPd Website][ref2]

[debian-etch-install]: /debian/installing-a-debian-etch-base-system/
[history]: /howto-history/
[ref1]: http://www.debian-administration.org/articles/383
[ref2]: http://www.pureftpd.org/