---
title: Setting the time using Terminal
date: 2017-06-14 20:25:00 +02:00
permalink: "/debian/server/setting-the-time-using-terminal/"
categories:
- debian
- server
layout: post
---

This is really basic stuff but if you aren't running a GUI then it isn't as easy as opening up a Time & Date Settings control panel so this is the way to do it.

## Table of Contents
<!-- MarkdownTOC -->

- [Page Migration](#page-migration)
- [Setting the timezone](#setting-the-timezone)
- [Synchronise your clock with a NTP server](#synchronise-your-clock-with-a-ntp-server)

<!-- /MarkdownTOC -->

## Page Migration
This page first appeared on the [original Debian Wiki][history] which was created over a decade ago.

 - Originally Published: 6 March 2007
 - Migrated: 14 June 2017
 - Last Updated: 14 June 2017

---

## Setting the timezone
You first want to check what the current date and time is on your machine. Do this by running the following in a Linux shell:
```bash
date
```
This should return something like this:
```bash
Tue Mar  6 20:47:59 SAST 2007
```
You can also check the timezone you're currently using by running:
```bash
cat /etc/timezone
```
Which would return something like this:
```bash
Africa/Johannesburg
```
Now if none of this is correct you want to setup your timezone by running this command:
```bash
tzconfig
```
On Debian Lenny (5.0) you need to run this command:
```bash
dpkg-reconfigure tzdata
```
Select your continent and type in your city and hit enter. This should return the following:
```bash
Your default time zone is set to 'Africa/Johannesburg'.
Local time is now:      Tue Mar  6 20:50:15 SAST 2007.
Universal Time is now:  Tue Mar  6 18:50:15 UTC 2007.
```
## Synchronise your clock with a NTP server
NTP, the Network Time Protocol, is used to keep computer clocks accurate over the Internet. `ntpdate` is a simple NTP client which allows a system's clock to be set to match the time obtained by communicating with one or more servers.

Install the ntp client
```bash
apt-get update
apt-get install ntpdate
```
`ntpdate` will automatically run when booting your system.

[history]: /howto-history/

