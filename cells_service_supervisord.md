//TODO

### With Supervisor

#### Requirements

You only need to install supervisor if not yet present:

```sh
sudo apt-get install supervisor
# Enable and start the service
sudo systemctl enable supervisor
sudo systemctl start supervisor
```

#### Configuration

_Note: this configuration assume you have done a vanilla setup following our install guides. Adapt to your specific setup if necessary._

You must then declare the path to your binary **cells** file in a supervisor configuration file:

- Create a file here `/etc/supervisor/conf.d/<the-file>.conf` named for instance `cells.conf`
- Add this after having replaced the `<path-to-binary>` and `<user-launching-cells>` place holders by their respective values depending on your setup:

```conf
[program:cells]
command=/home/<path-to-binary> start
directory=/home/<folder-of-the-binary>       ; directory to cwd to before exec (def no cwd)
;umask=022                                   ; umask for process (default None)
;priority=999                                ; the relative start priority (default 999)
autostart=true                               ; start at supervisord start (default: true)
autorestart=unexpected                       ; whether/when to restart (default: unexpected)
startsecs=15                                 ; number of secs prog must stay running (def. 1)
startretries=5                               ; max # of serial start failures (default 3)
exitcodes=0,2                                ; 'expected' exit codes for process (default 0,2)
stopsignal=QUIT                              ; signal used to kill process (default TERM)
stopwaitsecs=10                              ; max num secs to wait b4 SIGKILL (default 10)
stopasgroup=false                            ; send stop signal to the UNIX process group (default false)
;killasgroup=false                           ; SIGKILL the UNIX process group (def false)
user=<user-launching-cells>                  ; setuid to this UNIX account to run the program

redirect_stderr=true                         ; redirect proc stderr to stdout (default false)
stdout_logfile=/home/<user-launching-cells>/.config/pydio/cells/logs/cells.log
stdout_logfile_maxbytes=1MB                  ; max # logfile bytes b4 rotation (default 50MB)
stdout_logfile_backups=10                    ; # of stdout logfile backups (default 10)
stdout_capture_maxbytes=1MB                  ; number of bytes in 'capturemode' (default 0)
;stdout_events_enabled=false                 ; emit events on stdout writes (default false)
stderr_logfile=/home/<user-launching-cells>/.config/pydio/cells/logs/cells_err.log        ; stderr log path, NONE for none; default AUTO
;stderr_logfile_maxbytes=1MB                 ; max # logfile bytes b4 rotation (default 50MB)
;stderr_logfile_backups=10                   ; # of stderr logfile backups (default 10)
;stderr_capture_maxbytes=1MB                 ; number of bytes in 'capturemode' (default 0)
;stderr_events_enabled=false                 ; emit events on stderr writes (default false)
;environment=TMPDIR="/tmp",B="2"             ; process environment additions (def no adds)
;serverurl=AUTO                              ; override serverurl computation (childutils)
```

Configure supervisor to monitor this new program by using following command:

```sh
sudo supervisorctl reread
```

_Note: this triggers a reload of all `*.conf` files located within the `/etc/supervisor/conf.d` directory_

Then enact the changes with:

```sh
sudo supervisorctl update
```

#### Usage

You can now monitor your program by using `supervisorctl`

``` sh
$ sudo supervisorctl
cells                             RUNNING   pid 3365, uptime 1:10:26
supervisor>
```

To stop and start your program, you can then do:

```sh
supervisor> stop cells
long_script: stopped
supervisor> start cells
long_script: started
supervisor> restart cells
long_script: stopped
long_script: started
```

To check the status:

```sh
supervisor> status
cells                             RUNNING   pid 3365, uptime 1:13:07
supervisor>
```

Use `quit` to leave the supervisor menu.

You now have Pydio Cells running as a daemon and auto-restarting after server reboot.

## CentOS

On a RHEL/CentOS system, you have two options:

- Run as a service
- Use Supervisor

### Configure Cells as a systemd service

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
#ExecStartPre=echo \"Starting Pydio Cells - Home Edition service\""
ExecStart=/home/pydio/cells start
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

If you are running Pydio Cells in a production environment, you probably want to enable production logging.  
You have 2 options:

```conf
# Add en environment variable in the [Service] section
Environment=PYDIO_LOGS_LEVEL=production

# *or* replace the 13th line with
ExecStart=/home/pydio/cells start --log production
```

Then, enable and start the service:

```sh
systemctl enable cells
systemctl start cells
```

The output of the service can be seen via the journal service:

```sh
journalctl -f -u cells
```

### Using Supervisord

This configuration is based on a system that has a **pydio** Unix account. Please refer to [this tutorial](/en/docs/cells/v1/centosrhel-systems) or adapt to your custom setup if necessary.

- Install supervisor: `sudo yum install supervisor`
- Create a new file `/etc/supervisord.d/cells.ini` with following content:

```conf
[program:cells]
command=/home/pydio/cells start
directory=/home/pydio       ; directory to cwd to before exec (def no cwd)
;umask=022                     ; umask for process (default None)
;priority=999                  ; the relative start priority (default 999)
autostart=true                ; start at supervisord start (default: true)
autorestart=unexpected        ; whether/when to restart (default: unexpected)
startsecs=15                   ; number of secs prog must stay running (def. 1)
startretries=5                ; max # of serial start failures (default 3)
exitcodes=0,2                 ; 'expected' exit codes for process (default 0,2)
stopsignal=QUIT               ; signal used to kill process (default TERM)
stopwaitsecs=10               ; max num secs to wait b4 SIGKILL (default 10)
stopasgroup=false             ; send stop signal to the UNIX process group (default false)
;killasgroup=false             ; SIGKILL the UNIX process group (def false)
user=pydio                 ; setuid to this UNIX account to run the program

redirect_stderr=true          ; redirect proc stderr to stdout (default false)
stdout_logfile=/home/pydio/.config/pydio/cells/logs/cells.log
stdout_logfile_maxbytes=1MB   ; max # logfile bytes b4 rotation (default 50MB)
stdout_logfile_backups=10     ; # of stdout logfile backups (default 10)
stdout_capture_maxbytes=1MB   ; number of bytes in 'capturemode' (default 0)
;stdout_events_enabled=false   ; emit events on stdout writes (default false)
stderr_logfile=/home/pydio/.config/pydio/cells/logs/cells_err.log        ; stderr log path, NONE for none; default AUTO
;stderr_logfile_maxbytes=1MB   ; max # logfile bytes b4 rotation (default 50MB)
;stderr_logfile_backups=10     ; # of stderr logfile backups (default 10)
;stderr_capture_maxbytes=1MB   ; number of bytes in 'capturemode' (default 0)
;stderr_events_enabled=false   ; emit events on stderr writes (default false)
;environment=TMPDIR="/tmp",B="2"       ; process environment additions (def no adds)
;serverurl=AUTO                ; override serverurl computation (childutils)
```

- Enable supervisor start with system `systemctl enable supervisor && systemctl start supervisor`
- Update new program to supervisor: `supervisorctl update`
- Start cell program in supervisor: `supervisorctl start cells`

To insure everything is correctly configured, restart the machine. Pydio Cells is now launched by supervisord.

To watch the output, you can use: `tail -f /home/pydio/.config/pydio/cells/logs/cells.log`
