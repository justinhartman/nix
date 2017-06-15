---
title: DNS Installation and Setup using BIND9
date: 2017-06-14 20:05:00 +02:00
permalink: "/debian/server/automated-backup-to-amazon-s3-with-duplicity-on-debian-etch/"
categories:
- debian
- server
layout: post
---

<!-- MarkdownTOC -->

- [Page Migration](#page-migration)
- [Introduction](#introduction)
- [Access Requirements](#access-requirements)
- [Software Requirements](#software-requirements)
- [Installation](#installation)
- [Generate a new GnuPG key](#generate-a-new-gnupg-key)
- [Create a Shell Script](#create-a-shell-script)
- [Testing if it works](#testing-if-it-works)
- [Create a cronjob](#create-a-cronjob)
- [Troubleshooting](#troubleshooting)
    - [Python Error](#python-error)
    - [Amazon S3 Bucket Names](#amazon-s3-bucket-names)

<!-- /MarkdownTOC -->

## Page Migration
This page first appeared on the [original Debian Wiki][history] which was created over a decade ago.

 - Originally Published: 5 March 2007
 - Migrated: 14 June 2017
 - Last Updated: 14 June 2017

---

## Introduction
If you've tried to use Debian Etch's version of Duplicity you'll know there is no way to use Amazon S3 servers for remote backup as Duplicity on Etch is simply not current enough. This tutorial will show you how to install the most current Duplicity and set it up to backup your Debian Etch server.

## Access Requirements
 - Amazon S3 Access Key ID
 - Amazon S3 Secret Access Key
 - GPG Key
 - GPG Passphrase

## Software Requirements
 - [Duplicity][duplicity] v0.5.02 or later
 - Python v2.3 or later
 - librsync v0.9.6 or later
 - GnuPG for encryption
 - GnuPGInterface 0.3.2 or later
 - NcFTP version 3.1.9 or later
 - [Boto][boto] 0.9d or later
 - pexpect 2.1 or later
 - Python development files
 - librsync development files

## Installation
Update your packages.
```bash
apt-get update
apt-get upgrade
```
Install the Debian Etch packages that meet the minimum requirements.
```bash
apt-get install build-essential
apt-get install librsync1 librsync-dev python-gnupginterface ncftp python-pexpect python-dev
```
Download the packages that are outdated in Debian Etch.
```bash
wget http://savannah.nongnu.org/download/duplicity/duplicity-0.5.02.tar.gz
wget http://boto.googlecode.com/files/boto-1.4c.tar.gz
```
Extract the files.
```bash
tar -zxvf duplicity-0.5.02.tar.gz
tar -zxvf boto-1.4c.tar.gz
```
Install Boto
```bash
cd boto-1.4c
python setup.py install
```
Install Duplicity
```bash
cd duplicity-0.5.02
python setup.py install
```
## Generate a new GnuPG key
Generate a new gpg key for duplicity.
```bash    
gpg --gen-key
```
You will see the following output.

    gpg (GnuPG) 1.4.6; Copyright (C) 2006 Free Software Foundation, Inc.
    This program comes with ABSOLUTELY NO WARRANTY.
    This is free software, and you are welcome to redistribute it
    under certain conditions. See the file COPYING for details.

    Please select what kind of key you want:
    (1) DSA and Elgamal (default)
    (2) DSA (sign only)
    (5) RSA (sign only)
    Your selection?

Select DSA and Elgamal.

    DSA keypair will have 1024 bits.
    ELG-E keys may be between 1024 and 4096 bits long.
    What keysize do you want? (2048)

The default of 2048 is more than enough and you could change this if you like.

    Requested keysize is 2048 bits
    Please specify how long the key should be valid.
             0 = key does not expire
          <n>  = key expires in n days
          <n>w = key expires in n weeks
          <n>m = key expires in n months
          <n>y = key expires in n years
    Key is valid for? (0)

You must leave the key to not expire so the default is correct here once again.

    Key does not expire at all
    Is this correct? (y/N)

Yes, this is correct :)

    You need a user ID to identify your key; the software constructs the user ID
    from the Real Name, Comment and Email Address in this form:

    Real name: Duplicity Backup
    Email address: duplicity@example.com
    Comment: GPG Key for Duplicity
    You selected this USER-ID:
        "Duplicity Backup (Key for Duplicity) <duplicity@example.com>"

    Change (N)ame, (C)omment, (E)mail or (O)kay/(Q)uit?

You can enter anything you like in the Real Name, Email and Comment fields and once done select O for Okay and hit enter.

    You need a Passphrase to protect your secret key.

    Enter Passphrase:
    Repeat Passphrase:

Enter a password here and make it as complex as possible. Please remember your password you type here because we will need it later.

Your key will now begin its creation process and this can take a very long time (up to 10-15 minutes). Just be patient, it will generate a key and submit the results as shown below.

    gpg: key FB0B7A37 marked as ultimately trusted
    public and secret key created and signed.

    gpg: checking the trustdb
    gpg: 3 marginal(s) needed, 1 complete(s) needed, PGP trust model
    gpg: depth: 0  valid:   2  signed:   0  trust: 0-, 0q, 0n, 0m, 0f, 2u
    pub   1024D/FB0B7A37 2008-09-22
          Key fingerprint = B5A2 14BD 368B 0E7F 44F3  7C81 2460 B782 FC0B 6A25
    uid                  Duplicity Backup <duplicity@example.com>
    sub   2048g/FB0B7A37 2008-09-22

The results above are displayed once your key has been generate. From this you will need to remember your 8 digit key. In this example our key is `FB0B7A37`. Keep this in a safe place.

## Create a Shell Script
Because we'll be automating the backup process via cronjob we need to create a shell script that can do all the hard work for us. Below is a basic example shell script that you can modify.
```bash
#!/bin/bash
# Export some ENV variables so you don't have to type anything
export AWS_ACCESS_KEY_ID=<your-amazon-s3-access-key-id>
export AWS_SECRET_ACCESS_KEY=<your-amazon-s3-secret-access-key>
export PASSPHRASE="<your-gpg-passphrase>"

GPG_KEY=<your-8-digit-gpg-key>

# The source of your backup
SOURCE=/

# Select a unique name for your bucket below.
# The bucket doesn't need to exist on Amazon
# however you should make sure it's unique to
# avoid any conflict with existing buckets from
# other Amazon S3 users.
DEST=s3+http://<your-amazon-s3-bucket-name>

duplicity \
    --encrypt-key=${GPG_KEY} \
    --sign-key=${GPG_KEY} \
    --include=/boot \
    --include=/etc \
    --include=/home \
    --include=/root \
    --include=/var/lib/mysql \
    --exclude=/** \
    ${SOURCE} ${DEST}

# Reset the ENV variables. Don't need them sitting around
export AWS_ACCESS_KEY_ID=
export AWS_SECRET_ACCESS_KEY=
export PASSPHRASE=
```
Save the file and make it executable.
```bash
chmod 700 your-filename.sh
```
## Testing if it works
Prior to creating a cronjob you should test that everything is working perfectly. You can run the following command to see if your script backs up to Amazon S3.
```bash
/bin/bash /path/to/script/your-filename.sh
```
You should see the following output if your backup was successful.
```bash
No signatures found, switching to full backup.

--------------[ Backup Statistics ]--------------
StartTime 1222126413.99 (Tue Sep 23 01:33:33 2008)
EndTime 1222126736.68 (Tue Sep 23 01:38:56 2008)
ElapsedTime 322.69 (5 minutes 22.69 seconds)
SourceFiles 54198
SourceFileSize 311617938 (297 MB)
NewFiles 54198
NewFileSize 311617938 (297 MB)
DeletedFiles 0
ChangedFiles 0
ChangedFileSize 0 (0 bytes)
ChangedDeltaSize 0 (0 bytes)
DeltaEntries 54198
RawDeltaSize 136543331 (130 MB)
TotalDestinationSizeChange 112326763 (107 MB)
Errors 0
-------------------------------------------------
```
## Create a cronjob
Assuming you had no errors when you ran the test you can now set up the cronjob to automate the process.
```bash
crontab -e
0     0       *       *       *       /bin/bash /path/to/script/your-filename.sh
```
This will run the backup every day at midnight.

## Troubleshooting

### Python Error
If you get Python errors when trying to run your backup script then I recommend that you first create a unique bucket on Amazon S3 and make sure that you upload at least one file to your new bucket. I've noticed a strange problem with Amazon S3 that prevents the script from running if your bucket doesn't already have a file in it.

### Amazon S3 Bucket Names
Another common issue is the naming of your bucket on Amazon S3. Remember that bucket names have to be unique on the entire Amazon S3 server so if another user is using the same bucket name as you're trying to create then your backup will fail. Try using a name like `amazon_bucket_1234` when naming your bucket.

[history]: /howto-history/
[duplicity]: http://duplicity.nongnu.org/
[boto]: http://code.google.com/p/boto/
