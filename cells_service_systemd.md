When deployed in production environment, we generally advise to run Pydio Cells as a `systemd` service.

This configuration assumes that you have followed our [recommended best practices](./en/docs/cells/v2/best-practices) during the installation process. Adapt to your specific setup if necessary.

Thus you have:

- defined `CELLS_WORKING_DIR` as `/var/cells`
- the downloaded binary at `/opt/pydio/bin/cells`
- a `pydio` user that has correct (read and execute) permissions on `/opt/pydio` and `/var/cells`.

Create a new `/etc/systemd/system/cells.service` file with following content:

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
WorkingDirectory=/home/pydio
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

# Add environment variables
Environment=CELLS_LOGS_LEVEL=production
Environment=CELLS_ENABLE_METRICS=false
Environment=CELLS_WORKING_DIR=/var/cells

[Install]
WantedBy=multi-user.target
```

Then, enable and start the service:

```sh
sudo systemctl enable cells
sudo systemctl start cells
```

## Various Notes

### Loging

With the above configuration, Pydio Cells logs in rolling text files of 10MB under `<CELLS_WORKING_DIR>/logs/` folder. Typically, on Linux:

```sh
tail -200f /var/cells/logs/pydio.log
```

It is worth noting that logs are also outputed to the systemd standard loging system so that you can also see them with e.g.:

```sh
sudo journalctl -f -u cells --since "1 hour ago"
```

### Systemd working directory

In the above file, we also overwrite the default systemd configuration for the working directory by using:

```conf
...
[Service]
WorkingDirectory=/home/pydio
...
```

Please note that this directory **must exist and be writable** before launching the application.

If it is not the case, the system fails to start with a message that can be quite cryptic for people that are not _systemd fluent_:

```log
...
code=exited, status=200/CHDIR
...
```
