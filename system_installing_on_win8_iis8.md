**Author** Florian BURNEL from [http://www.it-connect.fr]

**Contributor** JR for the Authorization Rules [chiselapp]

**TO READ FOR V6** Make sure to check https://pydio.com/en/docs/kb/security/configure-applicationpool-pydio-windows2012-iis8-update-pydio-6 to grab the most up-to-date web.config instructions for Pydio6.

## Introduction
In this how-to, we’ll see the installation of Pydio (Put Your Data In Orbit), which is a good alternative to Dropbox, Drive, or ownCloud. In this case, the installation will be carried on a Microsoft IIS 8 webserver, which in this case, is installed on Windows 8.

## Prerequisites
Before you begin, verify that you have the following prerequisites :

+ IIS 8 webserver is installed

+ PHP engine is integrated at the IIS 8 webserver

+ MySQL is already installed (it is recommended, but only necessary if you want use Pydio with a database)

+ Pydio : You must [download the Pydio source](https://pyd.io/download/) before starting this how-to.

## Prepare your IIS webserver
First off, we will prepare the IIS webserver to have the proper configuration to serve Pydio. Open the IIS manager on your server.

### Prepare the site
Do a right click on « **Sites** » then « **Add Website** », you will obtain the form present in the screenshot below.

[:image-popup:system/installing_on_win8_iis8/pydioiisen1.png]

Complete the different fields :

+ **Site name** : Give a name to your site; it will be displayed in the IIS manager

+ **Physical path** : Root path of the site

+ **Type** : Use HTTP, or HTTPS if you have an SSL certificate

+ **Port** : Indicate 80 for HTTP, 443 for HTTPS, or another port if you’d prefer.

Complete the other fields if necessary. Once the configuration is complete, click on “**OK**” to finish the creation.

Now, open the Pydio archive that you previously downloaded. Then, unzip the contents of the pydio-core-5.x.x directory to the root of your website. Example:

[:image-popup:system/installing_on_win8_iis8/pydioiisen2.png]

### Requests Filtering
 [:image-popup:system/installing_on_win8_iis8/pydioiisen3.png] For security reason, it’s necessary to prohibit access to the “**data**” directory of Pydio. To do this, you can either place the data directory out of the Pydio website’s root, or you can create an IIS rule to prevent access, we will do the latter. For this, you must have the “**Request Filtering**” functionality in IIS (the name of this functionality is dependent on the version). Once in the Pydio site, click on the icon which is presented to the left of this tutorial, and click on “**Add hidden segment**” on the right in the IIS window, shown below. Fill the field “**data**”, then confirm to create the protection rule.

[:image-popup:system/installing_on_win8_iis8/pydioiisen4.png]

If not already accessible, you may have to add this module to IIS. To do this, activate the “**Request Filtering**” by “adding features”.

[:image-popup:system/installing_on_win8_iis8/pydioiisen5.png]

Once the addition is complete, you can perform the operation described previously.

### Authorization Rules
Using Request Filtering keeps the public folder from working in its default location.  We can use **Authorization Rules** to disallow access to certain subfolders, which allows overriding at a lower level.  Thus, let’s “deny all” at the data folder, and “allow all” at the public folder.  See below for as sample **_web.config_** files and their location using \ as the root installation location of Pydio.

+ <authorization> tags are for Authorization Rules.
+ <httpErrors> is to change the default error page for the public folder
+ <requestFiltering> is to deny access to “.ajxp_” files in the public folder, though this has not been tested.

**\conf\web.config**

	<?xml version="1.0" encoding="UTF-8"?>
	<configuration>
	<system.webServer>
	<security>
	<authorization>
	<remove users="*" roles="" verbs="" />
	<add accessType="Deny" users="*" />
	</authorization>
	</security>
	</system.webServer>
	</configuration>

\data\web.config

	<?xml version="1.0" encoding="UTF-8"?>
	<configuration>
	<system.webServer>
	<security>
	<authorization>
	<remove users="*" roles="" verbs="" />
	<add accessType="Deny" users="*" />
	</authorization>
	</security>
	</system.webServer>
	</configuration>

\data\public\web.config

	<?xml version="1.0" encoding="UTF-8"?>
	<configuration>
	<system.webServer>
	<security>
	<authorization>
	<remove users="*" roles="" verbs="" />
	<add accessType="Allow" users="*" />
	</authorization>
	<requestFiltering>
	<filteringRules>
	<filteringRule name="Pydio Public" scanUrl="true" scanQueryString="false">
	<denyStrings>
	<clear />
	<add string=".ajxp_" />
	</denyStrings>
	<scanHeaders>
	<clear />
	</scanHeaders>
	<appliesTo>
	<clear />
	</appliesTo>
	</filteringRule>
	</filteringRules>
	</requestFiltering>
	</security>
	<httpErrors>
	<remove statusCode="404" subStatusCode="-1" />
	<error statusCode="404" prefixLanguageFilePath="" path="/data/public/404.html" responseMode="ExecuteURL" />
	</httpErrors>
	</system.webServer>
	</configuration>

\files\plugins\editor.zoho\agent\files\web.config

	<?xml version="1.0" encoding="UTF-8"?>
	<configuration>
	<system.webServer>
	<security>
	<authorization>
	<remove users="*" roles="" verbs="" />
	<add accessType="Deny" users="*" />
	</authorization>
	</security>
	</system.webServer>
	</configuration>

\files\plugins\editor.pixlr\web.config

	<?xml version="1.0" encoding="UTF-8"?>
	<configuration>
	<system.webServer>
	<rewrite>
	<rules>
	<rule name="PixlrSave" stopProcessing="true">
	<match url="^fake_save_pixlr_(.*).php$" />
	<action type="Rewrite" url="fake_save_pixlr.php" />
	</rule>
	</rules>
	</rewrite>
	</system.webServer>
	</configuration>

\files\web.config

	<?xml version="1.0" encoding="UTF-8"?>
	<configuration>
	<system.webServer>
	<rewrite>
	<rules>
	<rule name="Pydio WebDAV" stopProcessing="true">
	<match url="^shares" ignoreCase="false" />
	<conditions>
	<add input="{REQUEST_FILENAME}" matchType="IsFile" ignoreCase="false" negate="true" />
	<add input="{REQUEST_FILENAME}" matchType="IsDirectory" ignoreCase="false" negate="true" />
	</conditions>
	<action type="Rewrite" url="./dav.php" />
	</rule>
	<rule name="Pydio API_REST" stopProcessing="true">
	<match url="^api" ignoreCase="false" />
	<action type="Rewrite" url="./rest.php" />
	</rule>
	</rules>
	</rewrite>
	</system.webServer>
	</configuration>
 

### [optional] Dedicated Windows User
Within IIS, you can convert /pydio to an Application, so that you can configure it to run under a different user account than the DefaultAppPool. Create a specific user for the application to run as that is just a local user with no group memberships.  Give this user read-only rights everywhere under /pydio, and add Modify rights (read/write/delete) to the data folder.

When doing an upgrade, add Modify rights at the root level and then remove it after you finish the in-Pydio upgrade.

### PHP configuration
Now, we will configure PHP to prevent Pydio’s “**PHP Output Buffer disabled**” warning on the diagnostic page. Click on “**PHP Manager**” then “**Manage all parameters**”.

[:image-popup:system/installing_on_win8_iis8/pydioiisen6.jpg]

Use the search area to find the parameter “**output_buffering**”. Double click on the parameter and write the value to “**off**”. The website php.net indicates:

**_“You can enable output buffering for all files by setting this directive to ‘On’. If you wish to limit the size of the buffer to a certain size – you can use a maximum number of bytes instead of ‘On’, as a value for this directive (e.g., output_buffering=4096). As of PHP 4.3.5, this directive is always Off in PHP-CLI.”_**

##Installing the application

The server is ready, the site is online, and now we can install Pydio.

With your favorite browser, go to your Pydio website. The diagnostics page will appear named “**Pydio Diagnostic Tool**”. Double click on “**click here to continue to Pydio**” – Unless you get one or more critical errors, but if you followed this tutorial properly, there won’t be a problem.

Regarding the “**SSL Encryption**” advertisement: It’s recommended to use a HTTPS connection on your website, so you must use HTTPS in your IIS site and have a certificate to secure exchanges between the client and the server.

If you want to activate the automatic redirection to HTTPS on your site, modify the file “**conf/bootstrap_conf.php**” of Pydio and remove comment:

define(“AJXP_FORCE_SSL_REDIRECT”, true);

Start the setup with the area “**Admin access**”, set a name for the Administrator account, a display name for this account, and enter a passphrase.

[:image-popup:system/installing_on_win8_iis8/pydioiisen7.png]

Go to “**Global options**” and choose “**English**” for the “**Default Language**”.

[:image-popup:system/installing_on_win8_iis8/pydioiisen8.png]

This next step is the trickiest. Indeed, to store data, Pydio can use a database but it isn’t an obligation. If you want to do an installation without database, go to step 1 below. If you want to do an installation with a database, go to step 2.

1. **Without a database**

Without a database, no configuration is necessary, you must only select “**No Database (Quick Start)**” and continue.

[:image-popup:system/installing_on_win8_iis8/pydioiisen9.png]

2. **With database**

You should already have a MySQL Server installed on your machine if you followed the prerequisites. If so, Open a CMD to establish a connection with the MySQL Server (make sure you cd to the bin directory of MySQL). Run the following command:

	mysql.exe –u root –p

You get this :

[:image-popup:system/installing_on_win8_iis8/pydioiisen10.png]

Then, for example I have decided to create a database named “**itconnectpydio**” for Pydio only, with a user named “**pydio**”, which has “password” as its passphrase. This is just an example. You should change these defaults in your environment.

The user “**pydio**” must have all privileges on tables in the database because it must manage the data stored in it. Enter the following commands:

**# Create the database**

	CREATE DATABASE itconnectpydio;

**# Create the user**

	CREATE USER "pydio"@"localhost" IDENTIFIED BY "password";

**# Assign rights**

	GRANT ALL PRIVILEGES ON itconnectpydio.* TO "pydio"@"localhost";

Then, go to the webpage of Pydio configuration, choose “Database” for “Storage Type” and “MySQL” for “Database”. After, complete the fields to establish a connection with the MySQL server.

Once that is complete, click on “Try connecting to the database” to verify the connection between Pydio and the SQL server.

If the configuration is correct, you will have this message: “**Connection established!**”

[:image-popup:system/installing_on_win8_iis8/pydioiisen11.png]

Testing connection parameters
This is the last step, create one or many users different of the account “Administrator” (optional). Then, click on “**Install Pydio Now!**”

5. **First connection**

When the installation is complete, you will be redirected on the login page of Pydio. Use the account “**Administrator**”, or another account that you created in the previous step.

[:image-popup:system/installing_on_win8_iis8/pydioiisen13.png]

You are now connected and ready to use Pydio!