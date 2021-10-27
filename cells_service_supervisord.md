 On UNIX-like operating systems, you can use [Supervisor](http://supervisord.org) to run your Pydio Cells instance as a service: this enables for instance automated application relaunch after a server reboot.

To install Supervisor on Debian-like systems for instance, you can do:

```sh
sudo apt-get install supervisor
# Enable and start the service
sudo systemctl enable supervisor
sudo systemctl start supervisor
```

## Configuration for Debian/Ubuntu based systems

_Note: this configuration assumes you have done a vanilla setup by following our install guides. Adapt to your specific setup if necessary._

You must then declare the path to your `cells` binary file in a supervisor configuration file:

- Create a `/etc/supervisor/conf.d/cells.conf` file
- Add this after having replaced the `<path-to-binary>` and `<user-launching-cells>` place holders by their respective values depending on your setup:

```conf
[program:cells]
command=<path-to-binary> start
directory=<path-to-cells-working-dir>       ; directory to cwd to before exec (def no cwd)
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
stdout_logfile=<path-to-cells-working-dir>/logs/cells.log
stdout_logfile_maxbytes=1MB                  ; max # logfile bytes b4 rotation (default 50MB)
stdout_logfile_backups=10                    ; # of stdout logfile backups (default 10)
stdout_capture_maxbytes=1MB                  ; number of bytes in 'capturemode' (default 0)
;stdout_events_enabled=false                 ; emit events on stdout writes (default false)
stderr_logfile=<path-to-cells-working-dir>/logs/cells_err.log        ; stderr log path, NONE for none; default AUTO
;stderr_logfile_maxbytes=1MB                 ; max # logfile bytes b4 rotation (default 50MB)
;stderr_logfile_backups=10                   ; # of stderr logfile backups (default 10)
;stderr_capture_maxbytes=1MB                 ; number of bytes in 'capturemode' (default 0)
;stderr_events_enabled=false                 ; emit events on stderr writes (default false)
;environment=TMPDIR="/tmp",B="2"             ; process environment additions (def no adds)
;serverurl=AUTO                              ; override serverurl computation (childutils)
```

Reload Supervisor configuration with:

```sh
sudo supervisorctl reread
```

_Note: this triggers a reload of all `*.conf` files located within the `/etc/supervisor/conf.d` directory_.

Then enact the changes with:

```sh
sudo supervisorctl update
```

### Usage

You can now monitor your program with `supervisorctl`

``` sh
$ sudo supervisorctl
cells                             RUNNING   pid 3365, uptime 1:10:26

# You can now manage Cells service life cycle:
supervisor> stop cells
long_script: stopped
supervisor> start cells
long_script: started
supervisor> restart cells
long_script: stopped
long_script: started
# To check the status:
supervisor> status
cells                             RUNNING   pid 3365, uptime 1:13:07
# To leave supervisor 
supervisor> quit
```

You now have Pydio Cells running as a daemon and auto-restarting after server reboot.

## For CentOS

On a RHEL/CentOS system and assuming you have followed our [recommended best practices](./install-cells-centosrhel) during installation, here is a config sample that will run Cells as a service.

This configuration is based on a system that has, among others, a **pydio** Unix account. Please adapt to your custom setup if necessary.

- Install supervisor: `sudo yum install supervisor`
- Create a new file `/etc/supervisord.d/cells.ini` with following content:

```conf
[program:cells]
command=/opt/pydio/bin/cells start
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
stdout_logfile=/var/cells/logs/cells.log
stdout_logfile_maxbytes=1MB   ; max # logfile bytes b4 rotation (default 50MB)
stdout_logfile_backups=10     ; # of stdout logfile backups (default 10)
stdout_capture_maxbytes=1MB   ; number of bytes in 'capturemode' (default 0)
;stdout_events_enabled=false   ; emit events on stdout writes (default false)
stderr_logfile=/var/cells/logs/cells_err.log        ; stderr log path, NONE for none; default AUTO
;stderr_logfile_maxbytes=1MB   ; max # logfile bytes b4 rotation (default 50MB)
;stderr_logfile_backups=10     ; # of stderr logfile backups (default 10)
;stderr_capture_maxbytes=1MB   ; number of bytes in 'capturemode' (default 0)
;stderr_events_enabled=false   ; emit events on stderr writes (default false)
;environment=TMPDIR="/tmp",B="2"       ; process environment additions (def no adds)
;serverurl=AUTO                ; override serverurl computation (childutils)
```

- Enable and launch Supervisor with Systemd `systemctl enable supervisor && systemctl start supervisor`
- Update Supervisor config: `supervisorctl update`
- Start Cells with Supervisor: `supervisorctl start cells`

To insure everything is correctly configured, restart the machine. Pydio Cells should be running, launched by Supervisord.

To watch the output, you can use: `tail -f /var/cells/logs/cells.log`
