Configuring public folder seems to be complicated and confused to some typical user. Some don’t know where is public folder, some don’t know how to configure “Rewrite Rules” and some others are stuck in blank page in public link. In this how-to, we hope that you can configure public folder easily.

# NIX System

## 1. URI
The most important is URI for Public in Pydio. As you know, there is a place in your system that all user can create a accessible path to file or folder to public where user from outside can download/upload without credential. Due to security reason, this location must be applied a special policy and could be located in other location than data and code of Pydio. Before doing configure public folder, make sure that you have URI for public folder.

### 1.1 Default URI
By default, public folder is shipped with pydio package (PYDIO/data/public). If you installed pydio from apt-get, you can have:
URL for pydio: http://your_server_address/pydio
URL for public: http://your_server_address/pydio/data/public
in this case URI for public is /pydio/data/public. In this case, you can access to public folder because of:
URI /pydio/data/public => VirtualDirectory /pydio in Apache point to /usr/share/pydio. => /usr/share/pydio/data (symbol link) => /var/lib/pydio/data/public

### 1.2 Customized URI for public link
If you don’t want to use default uri for public (because of long string), you can create a new uri by your own:
Create a VirtualDirectory in webserver point to public folder

	alias /public-for-pydio /var/pydio_public
	<Directory "/var/pydio_public">
	Options FollowSymLinks
	AllowOverride Limit FileInfo
	Require all granted
	</Directory>

In this case, the new URI for public is /public-for-pydio and user can access to data by URL: http://your_server_address/public-for-pydio/abcd01.php

## 2. .htaccess file
In Pydio version 5 and ealier, when you create a public share, Pydio will create a new real file in public folder. Since version 6, generation a real file has been no longer used but database. When you request http://your_server_address/public-for-pydio/abcd01.php, module rewrite in web server will convert abcd01.php to an url parameter for share.php – a file auto created in public folder. That means in version 6, there is no .php file in public folder except share.php.

### 2.1 Verify rewrite module
Be sure that rewrite module for web server is enabled.
In debian, you can enable by command:

	sudo a2enmod rewrite
	sudo service apache2 reload

### 2.2 .htaccess file
In this article, I take a example for .htaccess for apache, but you can easily convert .htaccess to rules in Nginx (http://winginx.com/en/htaccess) or web.config in IIS (with import tool)
.htaccess example in public folder /var/pydio_public

	Order Deny,Allow
	Allow from all
	<Files ".ajxp_*">
	deny from all
	</Files>

	RewriteEngine on
	RewriteBase /public-for-pydio
	RewriteCond %{REQUEST_FILENAME} !-f
	RewriteCond %{REQUEST_FILENAME} !-d
	RewriteRule ^([a-zA-Z0-9_-]+)\.php$ share.php?hash=$1 [QSA]
	RewriteRule ^([a-zA-Z0-9_-]+)--([a-z]+)$ share.php?hash=$1&lang=$2 [QSA]
	RewriteRule ^([a-zA-Z0-9_-]+)$ share.php?hash=$1 [QSA]

## 3. Configuration in Pydio
After defining URI for public, and appling rewrite rule in .htaccess, you open Pydio and finish configuration:
– SERVER URL: url for server, in this case is: http://your_server_address/pydio
– DOWNLOAD FOLDER: physical folder for public: /var/pydio_public
– DOWNLOAD URL: url for public folder: http://your_server_address/public-for-pydio

[:image-popup:miscellaneous/configuring_public_link_on_pydio6/pydiopublicconfig.png]

For permission of public folder, you can visit:https://pyd.io/permission-for-pydios-filesfolders/


# Windows

If you are running IIS 7 on windows server 2008, you have to install rewrite module manually.
Go: http://www.iis.net/downloads/microsoft/url-rewrite

In windows 2012 and IIS8, it is enabled

By default, public folder are shipped with pydio package in PYDIO/data/public. But in this article, I will take an general example.
I supposed:
– Pydio location: C:\inetpub\wwwroot\py605
– I need to use C:\inetpub\py605_public for public folder
– We are running this web app by application pool name “py605”.

1. Be sure permission is set correctly (for more information: https://pyd.io/configure-applicationpool-for-pydio-in-windows2012-iis8/)
2. Create a virtual host for public folder

In IIS management, right click and select Add Virtual Directory

[:image-popup:miscellaneous/configuring_public_link_on_pydio6/IIS-add-virtual-directory2.png]

Configure public folder/url in Pydio

[:image-popup:miscellaneous/configuring_public_link_on_pydio6/IIS-configig-public-link.png]

Try to generate a new public share link in Pydio, at this time, you got an error of IIS:

[:image-popup:miscellaneous/configuring_public_link_on_pydio6/IIS-public-not-found.png]

Because you have not configured yet rewrite rule for this folder

– Now, open IIS, click on Virtual Directory “public-for-pydio”

– Click on “rewrite URL”

– On the right menu, click on “Import Rule”

** Note: When you create a public share linke, Pydio will verify and create (if necessary) some default files: share.php, .htaccess, 404 …

Select file C:\inetpub\py605_public\ .htaccess

** Note: RewriteBase is incompatible line, you just delete this line to besure that there is no error. You also can modify rule name by right-click on each rule. At the end of this step, you click on “Apply” on the top of right menu.

[:image-popup:miscellaneous/configuring_public_link_on_pydio6/IIS-public-delete-line-htaccess.png]

Verify the share link you ‘ve create above and you can browse the link correctly

[:image-popup:miscellaneous/configuring_public_link_on_pydio6/IIS-public-good.png]

If you don’t want to follow steps above, you just simple create a C:\inetpub\public-for-pydio\web.config and fill it by:

	<?xml version="1.0" encoding="UTF-8"?>
	<configuration>
	    <system.webServer>
	        <rewrite>
	            <rules>
	                <rule name="Pydio-Public-Folder-Rule-1">
	                    <match url="^([a-zA-Z0-9_-]+)\.php$" ignoreCase="false" />
	                    <conditions logicalGrouping="MatchAll">
	                        <add input="{REQUEST_FILENAME}" matchType="IsFile" ignoreCase="false" negate="true" />
	                        <add input="{REQUEST_FILENAME}" matchType="IsDirectory" ignoreCase="false" negate="true" />
	                    </conditions>
	                    <action type="Rewrite" url="share.php?hash={R:1}" appendQueryString="true" />
	                </rule>
	                <rule name="Pydio-Public-Folder-Rule-2">
	                    <match url="^([a-zA-Z0-9_-]+)--([a-z]+)$" ignoreCase="false" />
	                    <action type="Rewrite" url="share.php?hash={R:1}&amp;lang={R:2}" appendQueryString="true" />
	                </rule>
	                <rule name="Pydio-Public-Folder-Rule-3">
	                    <match url="^([a-zA-Z0-9_-]+)$" ignoreCase="false" />
	                    <action type="Rewrite" url="share.php?hash={R:1}" appendQueryString="true" />
	                </rule>
	            </rules>
	        </rewrite>
	    </system.webServer>
	</configuration>

For more information about permission, visit: https://pyd.io/configure-applicationpool-for-pydio-in-windows2012-iis8/