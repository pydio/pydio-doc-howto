The following assumes you are running Pydio on a Windows based machine with Apache server. Adjust the parameters below according to your configuration, if necessary.

1. Open a Command Prompt (Start>Run>cmd.exe) and browse to your Apache /bin directory:
		cd c:/apache2.2.8/bin

	Change “apache2.2.8” above to match your current version/directory, if needed.

2. Create a server key with a 1024 bit encryption. Enter this command:
		openssl genrsa -des3 -out server.key 1024

	You will be asked for a pass phrase.

3. Remove the pass phrase from the RSA private key (backup the original file) and enter this command:
		copy server.key server.key.org

	Then, enter:

		openssl rsa -in server.key.org -out server.key

	Type the pass phrase again, when prompted.

4. Create a self-signed (X509) certificate with the RSA key you just made. Enter this command and follow on-screen prompts:
		openssl req -new -x509 -nodes -days 365 -key server.key -out


		server.crt -config C:apache2.2.8confopenssl.cnf

	* Change “apache2.2.8” above to match your current version/directory, if needed.
	** Change the path to “openssl.cnf”, if needed.

5. Create two folders in the C:apache2.2.8conf directory.
		/ssl.key


		/ssl.crt

6. Copy the “server.key” file to the /ssl.key folder and the “server.crt” file to the /ssl.crt folder.
7. Open “httpd.conf” in your Apache /conf and make the following edits:
	[FIND] – Line 119

		#LoadModule ssl_module modules/mod_ssl.so

	[REPLACE WITH]

		LoadModule ssl_module modules/mod_ssl.so

	Uncomment following line
	# Secure (SSL/TLS) connections
	Include conf/extra/httpd-ssl.conf

8. Open php.ini in C:apache2.2.8bin.
	Uncomment line :

		extension=php_openssl.dll

9. Open httpd_ssl.conf in your Apache /conf/extra directory.
	[FIND] – Line 77

		DocumentRoot C:/Program Files/Apache Software Foundation/Apache2.2/htdocs

	[REPLACE WITH]

		DocumentRoot C:/apache2.2.8/htdocs

	[FIND] – Lines 80-81

		ErrorLog C:/Program Files/Apache Software Foundation/Apache2.2/logs/error.log

		TransferLog C:/Program Files/Apache Software Foundation/Apache2.2/logs/access.log

	[REPLACE WITH]

		ErrorLog logs/sslerror.log

		TransferLog logs/sslaccess.log

	[FIND] – Line 99

		SSLCertificateFile C:/Program Files/Apache Software Foundation/Apache2.2/conf/server.crt

	[REPLACE WITH]

		SSLCertificateFile conf/ssl.crt/server.crt

	[FIND] – Line 107

		SSLCertificateKeyFile C:/Program Files/Apache Software Foundation/Apache2.2/conf/server.key

	[REPLACE WITH]

		SSLCertificateKeyFile conf/ssl.key/server.key

	[FIND] – Lines 193-195

		<Directory C:/Program Files/Apache Software Foundation/Apache2.2/cgi-bin>
		SSLOptions +StdEnvVars
		</Directory>

	[REPLACE WITH]

		<Directory C:/apache2.2.8/htdocs>
		Options Indexes FollowSymLinks MultiViews
		AllowOverride All
		Order allow,deny
		allow from all
		</Directory>

	[FIND] – Line 228 (may be commented out)

		CustomLog C:/Program Files/Apache Software Foundation/Apache2.2/logs/ssl_request.log

	[REPLACE WITH]

		CustomLog logs/ssl_request.log

10. In the previous Command Prompt, enter:
		httpd -t

	If it says Syntax is OK, proceed to step next. If not, correct the syntax (previous steps) and repeat.

11. Restart Apache server. Open a browser and enter [localhost] (without quotes). Be sure all referenced log files in the steps above are created in the respective directories.
12. If behind a router, make sure port 443 is forwarded to your computer. Also, make sure any firewalls are configured to allow incoming connections from port 443.
13. **_Optional_**: If you want to allow world wide web access to your HTTPS secure server, open httpd_ssl.conf:
	[FIND] – Line 78

		ServerName localhost:443

	[REPLACE WITH]

		ServerName yourdomain.com:443

	* Insert your FQDN (Fully Qualified Domain Name) above, or if you don’t have one, use your WAN IP (e.g. 12.34.567.890:443).

14. **_Optional_**: To direct all visits from http to https, create a file .htaccess in your /www folder with the following text:
		RewriteEngine On
		RewriteCond %{HTTPS} off
		RewriteRule (.*) [url]https://[/url]%{HTTP_HOST}%{REQUEST_URI}

15. **_Optional_**: To avoid browser warnings about self-signed (your own Certificate Authority (CA)) SSL certificates, consider purchasing one from GoDaddy for around $29.