Preparation

Install net-tools (optional)

	yum install net-tools

Add repository

	wget http://dl.fedoraproject.org/pub/epel/7/x86_64/e/epel-release-7-5.noarch.rpm
	rpm -ivh epel-release-7-5.noarch.rpm
	yum update

Install mariadb-server

	yum install -y mariadb mariadb-server

	systemctl start mariadb.service
	systemctl enable mariadb.service
	mysql_secure_installation

Install httpd server

	yum install httpd
	systemctl enable httpd.service
	systemctl start httpd.service

Switch to lestening on IPv4
On CentOS7, httpd by default listen on port 80 of tcp6. To enable IPv4, we do:
in /etc/httpd/conf/httpd.conf,
replace Listen 80 by Listen 0.0.0.0:80

Open firewall port for httpd

	firewall-cmd --permanent --zone=public --add-service=http
	firewall-cmd --permanent --zone=public --add-service=https
	firewall-cmd --reload

Install important dependencies

	yum -y install php php-gd php-ldap php-pear php-xml php-xmlrpc php-mbstring curl php-mcrypt* php-mysql

Install Pydio
Add repository

	rpm -Uvh http://dl.ajaxplorer.info/repos/pydio-release-1-1.noarch.rpm
	yum update
	yum --disablerepo=pydio-testing install pydio

**Configure permission for Pydio**

Visit this article: https://pyd.io/permission-for-pydios-filesfolders/

**htaccess**

Modify /var/lib/pydio/public/.htaccess

	Order Deny,Allow
	Allow from all
	<Files ".ajxp_*">
	deny from all

	RewriteEngine on
	RewriteBase pydio_public
	RewriteCond %{REQUEST_FILENAME} !-f
	RewriteCond %{REQUEST_FILENAME} !-d
	RewriteRule ^([a-zA-Z0-9_-]+)\.php$ share.php?hash=$1 [QSA]
	RewriteRule ^([a-zA-Z0-9_-]+)--([a-z]+)$ share.php?hash=$1&lang=$2 [QSA]
	RewriteRule ^([a-zA-Z0-9_-]+)$ share.php?hash=$1 [QSA]

Visit this article: https://pyd.io/upgrade-pydio-5-2-5-to-6-0-0/

**Public share link generation**

Modify /etc/httpd/conf.d/pydio.conf

	Alias /pydio /usr/share/pydio
	Alias /pydio_public /var/lib/pydio/public

	<Directory "/usr/share/pydio">
  	     	Options FollowSymLinks
 	     	AllowOverride Limit FileInfo
			Require all granted
  	    	php_value error_reporting 2



	<Directory "/var/lib/pydio/public">
    	    AllowOverride Limit FileInfo
			Require all granted
    	  	php_value error_reporting 2

In Pydio, Application Parameters >> Application core >> Main Options >> Sharing section

PUBLIC FOLDER: /var/lib/pydio/public

DOWNLOAD URL: http://youraddress/pydio_public

**php.ini**

	output_buffering = Off

	#for upload file less than 1GB
	post_max_size = 1G
	upload_max_filesize = 1G
	memory_limit = 1224M
 

**charset**

use shell to know current system lang

	echo $LANG

edit /etc/pydio/bootstrap_conf.php by adding this line

	define("AJXP_LOCALE", "en_US.UTF-8");

**Troubleshooting:**
**_Share link permission denined_**
https://pyd.io/f/topic/share-links-permission-denied-centos7/

**Change URI**
https://pyd.io/f/topic/change-pydio-default-url-centos7/