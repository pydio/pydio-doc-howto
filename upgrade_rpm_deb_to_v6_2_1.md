## Updating Pydio

### Supported version
Updating to Pydio 6.2.1 is only supported from a 6.0.8 version.
First make sure that you are using the last 6.0.8 version updated from our old repositories (dl.ajaxplorer.info)


### Prerequisites

Configure the access to the new pydio repositories packages.
Install the package pydio-release from: https://pydio.com/en/docs/v6-enterprise/install-pydio

### Updating Pydio
Just input this command to your terminal:
> `yum install pydio-all`

### Updating the database
Once RPM package update is done, you must update the database.
Files included in /usr/share/doc/pydio/sql

New package details:
- pydio-core: minimal Pydio Community installation
- pydio-plugin-<name>: plugin for Pydio Community version
- pydio:  Pydio install with officials plugins
- pydio-all: install Pydio Community with all plugins
