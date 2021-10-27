_This guide explains how to install and configure Pydio Cells on macOS_.

Cells comes as a self-contained binary that can be directly run. The only hard requirement is a recent MySQL server. You can use either MySQL (5.7 or 8) or MariaDB 10.3+, both are available in Homebrew.

`brew install mysql` or `brew install mariadb`

## Installation

### Pydio

Download the Pydio Cells binary on your server/machine with the following command:

```sh
# You can use this url as-is: it will be resolved automatically to latest version
wget https://download.pydio.com/latest/cells/release/{latest}/darwin-amd64/cells
```

### Port 80 & 443

You can only use these ports if you are connected as an Admin User or root.

By default, Apache is running on macOS, so you need to ensure that it - or any other webservers - is not bound to these ports.

To stop the default Apache, you can use:

```sudo apachectl stop```

To prevent Apache from starting during launch, you may use:

```sudo launchctl unload -w /System/Library/LaunchDaemons/org.apache.httpd.plist```

### Database configuration

In this section, we assume you have installed MySQL server. Adapt the following steps to your current setup.

```sh
# Go to mysql mode
sudo mysql -u root
# Create new user and set password
CREATE USER 'pydio'@'localhost' IDENTIFIED BY 'your password goes here';
CREATE DATABASE cells;
GRANT ALL PRIVILEGES ON cells.* to 'pydio'@'localhost';
FLUSH PRIVILEGES;
```

## Starting with Pydio

First, give execution permission on the file for your user. For instance, you can use `chmod u+x <binary>`.

Then run the installer with the following command:

```sh
cells configure
```

Once you have finished the configuration, you can start Cells with:

```
cells start
```

By default, the server is started with a self-signed certificate on port 8080: to access the webUI browse to `https://localhost:8080` and accept the certificate.


To configure a different URL and/or port for Cells, run the following command.

```
cells configure sites
```

## Troubleshooting

- The database service might not be started, you can look at its status using : `brew services list` and then `brew services start mysql` if needed.
- You can look at the webserver's error file located in `/Users/<Your User Name>/Library/Application Support/Pydio/cells`.
