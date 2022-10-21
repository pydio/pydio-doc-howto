_This guide explains how to install and configure Cells on a Raspberry Pi system_.

**Use case**

Deploy a self-contained Pydio Cells instance on your local home network with a simple Raspberry Pi.

**Requirements**

- Although we tested and could start Cells on a Rasberry Pi 3B with only 1GB of RAM, we suggest to use a version 4B with at least 4 GB RAM.
- **Storage**: 32 SD card
- **Operating System**:
  - Raspbian (Bullseye, Buster or Stretch), the official Raspberry Pi desktop OS (which a Raspbian repackaged the Raspberry Pi team) also works out of the box.  
  - An admin user with sudo rights that can connect to the server via SSH
- **Networking**: TODO.

## Installation

### Dedicated user and file system layout

We recommend to run Pydio Cells with a dedicated `pydio` user with **no sudo** permission:

```sh
# Create pydio user with a home directory
sudo useradd -m -s /bin/bash pydio

# Create necessary folders
sudo mkdir -p /opt/pydio/bin /var/cells/certs
sudo chown -R pydio: /opt/pydio /var/cells

# Add system-wide ENV var
sudo tee -a /etc/profile.d/cells-env.sh << EOF
export CELLS_WORKING_DIR=/var/cells
EOF
sudo chmod 0755 /etc/profile.d/cells-env.sh
```

#### Verification

Login as user `pydio` and make sure that the environment variables are correctly set:

```sh
user@raspberrypi:~$ sudo su - pydio 
pydio@raspberrypi:~$ echo $CELLS_WORKING_DIR
/var/cells
pydio@raspberrypi:~$ exit
```

### Database

We use the default mariadb-server package shipped with Bullseye, it installs the 10.5 version with no hassle:

```sh
sudo apt install mariadb-server
# You should run the script to secure your install
sudo mysql_secure_installation

# Open MySQL CLI to create your database and a dedicated user
sudo mysql -u root -p
```

Start a MySQL prompt and create the database and the dedicated `pydio` user.

```mysql
CREATE DATABASE cells;
CREATE USER 'pydio'@'localhost' IDENTIFIED BY '<PUT YOUR PASSWORD HERE>';
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

Note: we only started shipping the necessary ARM build for Cells at v4.

```sh
# As pydio user
sudo su - pydio 

# Download correct binary
distribId=cells 
# or for Cells Enterprise
# distribId=cells-enterprise 
wget -O /opt/pydio/bin/cells https://download.pydio.com/latest/${distribId}/release/{latest}/linux-arm/${distribId}

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

If you used the browser install, you can login in the web browser as user `admin`.

If you have done the CLI install, you first need to start the server:

```sh
sudo su - pydio 
cells start
```

Connect and login at `https://<YOUR PUBLIC IP>:8080`

**Note**:  
At this stage, we start the server in **foreground** mode. In such case, it is important that you **always stop** the server using the `CTRL + C` shortcut before calling the `start` command again.

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
sudo systemctl enable --now cells
```

#### Verification

```sh
# you can check the system logs to insure everything seems OK
journalctl -fu cells -S -1h
```

**You are now good to go**. Happy file sharing!

## Troubleshooting

### Main tips

With cells as a service, you can access the logs in different ways:

```sh
# Pydio file logs
tail -200f /var/cells/logs/pydio.log
# Some of the microservices have their own log files, check:
ls -lsah /var/cells/logs/

# Check systemd files
journalctl -fu cells -S -1h
```
