---
title: Installing CakePHP on Mac OS X running XAMPP
date: 2017-06-16 01:21:00 +02:00
permalink: "/php/cakephp/installing-cakephp-on-mac-os-x-running-xampp/"
categories:
- php
- cakephp
layout: post
published: true
---

The following Howto will install [CakePHP](http://cakephp.com) and get you up and running using [XAMPP](http:///xampp.com) on Mac OS X 10.7.5+.

This guide is a series of terminal commands that will:

 1. Install the latest version of Composer.
 2. Move the `composer phar` file to the XAMPP `bin` directory.
 3. Install the latest version of CakePHP.

## Table of Contents
<!-- MarkdownTOC -->

- [Minimum Requirements](#minimum-requirements)
- [Getting Composer](#getting-composer)
    - [Download and Build](#download-and-build)
    - [Moving things around](#moving-things-around)
- [Setting up a CakePHP project](#setting-up-a-cakephp-project)
    - [Working Directory](#working-directory)
    - [Install CakePHP](#install-cakephp)

<!-- /MarkdownTOC -->

## Minimum Requirements
I have successfully tested the installation using the following package versions:

* Mac OS X 10.7.5
* XAMPP 5.6.30-0
* Composer 1.4.1

Earlier versions of the above software may indeed work however I have not tested them so YMMV.

## Getting Composer
Open up a terminal window and run the following shell commands. This will download the composer setup file, verify the installer is valid and create the packaged `composer.phar` file.

### Download and Build
```bash
$ /Applications/XAMPP/bin/php -r "copy('https://getcomposer.org/installer', 'composer-setup.php');"
$ /Applications/XAMPP/bin/php -r "if (hash_file('SHA384', 'composer-setup.php') === '669656bab3166a7aff8a7506b8cb2d1c292f042046c5a994c43155c0be6190fa0355160742ab2e1c88d40d5be660b410') { echo 'Installer verified'; } else { echo 'Installer corrupt'; unlink('composer-setup.php'); } echo PHP_EOL;"
$ /Applications/XAMPP/bin/php composer-setup.php
$ /Applications/XAMPP/bin/php -r "unlink('composer-setup.php');"
```

### Moving things around
Once you see the `Installer Verified` message output in your terminal window you can then run the following command.
```bash
$ mv composer.phar /Applications/XAMPP/bin/composer
$ alias composer='/Applications/XAMPP/bin/composer/composer.phar'
```
This command moves the `composer.phar` file to the XAMPP `bin` directory so that you can access it from a centralised location within XAMPP. The `alias` command makes composer available globally. It is useful to move it as you may want or need to install other packages via Composer.

You will now be able to run composer at the following location:
```bash
$ /Applications/XAMPP/bin/composer/composer.phar
```
or simply run:
```bash
$ composer
```

## Setting up a CakePHP project
By default Composer will install the most current version of CakePHP that works with the PHP version in XAMPP. I did not want to run PHP 7 so I chose to download XAMPP using PHP version 5.6.30. Running PHP 5 or 7 really won't matter to installing CakePHP; either version will work.

### Working Directory
XAMPP serves website files from the `htdocs` folder so you will need to ensure that you are in the correct path before installing CakePHP.

In the terminal window run the following command to navigate to the `htdocs` folder.
```bash
$ cd /Applications/XAMPP/xamppfiles/htdocs
```

### Install CakePHP
Finally we are now ready to install CakePHP. You can do this by running the following command in your terminal window.
```bash
$ /Applications/XAMPP/bin/php composer create-project --prefer-dist helloworld/app helloworld
```
**Very Important:** In the above command you need to replace *helloworld* with the name of the project you are setting up.

Once the command has run you will now have a new folder in the `htdocs` folder. You can now fire-up the site in your browser.
