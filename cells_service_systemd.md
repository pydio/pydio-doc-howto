You can use systemd as a way to run cells as a service, it will provide you advantages such as automatically restarting your cells after failure.

_Note: this configuration assume you have done a vanilla setup following our installation guides. Adapt to your specific setup if necessary._

Create new `/etc/systemd/system/cells.service` file with following content:

```conf
[Unit]
Description=Pydio Cells
Documentation=https://pydio.com
Wants=network-online.target
After=network-online.target
AssertFileIsExecutable=/home/pydio/cells
[Service]
WorkingDirectory=/home/pydio/.config/
User=pydio
Group=pydio
PermissionsStartOnly=true
ExecStart=/bin/bash -c 'exec /home/pydio/cells start &>> /home/pydio/.config/pydio/cells/logs/cells.log'
Restart=on-failure
StandardOutput=journal
StandardError=inherit
LimitNOFILE=65536
TimeoutStopSec=5
KillSignal=INT
SendSIGKILL=yes
SuccessExitStatus=0
[Install]
WantedBy=multi-user.target
```

If you are running Pydio Cells in a production environment, you probably want to enable production logging:

```conf
# Add en environment variable in the [Service] section
Environment=PYDIO_LOGS_LEVEL=production
```

Then, enable and start the service:

```sh
sudo systemctl enable cells
sudo systemctl start cells
```

Logs can then be found in `/home/pydio/.config/pydio/cells/logs/cells.log`.

#### Alternative configuration

If you prefer having the logs integrated in the standard `journalctl` service, you can replace the `ExecStart` directive from the above file with:

```conf
ExecStart=/home/pydio/cells start
```

The output of the service can then be tailed with this command:

```sh
sudo journalctl -f -u cells