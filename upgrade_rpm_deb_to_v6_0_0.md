If Pydio was installed via **deb** or **rpm**, you can not use built-in update function of Pydio.

In this how-to, I will introduce to you steps neccessary to upgrade your system to the new version.

## Backup your system
### Backup database
You can use mysqldump to backup your database

`$ /usr/bin/mysqldump --user=MYSQL_USERNAME --password=MYSQL_PASSWORD MYSQL_DATABASE > backup-pydio-5.2.5.sql`

### Backup pydio files
All php file of Pydio are located in:

+ DEB:
    - /usr/share/pydio
    - /etc/pydio
    - /var/lib/pydio/data
+ RPM:
    - /usr/share/pydio
    - /etc/pydio
    - /var/cache/pydio
    - /var/lib/pydio

You can copy all of them to another location in the case where upgrade fails.

## Upgrade using package manager
For RPM:

    $ yum update pydio

For Debian

    $ apt-get update
    $ apt-get upgrade pydio

Your pydio should now be on version 6.

## Apply modifications manually
### Modify .htaccess
Two .htaccess files must be modified: /usr/share/pydio/.htaccess and /var/lib/pydio/data/public/.htaccess

Attention, depending on your uri and uri for public folder (configured in Main option, public url) you should specify correctly in .htaccess
Sample code:
/usr/share/pydio/.htaccess. Don’t forget to replace YOUR_URI by yours

    # You must set the correct values here if you want
    # to enable webDAV sharing. The values assume that your 
    # Pydio installation is at http://yourdomain/
    # and that you want the webDAV shares to be accessible via 
    # http://yourdomain/shares/repository_id/
    RewriteEngine on
    RewriteBase YOUR_URI
    RewriteCond %{REQUEST_FILENAME} !-f
    RewriteCond %{REQUEST_FILENAME} !-d
    RewriteRule ^shares ./dav.php [L]
    RewriteRule ^api ./rest.php [L]
    RewriteRule ^user ./index.php?get_action=user_access_point [L]
    RewriteCond %{REQUEST_URI} !^/index
    RewriteCond %{REQUEST_URI} !^/plugins
    RewriteCond %{REQUEST_URI} ^/YOUR_URI/dashboard|^/YOUR_URI/settings|^/YOUR_URI/welcome|^/YOUR_URI/ws-
    RewriteRule (.*) index.php [L]

    #Following lines seem to be necessary if PHP is working
    #with apache as CGI or FCGI. Just remove the #
    #See http://doc.tiki.org/WebDAV#Note_about_Apache_with_PHP_as_fcgi_or_cgi

    #RewriteCond %{HTTP:Authorization} ^(.*)
    #RewriteRule ^(.*) - [E=HTTP_AUTHORIZATION:%1]

There is small difference between DEB and RPM is location of public folder

+ DEB: /var/lib/pydio/data/public
+ RPM: /var/lib/pydio/public

.htaccess for public folder. Attention! you must replace YOUR_PUBLIC_URI

    Order Deny,Allow
    Allow from all
    <Files ".ajxp_*">
    deny from all
    </Files>

    RewriteEngine on
    RewriteBase YOUR_PUBLIC_URI
    RewriteCond %{REQUEST_FILENAME} !-f
    RewriteCond %{REQUEST_FILENAME} !-d
    RewriteRule ^([a-zA-Z0-9_-]+)\.php$ share.php?hash=$1 [QSA]
    RewriteRule ^([a-zA-Z0-9_-]+)--([a-z]+)$ share.php?hash=$1&lang=$2 [QSA]
    RewriteRule ^([a-zA-Z0-9_-]+)$ share.php?hash=$1 [QSA]

### Modify database schema
You can download file sql in pydio github and execute by folowing command:

Get the update script for your database from Github.

For example, for mysql, get the file

https://raw.githubusercontent.com/pydio/pydio-core/develop/dist/scripts/misc/5.2.5-6.0.0.mysql and pass it to mysql command:

    $ wget https://raw.githubusercontent.com/pydio/pydio-core/develop/dist/scripts/misc/5.2.5-6.0.0.mysql
    $ /usr/bin/mysql --user=MYSQL_USERNAME --password=MYSQL_PASSWORD MYSQL_DATABASE < 5.2.5-6.0.0.mysql

### Clear Pydio caches
You may face to message “Ooups, this should not happen, your message file is empty!”.
If there is opened connection, the cache will be overrided right after deletion. Then be sure that you have no opened connection (or stop web server for a moment if it’s possible) during the cache clearance.

+ DEB
    - Clear plugin cache: `$ rm -rf /var/lib/pydio/data/cache/*.ser`
    - Clear lang cache: `$ rm -rf /var/lib/pydio/data/cache/i18n/*`
+ RPM
    - Clear plugin cache: `$ rm -rf /var/cache/pydio/*.ser`
    - Clear lang cache: `$ rm -rf /var/cache/pydio/i18n/*`
    - Attention: After execute RPM package, please verify and DELETE 02 plugins in /usr/share/pydio if they are exist:
– /usr/share/pydio/downloader.http
– /usr/share/pydio/auth.cas

### Update ROOT_ROLE
Pydio 6 introduces a new workspace for the “Welcome” page, called “Home”. To make sure that every users can access it by clicking on the Pydio logo, update the ROOT_ROLE access control list manually.

See Capture below.

[:image-popup:upgrades/upgrade_rpm_deb_to_v6.0.0/root_role_config.png]

### [optional] Update Theme
The GUI plugin has a default value for the Theme. When upgrading and clearing the cache, it will reload the default value to switch to the new theme. If you have once saved this plugins parameter, it may still use the old value (Vision). So if you don’t see any difference in your interface, make sure to edit the plugin under Feature Plugins > Action Plugins > Graphical User Interface > Client Driver.

Switch the Theme to Orbit, save, and reload the interface. You should now see the new layout.