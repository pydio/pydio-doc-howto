
_This guide contains strongly opinionated choices and best practices. It will show you the steps required for a production-ready and reasonnably secured server. For a simple test on a RHEL-like server, you can skim through [this page](./quick-start) instead_.

**Usecase**

Deploy a self-contained Pydio Cells instance on a web-facing CentOS 7 server,  
exposed at `https://<your-fqdn>` using a Let's Encrypt certificate.

**Requirements**

- **CPU/Memory**: 4GB RAM, 2 CPU
- **Storage**: 100GB SSD hard drive
- **Operating System**:
  - CentOS, RHEL, Scientific Linux (6, 7, 8).  
  - An admin user with sudo rights that can connect to the server via SSH
  - _Note: The present guide uses a CentOS 7 server. You might have to adapt some commands if you use a different version or flavour._
- **Networking**:
  - One Network Interface Controller connected to the internet
  - A registered domain that points toward the public IP of your server: if you already know your IP address, it is a good idea to already add a `A Record` in your provider DNS so that the record has been already propagated when we need it.

## Installation

### Dedicated user and file system layout

We recommend to run Pydio Cells with a dedicated `pydio` user with **no sudo** permission.

As admin user on your server:

```sh
# Create pydio user with a home directory
sudo useradd -m -s /bin/bash pydio

# Create necessary folders
sudo mkdir -p /opt/pydio/bin /var/cells/certs
sudo chown -R pydio: /opt/pydio /var/cells

# Add system-wide ENV var
sudo tee -a /etc/profile.d/cells-env.sh << EOF
export CELLS_WORKING_DIR=/var/cells
export CADDYPATH=/var/cells/certs
EOF
sudo chmod 0755 /etc/profile.d/cells-env.sh
```

#### Verification

Login as user `pydio` and make sure that the environment variables are correctly set:

```sh
sysadmin@server:~$ sudo su - pydio 
pydio@server:~$ echo $CELLS_WORKING_DIR
/var/cells
pydio@server:~$ exit
```

### Database

The default MariaDB package shipped with CentOS 7 is too old, so we install MariaDB repo to get version 10.4:

```sh
# Add MariaDB 10.4 CentOS repository list
# See http://downloads.mariadb.org/mariadb/repositories/
sudo mkdir -p /etc/yum.repos.d
sudo tee /etc/yum.repos.d/MariaDB.repo << EOF
[mariadb]
name = MariaDB
baseurl = http://yum.mariadb.org/10.4/centos7-amd64
gpgkey=https://yum.mariadb.org/RPM-GPG-KEY-MariaDB
gpgcheck=1
EOF

# Install and start the server
sudo yum install MariaDB-server
sudo systemctl enable mariadb
sudo systemctl start mariadb
# Run the script to secure your install
sudo mysql_secure_installation

# Open MySQL CLI to create your database and a dedicated user
sudo mysql -u root -p
```

Start a MySQL prompt and create the database and the dedicated `pydio` user.

```mysql
CREATE DATABASE cells;
CREATE USER 'pydio'@'localhost' IDENTIFIED BY 'cells';
GRANT ALL PRIVILEGES ON cells.* to 'pydio'@'localhost';
FLUSH PRIVILEGES;
exit
```

#### Verification

Check the service is running and that the user `pydio` is correctly created:

```sh
sudo systemctl status mariadb
mysql -u pydio -p
```

### Retrieve binary

```sh
# As pydio user
sudo su - pydio 

# Download correct binary
distribId=cells 
# or for Cells Enterprise
# distribId=cells-enterprise 
wget -O /opt/pydio/bin/cells https://download.pydio.com/latest/${distribId}/release/{latest}/linux-amd64/${distribId}

# Make it executable
chmod a+x /opt/pydio/bin/cells
exit

# As sysadmin user 
# Add permissions to bind to default HTTP ports
sudo setcap 'cap_net_bind_service=+ep' /opt/pydio/bin/cells

# Declare the cells commands system wide
sudo ln -s /opt/pydio/bin/cells /usr/local/bin/cells
```

