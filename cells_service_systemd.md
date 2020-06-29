When deployed in production environment, we generally advise to run Pydio Cells as a systemd service.

This configuration assumes that you have followed our recommended best practices during installation process, see our [detailed installation guides](/en/docs/cells/v2/os-specific-guides). Adapt to your specific setup if necessary.

Thus you have:

- defined `CELLS_WORKING_DIR` as `/var/cells`
- the downloaded binary at `/opt/pydio/bin/cells`
- a `pydio` user that has correct rights on `/opt/pydio` (read and execute) and `/var/cells`
- the `pydio` user has **only** `sudo` permission to execute the setcap command. Typically on Linux, do:

```sh
echo "pydio        ALL=(ALL)       NOPASSWD: /sbin/setcap 'cap_net_bind_service=+ep' /opt/pydio/bin/cells" | sudo tee -a /etc/sudoers.d/pydio
```

Create a new `/etc/systemd/system/cells.service` file with following content:

```conf
[Unit]
Description=Pydio Cells
Documentation=https://pydio.com
Wants=network-online.target
After=network-online.target
AssertFileIsExecutable=/opt/pydio/bin/cells

[Service]
WorkingDirectory=/tmp/cells
User=pydio
Group=pydio
PermissionsStartOnly=true
ExecStartPre=/usr/bin/sudo /sbin/setcap 'cap_net_bind_service=+ep' /opt/pydio/bin/cells
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
Environment=PYDIO_LOGS_LEVEL=production
Environment=CELLS_WORKING_DIR=/var/cells
Environment=PYDIO_ENABLE_METRICS=false

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
tail -200f /var/cells/logs/cells.log
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
WorkingDirectory=/tmp/cells
...
```

Thus the _current directory_ for the various processes that are launched by the app is `/tmp/cells`, that we find safer than the default location that is usually the home directory of the user that runs the app.

Please note that this directory **must exist and be writable** before launching the application.

If it is not the case, the system fails to start with a message that can be quite cryptic for people that are not _systemd fluent_:

```log
...
code=exited, status=200/CHDIR
...
```
