*Starting at version 6.2.1, Pydio Linux Repositories are moved from http://dl.ajaxplorer.info to https://download.pydio.com/linux/[package_type]. This document describes the necessary steps to upgrade existing 6.0.8 installations to the new repositories.*

### Supported version and OS
Updating to Pydio 6.2.1 is only supported from a 6.0.8 version.
First make sure that you are using the last 6.0.8 version updated from our old repositories.
The new repositories currently support the following architecture: CentOS7 / RHEL7 and Debian 8. More will come.

### Install new repositories

First step is to configure the access to the new repositories. For Debian, you will have to install our new Public Key and manually update the /etc/sources.list file. For CentOS/RHEL, you can download a "release" package that will install the repository. As a side-note, CentOS/RHEL will require the installation of the EPEL repository.

Please follow the procedure described in our Administrator Guide: https://pydio.com/en/docs/v6/install-pydio

### Updating Pydio

Once the repositories are properly configured, just input this command to your terminal:
> `sudo yum install pydio-all`
or
> `sudo apt-get install pydio-all`

Use the meta-package "pydio-all" to make sure you don't miss some plugins that your using after the upgrade. 
The new packages are structured as follow: 
- pydio-core: minimal Pydio Community installation
- pydio-plugin-<name>: plugin for Pydio Community version
- pydio:  Pydio install with officials plugins
- pydio-all: install Pydio Community with all plugins

### Updating the database

Yum/Apt-get install command does not touch your database. As some modifications were introduced for v6.2.0, you will have to manually upgrade the DB.  
Depending on your DB type (MySQL / PostGreSql / Sqlite), you may have to use different command line tool to apply the SQL scripts.
Files included in /usr/share/doc/pydio/sql

This step is important, otherwise you will have some errors in the interface.

You should be all set!

