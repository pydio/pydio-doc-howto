This How-To comes in addition to [the public documentation about how to manage Backup](https://pydio.com/en/docs/cells/v4/backup-recover) with Pydio Cells.

## Prepare the new machine

The following assumes that your server is running under a recent _Red Hat Enterprise Linux_-like server, installed from a minimal iso.

```sh
# As sudo user on target machine
sudo useradd -m -s /bin/bash pydio

# Add some utils 
sudo apt install wget rsync 

# Registrer CELLS_WORKING_DIR as global environment variable for everyone
sudo tee -a /etc/profile.d/cells-env.sh << EOF
export CELLS_WORKING_DIR=/var/cells
EOF
sudo chmod 0755 /etc/profile.d/cells-env.sh
```

If you have no DB yet, refer to this page https://pydio.com/en/docs/kb/deployment/install-cells-centosrhel for hints about mariaDB installation

```sh
# prepare database
dbUserPwd=<your pwd here>
mysql -u root -p -e "CREATE USER \"pydio\"@\"localhost\" IDENTIFIED BY \"$dbUserPwd\"; CREATE DATABASE cells; GRANT ALL PRIVILEGES ON cells.* to \"pydio\"@\"localhost\"; FLUSH PRIVILEGES;"

# prepare tree structure
sudo mkdir -p /opt/pydio/bin

# retrieve correct version of the binary
sudo wget --output-document=/opt/pydio/bin/cells https://download.pydio.com/latest/cells-enterprise/release/{latest}/linux-amd64/cells-enterprise

# Permissions
sudo chmod a+x /opt/pydio/bin/cells
sudo chown -R pydio:pydio /opt/pydio
sudo ln -s /opt/pydio/bin/cells /usr/local/bin/cells

# if necessary (use of reserved ports 80 & 443)
sudo setcap 'cap_net_bind_service=+ep' /opt/pydio/bin/cells

# Configure systemd
sudo tee /etc/systemd/system/cells.service << EOF
[Unit]
Description=Pydio Cells Enterprise Distribution
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
Environment=PYDIO_ENABLE_METRICS=false

[Install]
WantedBy=multi-user.target
EOF

# and enable the service
sudo systemctl daemon-reload
sudo systemctl enable cells
```

## Backup of the old machine

With vanilla setups, to fully restore the system, you only needs 2 things:

- a dump of the Cells DB
- a copy of the data.

### DB Backup

```sh
# On the source machine

# If it is the "Go Live" migration, it is better to stop the cells service before moving the data. If it is for the staging test, you can leave the app "on".
sudo systemctl stop cells

# As the user who run the app, typically, "pydio"
sudo su - pydio
mkdir ~/backup
cd ~/backup 

## Perform a MySQL dump
dbName=cells
dbUser=pydio
mysqldump -u $dbUser -p $dbName > /home/pydio/backups/current/cells-dump.sql
```

You can then copy the mysql dump to the new machine.

### Data backup

By default, in a single node deployment, all data (both technical and business data) is in a single folder:
- by default at `/home/pydio/.config/pydio/cells`
- if you have followed our best practices at `/var/cells`

You should `rsync` these folder to the new machine.
**We strongly advise** to migrate to `/var/cells` standard location for the base Cells working dir, even if you used another one on the source machine.  
If you choose another working dir, **modify the various scripts and config file accordingly**.

Warning: if you have defined file datasources that are not at the default location (that is under `${CELLS_WORKING_DIR}/data`), you must also copy these folder to your chosen location on the target machine.

## Restoration itself

```sh
# On the **target** machine, as sudo user

# Retrieve the data via rsync:

# If you can afford 2 passes (staging and then prod) and have enough place,
# make the rsync to a tmp folder in the new machine and make a local copy: this speeds up the second pass
sudo mkdir /var/cells_tmp

# WARNING: you should use the pydio user in the source machine, otherwise you will encounter permission issues, 
# typically on the bleve DB and on the keyring file.
sourcePort=56789
sourceDN=your.old.server
sourceUser=pydio
sudo rsync -arvz -e 'ssh -p $sourcePort' --progress $sourceUser@$sourceDN:/var/cells /var/cells_tmp/
# or 
sudo rsync -arvz -e 'ssh -p $sourcePort' --progress $sourceUser@$sourceDN:/home/$sourceUser/.config/pydio/cells /var/cells_tmp/

# Copy locally the working dir
sudo cp -r /var/cells_tmp /var/cells
sudo chown -R pydio:pydio /var/cells

## WARNING: switch to pydio user

# As pydio user 

# Restore SQL dump
scp -P $sourcePort $sourceUser@$sourceDN:/home/$sourceUser/backup/cells-dump.sql .
mysql -u pydio -p cells < cells-dump.sql
# Skipping foreign key checks upon import works around the creation order issue that sometimes happens
mysql --init-command="SET foreign_key_checks=0;" -u pydio -p cells < cells-dump.sql 

# Adapt the pydio.json file

# first make a copy
cp /var/cells/pydio.json /var/cells/pydio.json.orig

## Warning: below commands must be adapted to your specific use case, think twice or ask for help if necessary

# If you went from the "home" layout to the standard "var/cells" layout
sed -i "s;/home/pydio/.config/pydio/cells/;/var/cells/;g" /var/cells/pydio.json

# In single node deployment, if the IP or hostname can change, it might be better to get rid of the peer address param
sed -i "s;\"PeerAddress\": \"10.0.10.143\",;;g" /var/cells/pydio.json

# Adapt URLs
sed -i "s;\"urlInternal\": \"http://oldname.local:8080\";\"urlInternal\": \"http://0.0.0.0:8080\";g"  /var/cells/pydio.json
# or if only (part of) your fqdn has changed, e.g from old-name.example.com to new-name.example.com
sed -i "s/old-name.example.com/new-name.example.com/g"  /var/cells/pydio.json

# If necessary, change the DB login/mdp and then insure everything is correct
vi /var/cells/pydio.json 

```

You should be good to go.

First try to start the server from the command line **as pydio user**, to be able to monitor first start. Then stop the app and restart it with systemd

## Troubleshooting and tips

### Things to check when adapting pydio.json config file

**Make a copy of the file before starting to tweak!!**

- DB URL (typically user & password)
- Peer addresses
- Path of local datasource folder(s) on the new FS
- Bind & public URL

### Clean an existing machine

If you are restoring on a machine that already had cells, you should start with removing legacy data.

**Warning**: the following erase any cells related data on your machine. Think twice.

```sh
# On the target machine, as a sysadmin user

sudo systemctl stop cells
sudo rm -rf /home/pydio/.config/pydio/cells
# or 
sudo rm -rf /var/cells

mysql -u pydio -p -e "DROP DATABASE cells; CREATE DATABASE cells;"

# for the record if you want to change user password
mysql -u root -p -e "ALTER USER 'pydio'@'localhost' IDENTIFIED BY '<the new PWD here>';"

```

### Bleve: cannot create new index

```sh
Dez 18 19:07:20 pydio.example.com cells[115428]: [pydio.grpc.log] Cannot open bleve index /var/cells/services/pydio.grpc.log/syslog.bleve cannot create new index, path already exists
```

This generally means that the corresponding folder has not been correctly / completely retrieved. It usually happens if you forgot to stop the `Cells` service before copying the `CELLS_WORKING_DIRECTORY` content. Double check and try again.

### Cannot restore mysql dump

- Check if db is empty, if user and permission are correct
- Check version of source and target DB
- When getting an error upon import, try with no check on foreign key:

```sh
mysql --init-command="SET foreign_key_checks=0;" -u pydio -p cells < cells-dump.sql
```

### Check I/O on disk

```sh
dd if=/dev/zero  of=/tmp/test1.img  bs=1G  count=1  oflag=dsync
```

### Check DS

If you **are running an enterprise distribution instance** and see some dupplication errors during the indexing phase while starting the **new** instance, you can fix the index with following commands:

```sh
# As Pydio user
cells tools find-duplicate --dry-run --datasource pydiods1
# if it seems OK:
cells tools find-duplicate --dry-run=false --datasource pydiods1

# Before 2.2
# As Pydio user
cells data find-duplicate --dry-run --datasource pydiods1
# if it seems OK:
cells data find-duplicate --dry-run=false --datasource pydiods1
```

### network / certificate issues

It is always better to check the target URL from the command line with curl in order to skip the noise introduced by the various caches, typically:

```sh
curl -v --insecure https://files.example.com/index.html
```