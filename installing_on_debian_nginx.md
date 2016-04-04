**Author** Grant Carthew from http://uglygizmo.blogspot.fr/

**Update** Updated on Avr. 4 2016 to update new img for Pydio 6

Following are the instructions for installing [Pydio](https://pydio.com/) with [Nginx](http://nginx.org/en/) on [Debian Jessie](http://www.debian.org/intro/about).   The hardware I am using is a [Raspberry Pi](http://www.raspberrypi.org/faqs) with a [4TB Western Digital USB](http://www.wdc.com/en/products/products.aspx?id=870) hard drive attached.   Because the Raspberry Pi is a low powered device I am using Nginx as the web server with PHP-FPM for processing php.

**Assumptions**

You already have Debian 8.0 (Jessie) running.
You can download the Pydio compressed file to your Debian server.
To start off with we need to install the prerequisite packages.   I am keen to keep my Raspberry Pi lean and so I did some testing to determine the minimum required packages needed to get the full functionality of Pydio.   Note I am not including the requirements for the Pydio desktop client yet because it is in beta and I am not interested in testing it.   If you wish to use the desktop client you will need some rsync php related packages.

First we need to authenticate the nginx repository signature, we add the key used to sign the nginx packages and repository to the apt program keyring. Download this [key](http://nginx.org/keys/nginx_signing.key) and add it to the apt program keyring with the following command:

    apt-key add nginx_signing.key

Then you can add at the end of `/etc/apt/sources.list` file:

    deb http://nginx.org/packages/ubuntu/ jessie nginx
    deb-src http://nginx.org/packages/ubuntu/ jessie nginx

So after that, you can install the prerequisites;

    apt-get update
    apt-get install nginx php5 php5-fpm php5-gd php5-cli php5-mcrypt

Once the prerequisites are installed, create the www directory and set ownership;

    mkdir /var/www
    chown www-data:www-data /var/www

We need to configure php to support larger file uploads so edit the php.ini file;

    vim /etc/php5/fpm/php.ini

Edit the following values to your liking;

    file_uploads = On
    post_max_size = 20G
    upload_max_filesize = 20G
    max_file_uploads = 20000
    output_buffering = Off

Then restart php fpm service

    /etc/init.d/php5-fpm restart

Now we need to configure Nginx to setup our Pydio web site (use your own domain name below);

    vim /etc/nginx/sites-available/x.yourdomain.com

## Here is the x.yourdomain.com config file I am using. 

Make sure you change the max_body_size value and replace x.yourdomain.com with your servers DNS name;

### On Pydio 5

    server {
    listen 80;
    server_name x.yourdomain.com;

    root /var/www;
    index index.php;
    client_max_body_size 20G;
    access_log /var/log/nginx/x.yourdomain.com.access.log;
    error_log /var/log/nginx/x.yourdomain.com.error.log;

    location / {
    }

    location ~* \ .(?:ico|css|js|gif|jpe?g|png)$ {
    expires max;
    add_header Pragma public;
    add_header Cache-Control "public, must-revalidate, proxy-revalidate";
    }

    include drop.conf;
    include php.conf;
    }

Take note of the two include statements at the bottom of the Nginx site file.   You will need to make these files also;

    vim /etc/nginx/drop.conf

And here is my drop.conf contents;

    location = /conf/       { deny all; }
    location = /data/       { deny all; }
    location = /robots.txt  { access_log off; log_not_found off; }
    location = /favicon.ico { access_log off; log_not_found off; }
    location ~ /\ .          { access_log off; log_not_found off; deny all; }
    location ~ ~$           { access_log off; log_not_found off; deny all; }

Note the first two deny all statements above are specific to Pydio.
Now create the php.conf file;

    vim /etc/nginx/php.conf

Here is my php.conf contents;

    location ~ \ .php {
    try_files $uri =404;
    fastcgi_param  QUERY_STRING       $query_string;
    fastcgi_param  REQUEST_METHOD     $request_method;
    fastcgi_param  CONTENT_TYPE       $content_type;
    fastcgi_param  CONTENT_LENGTH     $content_length;
    fastcgi_param  SCRIPT_NAME        $fastcgi_script_name;
    fastcgi_param  SCRIPT_FILENAME    $request_filename;
    fastcgi_param  REQUEST_URI        $request_uri;
    fastcgi_param  DOCUMENT_URI       $document_uri;
    fastcgi_param  DOCUMENT_ROOT      $document_root;
    fastcgi_param  SERVER_PROTOCOL    $server_protocol;
    fastcgi_param  GATEWAY_INTERFACE  CGI/1.1;
    fastcgi_param  SERVER_SOFTWARE    nginx;
    fastcgi_param  REMOTE_ADDR        $remote_addr;
    fastcgi_param  REMOTE_PORT        $remote_port;
    fastcgi_param  SERVER_ADDR        $server_addr;
    fastcgi_param  SERVER_PORT        $server_port;
    fastcgi_param  SERVER_NAME        $server_name;
    fastcgi_pass unix:/var/run/php5-fpm.sock;
    }

The last statement in the above file is the mapping between Nginx and PHP5-FPM.


### On Pydio 6

This is the new config, contributed by Vlad

    server {
            server_name www.example.com;
            listen 80;
            rewrite ^ https://$server_name$request_uri? permanent;
    }
    server {
            server_name www.example.com;
            root /var/www/localhost/htdocs/pydio6;
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
                    try_files $uri =404;
                    fastcgi_pass unix:/tmp/php5-fpm.sock;
            }

           # Enables Caching
            location ~* \.(ico|css|js)$ {
                     expires 7d;
                    add_header Pragma public;
                    add_header Cache-Control "public, must-revalidate, proxy-revalidate";
            }
    }


## Now that all the Nginx files are created

We can enable the site by deleting the default Nginx site and linking to the new site;

    cd /etc/nginx/sites-enabled
    rm default
    ln -s ../sites-available/x.yourdomain.com

Time to get the Pydio files. [Download](https://pyd.io/download/) the latest version and save it to your /var/www directory. Extract the downloaded file to the root of the www directory;

    cd /var/www
    tar -xzf <gz file name here>
    ls -l
    mv <extracted directory name>/* /var/www
    rm -R /var/www/<extracted directory name>
    chown -R www-data:www-data /var/www

Prior to opening a browser and seeing the result we need to restart the required services to pick up the new config files;

`service php5-fpm restart`
`service nginx restart`

Now open a browser and hit your IP address or DNS name.   The first time you access Pydio you will see a diagnostics page that will look like this.

[:image-popup:system/installing_on_debian+nginx/PydioDiagnosticTool.png]

You will need to fix any issues discovered by the Diagnostics program before continuing. You will notice the warning about SSL Encryption.   I am accessing my server using [Pound](http://www.apsis.ch/pound) as a reverse proxy to encrypt the pages using SSL.

Once you have fixed any errors reported by the Diagnostics page click on the link under the title to continue to the Pydio main interface. You will need to log in
with a username of admin and a password of admin for the first access.   Make sure you change the password at some point.

The last required configuration for the installation is to adjust the Pydio upload file size limit. This is achieved under settings;

[:image-popup:system/installing_on_debian+nginx/PydioCoreConfigUploader.png]

That’s it.   Pydio is now installed and waiting for you to configure repositories and other customizations.  There are [plugins available](https://pyd.io/plugins/) and client applications.   I am using the [Android client](https://play.google.com/store/apps/details?id=info.ajaxplorer.android&hl=en) successfully and will look at the [Desktop client](https://pyd.io/extensions/desktop-sync/) once it is out of beta.

The Raspberry Pi is an amazing platform for free open tools like this and I am now using a low powered Pi with a USB disk as my home file server cutting my electricity bill and reducing my carbon foot print.