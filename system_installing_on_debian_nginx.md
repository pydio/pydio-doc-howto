**Last Update** Updated on Dec. 8 2016 for Pydio 7

Following are the instructions for installing [Pydio](https://pydio.com/) with [Nginx](http://nginx.org/en/) on [Debian Jessie](http://www.debian.org/intro/about).

**Assumptions**

You already have Debian 8.0 (Jessie) running.

You can download the Pydio compressed file to your Debian server. **/var/www/pydio**
To start off with we need to install the prerequisite packages.  I am keen to keep my Raspberry Pi lean and so I did some testing to determine the minimum required packages needed to get the full functionality of Pydio.   Note I am not including the requirements for the Pydio desktop client yet because it is in beta and I am not interested in testing it.   If you wish to use the desktop client you will need some rsync php related packages.

***Add Repository Sources***

First we need to authenticate the nginx repository signature, we add the key used to sign the nginx packages and repository to the apt program keyring. Download this [key](http://nginx.org/keys/nginx_signing.key) and add it to the apt program keyring with the following command:

    wget -O- http://nginx.org/keys/nginx_signing.key | apt-key add -
    wget -O- https://www.dotdeb.org/dotdeb.gpg | apt-key add -

Then you can add at the end of /etc/apt/sources.list file:

    echo deb http://nginx.org/packages/debian/ jessie nginx > /etc/apt/sources.list.d/nginx.list
    echo deb-src http://nginx.org/packages/debian/ jessie nginx >> /etc/apt/sources.list.d/nginx.list

And 

    echo "deb http://packages.dotdeb.org jessie all" > /etc/apt/sources.list.d/dotdeb.list

**Installing Nginx, PHP7-fpm and MySQL Server**

So after that, you can install the prerequisites;

    apt-get update
    apt install nginx mysql-server php7.0 php7.0-fpm php7.0-mysql php7.0-curl php7.0-json php7.0-gd php7.0-intl php7.0-mbstring php7.0-xml php7.0-zip php7.0-exif php7.0-apcu

Once the prerequisites are installed, create the www directory and set ownership;

    chown -R www-data:www-data /var/www

We need to configure php to support larger file uploads so edit the php.ini file;

    vim /etc/php/7/fpm/php.ini

Edit the following values to your liking;

    file_uploads = On
    post_max_size = 20G
    upload_max_filesize = 20G
    max_file_uploads = 20000
    output_buffering = Off

Then restart php fpm service

    /etc/initp.d/php7.0-fpm restart

Now we need to configure Nginx to setup our Pydio web site (use your own domain name below);

    vim /etc/nginx/sites-available/pydio.conf

### For Pydio 7 (for Pydio 6 see below)

As we are security concerned, everything hitting the port 80 is redirected to port 443 over HTTPS. Pydio installation folder is assumed to be **/var/www/pydio**.

	server {
		listen 80;
        ### Change the following line to match your website name
		server_name YOUR_SERVER_NAME;
		rewrite ^ https://$server_name$request_uri? permanent;
	}

	server {
        listen 443 ssl;
        ### Change the following line to match your website name
        server_name YOUR_SERVER_NAME;
        root /var/www/pydio;
        index index.php;

        ### If you changed the maximum upload size in PHP.ini, also change it below
        client_max_body_size 20G;

        # Prevent Clickjacking
        add_header X-Frame-Options "SAMEORIGIN";

        # SSL Settings
        ### If you are using different names for your SSL certificate and key, change them below:
        ssl on;
        ssl_certificate /etc/nginx/ssl/nginx.crt;
        ssl_certificate_key /etc/nginx/ssl/nginx.key;

		# This settings are destined to limit the supported crypto suites, this is optional and may restrict the availability of your website.
		#ssl_ciphers 'EECDH+AESGCM:EDH+AESGCM:AES256+EECDH:AES256+EDH';
		#ssl_protocols TLSv1 TLSv1.1 TLSv1.2;

        add_header Strict-Transport-Security "max-age=16070400; includeSubdomains";

        keepalive_requests    10;
        keepalive_timeout     60 60;
        access_log /var/log/nginx/access_pydio7.log;
        error_log /var/log/nginx/error_pydio7.log;

        client_body_buffer_size 128k;
		# All non existing files are redirected to index.php
        if (!-e $request_filename){
			# For old links generated from Pydio 6
			rewrite ^/data/public/([a-zA-Z0-9_-]+)$ /public/$1?;
            rewrite ^(.*)$ /index.php last;
        }

		# Manually deny some paths to ensure Pydio security
        location ~* ^/(?:\.|conf|data/(?:files|personal|logs|plugins|tmp|cache)|plugins/editor.zoho/agent/files) {
                deny all;
        }

        # Forward PHP so that it can be executed
        location ~ \.php$ {

                fastcgi_param  GATEWAY_INTERFACE  CGI/1.1;
                fastcgi_param  SERVER_SOFTWARE    nginx;
                fastcgi_param  QUERY_STRING       $query_string;
                fastcgi_param  REQUEST_METHOD     $request_method;
                fastcgi_param  CONTENT_TYPE       $content_type;
                fastcgi_param  CONTENT_LENGTH     $content_length;
                fastcgi_param  SCRIPT_FILENAME    $document_root$fastcgi_script_name;
                fastcgi_param  SCRIPT_NAME        $fastcgi_script_name;
                fastcgi_param  REQUEST_URI        $request_uri;
                fastcgi_param  DOCUMENT_URI       $document_uri;
                fastcgi_param  DOCUMENT_ROOT      $document_root;
                fastcgi_param  SERVER_PROTOCOL    $server_protocol;
                fastcgi_param  REMOTE_ADDR        $remote_addr;
                fastcgi_param  REMOTE_PORT        $remote_port;
                fastcgi_param  SERVER_ADDR        $server_addr;
                fastcgi_param  SERVER_PORT        $server_port;
                fastcgi_param  SERVER_NAME        $server_name;

                try_files $uri =404;
                fastcgi_pass unix:/run/php/php7.0-fpm.sock ;
        }

		# Enables Caching
        location ~* \.(ico|css|js)$ {
			expires 7d;
			add_header Pragma public;
			add_header Cache-Control "public, must-revalidate, proxy-revalidate";
        }
	}

## Create database

Log in to mysql server and create a database. You must have set a password for root user during MySQL installation:

    mysql -u root -p
    create database pydio;

## Now that all the Nginx files are created

We can enable the site by deleting the default Nginx site and linking to the new site;

    cd /etc/nginx/sites-enabled
    rm default
    ln -s ../sites-available/pydio.conf

Time to get the Pydio code. [Download](https://pydio.com/en/get-pydio) the latest version and save it to your /var/www directory. Extract the downloaded file to the root of the www directory;

    cd /var/www
    tar -xzf <gz file name here>
    ls -l
    mv <extracted directory name> pydio
    rm -R /var/www/<gz file name here>
    chown -R www-data:www-data /var/www/pydio

Now open a browser and hit your IP address or DNS name.   The first time you access Pydio you will see a diagnostics page that will look like this.

[:image-popup:system/installing_on_debian+nginx/PydioDiagnosticTool.png]

You will need to fix any issues discovered by the Diagnostics program before continuing. You will notice the warning about SSL Encryption.   I am accessing my server using [Pound](http://www.apsis.ch/pound) as a reverse proxy to encrypt the pages using SSL.

Once you have fixed any errors reported by the Diagnostics page click on the link under the title to continue to the Pydio installation wizard. Use the MySQL credentials and 'pydio' database you just created, and make sure to use the correct URL for the server (like https://your_server_name/), otherwise you may have issues loading public links.

That’s it.   Pydio is now installed and waiting for you to configure workspaces and other customizations. There are [plugins available](https://pydio.com/en/docs/references/plugins) and [client applications](https://pydio.com/products/downloads/)


## PS: Pydio 6 Rewrite Rules

This is the old config.

    server {
            server_name www.example.com;
            listen 80;
            rewrite ^ https://$server_name$request_uri? permanent;
    }
    server {
            server_name www.example.com;
            root /var/www/<extracted directory name>;
            index index.php;
            listen 443 ssl;
            keepalive_requests    10;
            keepalive_timeout     60 60;
            access_log /var/log/nginx/access_pydio6_log;
            error_log /var/log/nginx/error_pydio6_log;

            client_max_body_size 15M;
            client_body_buffer_size 128k;

            rewrite ^/dashboard|^/settings|^/welcome|^/ws- /index.php last;
            if ( !-e $request_filename ) {
                    # WebDAV Rewrites
                    rewrite ^/shares /dav.php last;
                    # Sync client
                    rewrite ^/api /rest.php last;
                    # External users
                    rewrite ^/user ./index.php?get_action=user_access_point last;
                    # Public shares
                    rewrite ^/data/public/([a-zA-Z0-9_-]+)\.php$ /data/public/share.php?hash=$1?;
            }
            rewrite ^/data/public/([a-zA-Z0-9_-]+)--([a-z]+)$ /data/public/share.php?hash=$1&lang=$2?;
            rewrite ^/data/public/([a-zA-Z0-9_-]+)$ /data/public/share.php?hash=$1?;

            # Prevent Clickjacking
            add_header X-Frame-Options "SAMEORIGIN";

            # Only allow these request methods and do not accept DELETE, SEARCH and other methods
            if ( $request_method !~ ^(GET|HEAD|POST|PROPFIND|OPTIONS)$ ) {
                    return 444;
            }

            location ~* ^/(?:\.|conf|data/(?:files|personal|logs|plugins|tmp|cache)|plugins/editor.zoho/agent/files) {
                    deny all;
            }
            # Enables PHP
            location ~ \.php$ {
                    # for ^/(index|plugins) request_uri should be changed
                    set $request_url $request_uri;
                    if ( $uri ~ ^/(index|plugins) ) {
                            set $request_url /;
                    }
                    include fastcgi.conf;
                    fastcgi_param  REQUEST_URI $request_url;
                    fastcgi_param SCRIPT_FILENAME $document_root$fastcgi_script_name;
                    try_files $uri =404;
                    fastcgi_pass unix:/run/php/php7.0-fpm.sock ;
            }

           # Enables Caching
            location ~* \.(ico|css|js)$ {
                     expires 7d;
                    add_header Pragma public;
                    add_header Cache-Control "public, must-revalidate, proxy-revalidate";
            }
    }
