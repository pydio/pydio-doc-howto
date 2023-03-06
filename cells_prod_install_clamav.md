This article explains how to install ClamAV software, to be used in conjunction with [Cells Flows job for antivirus scan](https://pydio.com/en/docs/cells-flows/scan-antivirus).

From the [Clam AV website](https://www.clamav.net/documents/introduction):

> Clam AntiVirus is an open source (GPLv2) anti-virus toolkit, designed especially for e-mail scanning on mail gateways. It provides a number of utilities including a flexible and scalable multi-threaded daemon, a command line scanner and advanced tool for automatic database updates. The core of the package is an anti-virus engine available in a form of shared library.

Below information are based on [the official installation guide](https://www.clamav.net/documents/installing-clamav).

### Install

Following commands will install:

- **clamav:** the antivirus and also the client to scan.
- **clamav-daemon:** installs the daemon that can be used with TCP or UNIX and allow scanning.
- **clamav-freshclam:** this package handles all the updates for the antivirus database.

```sh
sudo apt install clamav clamav-daemon clamav-freshclam
```

### Configure

After you have installed all the packages, run this command to configure `clamav-daemon`. You can keep default settings for almost everything, see details below for the parameters that have to be set explicitly.

```sh
sudo dpkg-reconfigure clamav-daemon
```

Important settings:

- Socket: choose local UNIX socket if both Cells and ClamAV run on the same machine, in such case, you should remember the path to the socket file (typically: `/var/run/clamav/clamd.ctl`). You can choose TCP otherwise, and note FQDN and port used.
- Group Owner of the socket: choose pydio (or the user that runs the `cells` process if you did not follow our recommendations during Cells install)
- Maximum stream length: if you want clamd to scan bigger files, put a larger value than the 25MB default. *Warning*: this will also consume more resources.
- Choose pydio / pydio as user & group for the daemon process

The daemon is started with systemd, at this point, the service should be started and enabled, to double check:

```sh
sudo systemctl status clamav-daemon
```