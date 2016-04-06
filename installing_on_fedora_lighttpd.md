**Author** Dmitry

## Why lighttpd
I believe using alternative web servers like lighttpd (or [nginx](http://nginx.org/)) is a better option for most Pydio deployments as these are faster and less resouce-hungry (out of the box, at least) than Apache, tuning which sometimes looks like black magic to me. Lighttpd, which is important for Pydio, is optimized for a large number of parallel (keep-alive) connections (allows high performance AJAX applications) and has async IO capatibility.

## Setting LLMP up
This part is not tied to Pydio, so one may use it just to get LLMP running.

## OS
Server (no-GUI) version of Fedora can be installed via either DVD or netinst image; to get it, select ‘Infrastructure server’ or ‘Minimal’ (if you are an experienced user) in Software Selection menu. You may also edit hostname in Network Settings.

## Init
Switch to root:

`sudo su`

Disclaimer : If your Fedora version is more than 22 you must replace yum by [dnf](https://fedoraproject.org/wiki/Features/DNF)
Install SELinux management utilities:

`yum install policycoreutils-python`

Allow traffic on ports 80 and 443:

`firewall-cmd --add-service=http --permanent`
`firewall-cmd --add-service=https --permanent`

## Web server
### Init

Install:

`yum install lighttpd -y`

After that you must check in lighttpd.conf file if your localhost bind in the good directory :

`server.document_root = server_root + "/lighttpd"`

Start:

`systemctl start lighttpd`

Check if it runs (should output active (running)):

`systemctl status lighttpd --full`

### Fixing warnings
Don’t forget to check if you’ve done everything right with the previous command after fixing each one.

+ (network.c.260) warning: please use server.use-ipv6 only for hostnames, not without server.bind / empty address; your config will break if the kernel default for IPV6_V6ONLY changes:
IPv6 is not covered here, [this manual](http://redmine.lighttpd.net/projects/lighttpd/wiki/IPv6-Config) may help if you need it.
Else disable it in lighttpd’s conf:
`sed s/"server.use-ipv6 = \"enable\""/"server.use-ipv6 = \"disable\""/ /etc/lighttpd/lighttpd.conf -i`
+ (server.c.915) can’t have more connections than fds/2:  1024 1024:
Allow lighttpd to change max number of file descriptors (impacts performance) on startup:
`setsebool -P httpd_setrlimit on #takes a while`
Tell lighttpd to do it:
`sed s/"#server.max-fds = 2048"/"server.max-fds = 2048"/ /etc/lighttpd/lighttpd.conf -i`

### Fin
Restart and enable to start at each boot:

`systemctl restart lighttpd`
`systemctl enable lighttpd`

Point your browser to server’s hostname or IP. You should see lighttpd’s logo.

## PHP
### Init
Install (this includes an accelerator (opcache)):

`yum install php-fpm php-cli lighttpd-fastcgi php-opcache php-xml -y`

### Conf
#### _PHP_

Change user and group from apache to lighttpd PHP-FPM’s conf:

`sed s/apache/lighttpd/ /etc/php-fpm.d/www.conf -i`

Enable cgi.fix_pathinfo:

`sed s/";cgi.fix_pathinfo=1"/"cgi.fix_pathinfo=1"/ /etc/php.ini -i`

#### _Web server_

Enable fastcgi for lighttpd:

`sed s/"#include \"conf.d\/fastcgi.conf\""/"include \"conf.d\/fastcgi.conf\""/ /etc/lighttpd/modules.conf -i`

And configure it:

	echo 'fastcgi.server += ( ".php" =>
	 ((
	 "socket" => "/var/run/php-fpm.sock",
	 "broken-scriptfilename" => "enable"
	 ))
	 )' >> /etc/lighttpd/conf.d/fastcgi.conf

Allow lighty to write to it’s webroot directory:

`semanage fcontext -a -t httpd_sys_rw_content_t "/var/www/lighttpd(/.*)?" #change default selinux context for webroot`
`restorecon /var/www/lighttpd/ #change current context to default`
`chown -R lighttpd:lighttpd /var/www/lighttpd #make lighttpd owner of webroot`
`chmod -R 770 /var/www/lighttpd #allow it to write there`

#### _Optimization_

Switch to sockets (reduce overhead):

`sed s/"listen = 127.0.0.1:9000"/"listen = \/var\/run\/php-fpm.sock"/ /etc/php-fpm.d/www.conf -i`

Restart php-fpm after optimizing.

If you have error it's because you need to edit with a text editor the www.conf file :

`listen = /var/run/php-fpm.sock`

You need to change to owner the of /var/lib/php/session directory for lighttpd

`chown -R lighttpd:lighttpd /var/lib/php/session`

### Fin
Restart php-fpm and lighttpd.

Enable php-fpm to autostart at boot:

`systemctl enable php-fpm`

Create a phpinfo file with write check:

	echo "<?php
	fopen('test.txt', 'w') or die('Bad file permissions or SELinux configuration');
	phpinfo();
	?>" > /var/www/lighttpd/test.php

Point browser to http://server-address/test.php. If you see a table with various PHP information and get an empty file named ‘test.txt’, congratulations – you only have DB left to configure! Don’t forget to remove testing files after finishing testing (`rm -f /var/www/lighttpd/test*`).

### Troubleshooting
+ 403 or ‘Bad file permissions or SELinux configuration’: rerun commands from ‘Allow lighty to write to it’s webroot directory’
+ 503: restart php-fpm and/or lighttpd

## DB
We are going to use MariaDB, which is default in Fedora 19+. If you have an older version, add a repository from [here](https://downloads.mariadb.org/mariadb/repositories/).

### Init
Install and start it:

`yum install mariadb-server -y`
`systemctl start mariadb`

### Conf
Run a security enhancement wizard (it’s best to answer positively to all the questions):

`mysql_secure_installation`

### Testing
Try connecting to DB:

`mysql -p`

### Fin
Restart and enable to start at boot:

`systemctl restart mariadb`
`systemctl enable mariadb`

# Pydio deployment
## Init
Install prerequisites:

`sudo yum install php-mcrypt php-gd php-mysql -y`

## Conf
### Mandatory
Create an SSL certificate:

`openssl req -x509 -newkey rsa:2048 -keyout /etc/ssl/lighttpd.pem -out /etc/ssl/lighttpd.pem -days 365 -nodes`

Make a virtual host configuration file:

`FQDN=(enter FQDN)`
`INSTALL_DIR=(enter install dir, default is "pydio")`

`echo "\$SERVER[\"socket\"] == \":443\" {`
`ssl.engine = \"enable\"`
`ssl.pemfile = \"/etc/ssl/lighttpd.pem\"`
`\$HTTP[\"host\"] == \"$FQDN\" {`
`server.name = \"$FQDN\"`
`server.document-root = \"/var/www/lighttpd/$INSTALL_DIR\"`
`server.errorlog = \"/var/log/lighttpd/$FQDN-error.log\"`
`accesslog.filename = \"/var/log/lighttpd/$FQDN-access.log\"`
`}`
`}" > /etc/lighttpd/vhosts.d/$FQDN.conf`

Tell lighttpd to use it:

`echo "include \"vhosts.d/$FQDN.conf\"" >> /etc/lighttpd/lighttpd.conf`

### Additional
#### _Security_

Don’t allow to list user’s files bypassing Pydio:

`echo "\$HTTP[\"url\"] =~ \"^/data/\" {`
`url.access-deny = (\"\")`
`}" >> /etc/lighttpd/vhosts.d/$FQDN.conf`

#### _WebDAV_

Install dependencies:

`yum install php-mbstring -y`

Enable mod_rewrite:

`sed -i '1iserver.modules += ( "mod_rewrite" )\n\n' /etc/lighttpd/vhosts.d/$FQDN.conf`

Rewrite requests to specific path to pydio’s internal WebDAV server:

`WEBDAV_PATH=(enter path for WebDAV, default is "webdav")`
`echo "url.rewrite = (\"^/$WEBDAV_PATH(.*)\$\" => \"/dav.php\$1\")" >> /etc/lighttpd/vhosts.d/$FQDN.conf`

### Optimization
Disable PHP output buffering:

`sed s/"output_buffering = 4096"/"output_buffering = Off"/ /etc/php.ini -i`

Tune max upload size (upload_max_filesize < post_max_size < memory_limit):

`sed s/"upload_max_filesize = 2M"/"upload_max_filesize = 500M"/ /etc/php.ini -i`
`sed s/"post_max_size = 8M"/"post_max_size = 506M"/ /etc/php.ini -i`
`sed s/"memory_limit = 128M"/"memory_limit = 512M"/ /etc/php.ini -i`
`sed s/"max_file_uploads = 20"/"max_file_uploads = 50"/ /etc/php.ini -i`

Don’t forget to change upload file size limit in Pydio admin UI according to upload_max_filesize you set here.

## Main
Move into the webserver directory:

`cd /var/www/lighttp`d

Download pydio:

`wget "http://sourceforge.net/projects/ajaxplorer/files/latest/download" -O pydio.zip`

Unzip it:

`unzip pydio.zip`

Remove original:

`rm pydio.zip`

Rename pydio folder:

`mv pydio* pydio`

Correct file permissions for new files:

`chown -R lighttpd:lighttpd /var/www/lighttpd/pydio`
`chmod -R 770 /var/www/lighttpd/pydio`

## Fin
Restart lighttpd and php-fpm.

You should be done by now. Access Pydio by visiting (default) https://$FQDN/pydio and WebDAV – by visiting https://$FQDN/webdav (you should first enable it in admin panel and then individually for each user in non-admin UI).

## Troubleshooting
+ The temporary folder used by PHP to save the session data is either incorrect or not writable:
`mkdir /var/lib/php/session`
`chown -R lighttpd:lighttpd /var/lib/php/session`
 

# Fin
Congratulations, you now (probably) have a working file cloud setup. In case not, a good way to get help is a forum thread and a comment to this page with link to it.

You may now consider exploring [plugins](https://pydio.com/en/docs/references/plugins) or using clients: mobile: [iOS](https://itunes.apple.com/us/app/pydio/id709275884) and [Android](https://play.google.com/store/apps/details?id=com.pydio.android.Client), [Thunderbird File](https://pydio.com/fr/node/696) Link and [Desktop Sync](https://pydio.com/fr/node/19).

Also, after using for a while and understanding how nice Pydio is, you may want to [tell developers about it](https://pydio.com/forum/f/forum/troubleshooting/testimonies/). They would be quite pleased if you did.
