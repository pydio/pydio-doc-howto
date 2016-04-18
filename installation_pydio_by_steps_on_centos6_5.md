This article will introduce steps to install Pydio step by step in CentOS 6.5 environement.

If you are running CentOS7, please make sure to read the [dedicated article here](https://pydio.com/en/docs/kb/system/install-pydio-600-centos-7/).

Firstly, download pydio from sourceforge at https://pyd.io/download/. In this case, we are going to use installation from zip file

## Step 1: Preparation
– Download CentOS from http://wiki.centos.org/Download. In this article, we used CentOS-6.5-x86_64-minimal.iso

– Install operating system

– Download Pydio from https://pydio.com/en/get-pydio. In this case, we are going to use installation from zip file

## Step 2: Installation dependencies for Pydio
– Update repository database


	# for mcrypt module

	wget http://dl.fedoraproject.org/pub/epel/6/x86_64/epel-release-6-8.noarch.rpm

	wget http://rpms.famillecollet.com/enterprise/remi-release-6.rpm

	rpm -Uvh remi-release-6*.rpm epel-release-6*.rpm

	yum -y update

### PHP and dependencies:
	yum install -y php php-apc php-mbstring php-pecl-apc php-mysql php-cli php-devel php-gd php-ldap php-pecl-memcache php-pspell php-snmp php-xmlrpc php-xml php-imap php-mcrypt*

Configure max size file for uploading in php.ini

upload_max_filesize = 1024M
post_max_size = 1024M

Turn off output_buffering in php.ini

output_buffering = Off

### MySQL
 	yum install -y mysql-server

running /usr/bin/mysql_secure_installation and following the instruction for security mysql server

 	/usr/bin/mysql_secure_installation

– MySQL preparation database


	#Create database

service mysqld start

	# connect to mysql server

	mysql -u username -p

	# in mode mysql>_

	create database pydio
	create user pydio@localhost identified by 'mysqlpassword'
	grant all privileges on pydio.* to pydio@localhost identified by 'mysqlpassword' with grant Options



	chkconfig mysqld --levels 235 on

### Apache2

	yum install httpd

	# install module ssl

	yum install -y openssl mod_ssl

	# start apache with system

chkconfig httpd --levels 235 on
Apache2 generation automatically certificate

sample shell script: [generate-certificte.sh](https://github.com/pydio/configs/blob/master/archive/configs/generate-certificate.sh)

After running this script, two files are created:

Key file: /etc/pki/tls/private/pydio.pem

Certificate file: /etc/pki/tls/certs/pydio.csr

Attach this key/cert to apache

	# override pydio.conf when we use SSL
	sed -i "s/localhost.crt/pydio.csr/g" /etc/httpd/conf.d/ssl.conf
	sed -i "s/localhost.key/pydio.pem/g" /etc/httpd/conf.d/ssl.conf

## Step 3. Install PYDIO
Click on https://pydio.com/en/get-pydio to choose the way you would like to install pydio. In this article, we supposed using install from linux package on CentOS system

Install the Pydio repository:

	rpm -Uvh http://dl.ajaxplorer.info/repos/pydio-release-1-1.noarch.rpm

Or simply click on [this link](http://dl.ajaxplorer.info/repos/pydio-release-1-1.noarch.rpm) and install via GUI tools. This will install /etc/yum.repos.d/pydio.repo which configures the repository, allowing update management. Once installed, update the yum database and install pydio :


	yum update

	yum install pydio

#### _Configure Virtual Directory for pydio_

sample configure file in /etc/httpd/conf.d/pydio.conf

In this sample configuration, all http request will be ridirected to HTTPS in using module rewrite and Certificate which is generated in previous step
http://192.168.0.111/pydio_public/38f3b5.php

## Step 4. Post-installation
Go to Global Settings >> Application Core >> Main Options >> Sharing >>
Modify Download URL: http://192.168.0.111/pydio_public

## Step 5. Hardening
### Minimize permission on your Pydio data

	# www is group name for httpd
	chown -R root:www PYDIO_INSTALL_DIR
	cd PYDIO_INSTALL_DIR/
	find ./ -type d -exec chmod u=rwx,g=rx,o= '{}' \;
	find ./ -type f -exec chmod u=rw,g=r,o= '{}' \;

	#Mod data dir for config changes and logs
	find data -type d -exec chmod ug=rwx,o= '{}' \;
	find data -type f -exec chmod ug=rw,o= '{}' \;

### Configure iptables service:
sample for iptables: [iptables.sh](https://github.com/pydio/configs/blob/master/shell-script/iptables.sh)

### Harden you server with several steps
sample shell script: [harden-centos.sh](https://github.com/pydio/configs/blob/master/shell-script/harden-centos)

Enforce by using SELinux: [see](https://pydio.com/en/docs/kb/security/pydio-security-enhanced-linux-selinux)