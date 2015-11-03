This article is takes the HowToForge articles as reference, adding some additional remarks. CentOS 7 introduced major changes compared to CentOS6.X. The following tutorials are very complete and you can follow them step by step.

### Install LAMP
https://www.howtoforge.com/apache_php_mysql_on_centos_7_lamp

## Install Pydio
Reference: https://www.howtoforge.com/tutorial/how-to-install-pydio-on-centos-7/

Our notes

+ Make sure to enable Apache ModRewrite (the module is already here, you just have to create a file inside /etc/httpd/conf.modules.d/ to load it) and use AllowOverride All instead of AllowOverride None inside the Pydio virtual host (otherwise mod_rewrite cannot be used).
This is quite critical for Pydio 6
+ Add php-mcrypt to the list of php modules you install using yum.
+ Type echo $LANG on your command line to read the exact locale setting on your server (and use it during the Pydio installer).
+ Edit /etc/php.ini and replace output_buffering value to Off instead of default 4096

### Fix Sharing htaccess
Share links will not work out of the box. Follow the steps:

+ Update the Pydio Main Options (in the Admin Panel), leave the “Download Folder” as is, but change the Download URL” to http://YOUR_IP/pydio_public
+ Go to a workspace (Common Files) and “Share” a file. The link should be correct, but not working yet
+ On the server, edit the /var/lib/pydio/public/.htaccess file and replace it with the following content

	#Order Deny,Allow
	#Allow from all
	Require all granted

	<Files ".ajxp_*">
	deny from all
	RewriteEngine on
	RewriteBase /pydio_public
	RewriteCond %{REQUEST_FILENAME} !-f
	RewriteCond %{REQUEST_FILENAME} !-d
	RewriteRule ^([a-zA-Z0-9_-]+)\.php$ share.php?hash=$1 [QSA]
	RewriteRule ^([a-zA-Z0-9_-]+)--([a-z]+)$ share.php?hash=$1&lang=$2 [QSA]
	RewriteRule ^([a-zA-Z0-9_-]+)$ share.php?hash=$1 [QSA]
	
Main changes being the “Require all granted” replacing the two first instructions, and the RewriteBase fixed to /pydio_public.

Now the share link should be working!

Enjoy!