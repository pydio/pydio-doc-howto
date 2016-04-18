**Author** Thomas Sarboni from https://blog.max-k.org

Following are the instructions for installing [Pydio](https://pydio.com/) with [Nginx](http://nginx.org/en/) on [Archlinux](https://wiki.archlinux.org/index.php/Arch_Linux).

In this guide, we will use PHP-FPM for processing php.

## Packages installation
Pydio is available on Archlinux using [pydio package](https://aur.archlinux.org/packages/pydio/) on [AUR](https://aur.archlinux.org/) so, to install it, the simplest way is to use [yaourt](https://wiki.archlinux.org/index.php/yaourt) :

	$ yaourt -S nginx php-fpm pydio

## Php configuration
To allow pydio to work properly, some customizations must be done to php configuration file (**/etc/php.ini**) :

### Extensions activation
Make sur these lines are uncommented :

	extension=gd.so
	extension=iconv.so
	extension=mcrypt.so

### File upload parameters
It can be usefull to allow bigger uploads (here, we set a maximum size of 20GiB)  :

	file_uploads = On
	post_max_size = 20G
	upload_max_filesize = 20G
	max_file_uploads = 20000

### Other parameters
We explicitly set sessions directory :

	session.save_path = "/tmp"

We disable output_buffering to improve performances :

	output_buffering = Off

To allow background tasks to run, we need to allow to php to access to the webapp directory :

	open_basedir = /srv/http/:[...]:/var/lib/pydio/

## Nginx configuration
An example nginx vhost is already provided by the package so we need to make a copy (**as root**).

	# cp /usr/share/doc/pydio/example_nginx_vhost.conf /etc/webapps/pydio/nginx.conf

And customize it with our real domain name (eg : pydio.max-k.org) (**as root**) :

	# sed -i 's/pydio.local/pydio.max-k.org/g' /etc/webapps/pydio/nginx.conf

Now, we include it from our main nginx configuration file (**/etc/nginx/nginx.conf**) :

	http {
	    [...]
	    include /etc/webapps/pydio/nginx.conf;
	    [...]
	 }

**An SSL example vhost is also included. If you want to use it, you need to customize it to point to your SSL x509 certificates.**

## Services starting and tests
### Services starting
To start, or restart,  needed services, we need to run the following command (as root) :

	# systemctl restart php-fpm.service
	# systemctl restart nginx.service

**We deliberately skip the DNS part of the configuration here because it’s beyond the scope of this how-to.**

**You can add an entry to your /etc/hosts for testing purpose.**

### Let’s test
Open a browser on you desktop and access you Pydio domain name :

[:image-popup:system/installing_on_archlinux/PydioDiagnosticTool.png]

Pydio is working now.

You can now follow the instructions to continue your installation.

**If you have increased the maximum upload file size, you also need to set upload file size limit on pydio admin interface.**

### Automatically start services
To automatically start services at startup, you need to enable them (**as root**) :

	# systemctl enable php-fpm.service
	# systemctl enable nginx.service

## Desktop client installation
Desktop client is available using [pydio-sync package](https://aur.archlinux.org/packages/pydio-sync/) on [AUR](https://aur.archlinux.org/) so, to install it, the simplest way is to use [yaourt](https://wiki.archlinux.org/index.php/yaourt) :

	$ yaourt -S pydio-sync

You can launch it using its included freedesktop.org compliant menu entry or using the following command :

	$ pydio-sync

A git version is also available [here](https://aur.archlinux.org/packages/pydio-sync-git/). To install it, use the following command :

	$ yaourt -S pydio-sync-git