#### Verification

Call the command `version` as user `pydio`:

```sh
sudo su - pydio 
cells version
```

## Configuration

### Configure the server

Call the command `configure` as user `pydio`:

```sh
sudo su - pydio 
cells configure
```

If you choose `Browser install` at the first prompt, you can access the configuration wizard at `https://<YOUR PUBLIC IP>:8080` after accepting the self-signed certificate. (Ensure the port `8080` is free and not blocked by a firewall).

You can alternatively finalise the configuration from the command line by answering a few questions.

#### Verification

If you used the browser install, you can login in the web browser as user `admin`

First insure your firewall does not block the port 8080:

```sh
sudo firewall-cmd --add-port=8080/tcp
```

If you have done the CLI install, you first need to start the server:

```sh
sudo su - pydio 
cells start
```

Connect and login at `https://<YOUR PUBLIC IP>:8080`

**Note**:  
At this stage, we start the server in **foreground** mode. It is important that you **always stop** the server using the `CTRL + C` shortcut before calling the `start` command again.

### Declare site and generate Let's Encrypt Certificate

At this point, we assume that:

- your `A record` has been propagated: verify with `ping <YOUR_FQDN>` from your local workstation
- both port 80 and 443 are free and not blocked by any firewall `sudo netstat -tulpn`

Create a site:

```sh
sudo su - pydio 
cells configure sites
```

- Choose "Create a new site"
- Choose `443` as the port to bind to
- Enter your FQDN as the address to bind to
- Choose "Automagically generate certificate with Let's Encrypt"
- Enter your Email, Accept Let's Encrypt EULA
- Redirect default `HTTP` port towards `HTTPS`  
- Double check and save.

#### Verification


```sh
# Open permanently standard HTTP ports on Firewall
firewall-cmd --permanent --zone=public --add-service=http
firewall-cmd --permanent --zone=public --add-service=https

# Restart your server
sudo su - pydio 
cells start
```

Connect to your web site at `https://<YOUR_FQDN>`. A valid certificate is now used.

Stop your server once again before performing the finalisation steps.

## Finalisation

### Run your server as a service with systemd

Create a configuration file `sudo vi /etc/systemd/system/cells.service` with the following:

```conf
[Unit]
Description=Pydio Cells
Documentation=https://pydio.com
Wants=network-online.target
After=network-online.target
AssertFileIsExecutable=/opt/pydio/bin/cells

[Service]
User=pydio
Group=pydio
PermissionsStartOnly=true
AmbientCapabilities=CAP_NET_BIND_SERVICE
ExecStart=/opt/pydio/bin/cells start
Restart=on-failure
StandardOutput=journal
StandardError=inherit
LimitNOFILE=65536
TimeoutStopSec=5
KillSignal=INT
SendSIGKILL=yes
SuccessExitStatus=0
WorkingDirectory=/home/pydio

# Add environment variables
Environment=CELLS_WORKING_DIR=/var/cells

[Install]
WantedBy=multi-user.target
```

Reload systemd daemon, enable and start cells:

```sh
sudo systemctl daemon-reload
sudo systemctl enable cells
sudo systemctl restart cells
```

#### Verification

```sh
# you can check the system logs to insure everything seems OK
sudo journalctl -fu cells -S -1h
```

Connect to your certified web site at `https://<YOUR_FQDN>`.

**You are now good to go**. Happy file sharing!

## Troubleshooting

### Main tips

With Cells running as a service, you can access the logs in different ways:

```sh
# Pydio file logs
tail -200f /var/cells/logs/pydio.log
# Some of the microservices have their own log files, check:
ls -lsah /var/cells/logs/

# Check systemd files
journalctl -fu cells -S -1h
```

### SELinux is enforced

If, after a successful installation and when you try to navigate to the main application page with your browser, you land on a blank page with following message:

> Access denied.

ensure you have modified SELinux to be in permissive mode.
