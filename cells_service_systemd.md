When deployed in production environment, we generally advise to run Pydio Cells as a systemd service.

This configuration assumes that you have followed our recommended best practices during installation process, see our [detailed installation guides](/en/docs/cells/v2/os-specific-guides). Adapt to your specific setup if necessary.

Thus you have:

- the downloaded binary at `/home/pydio/cells`
- the `pydio` user has **only** `sudo` permission to execute the setcap command. Typically on Linux, do:

```sh 
echo "pydio        ALL=(ALL)       NOPASSWD: /sbin/setcap 'cap_net_bind_service=+ep' /home/pydio/cells" | sudo tee -a /etc/sudoers.d/pydio
```

Create a new `/etc/systemd/system/cells.service` file with following content:

```conf
[Unit]
Description=Pydio Cells
Documentation=https://pydio.com
Wants=network-online.target
After=network-online.target
AssertFileIsExecutable=/home/pydio/cells

[Service]
WorkingDirectory=/tmp/cells
User=pydio
Group=pydio
PermissionsStartOnly=true
ExecStartPre=/usr/bin/sudo /sbin/setcap 'cap_net_bind_service=+ep' /home/pydio/cells
ExecStart=/home/pydio/cells start
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

[Install]
WantedBy=multi-user.target
```

Then, enable and start the service:

```sh
sudo systemctl enable cells
sudo systemctl start cells
```

By default, logs can then be found in `<CELLS_WORKING_DIR>/logs/` folder, typically, on Linux:

```sh
tail -200f /home/pydio/.config/pydio/cells/logs/pydio.log
```
