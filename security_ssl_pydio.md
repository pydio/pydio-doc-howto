### Installing SSL Security on your webserver for Pydio

#### 1. Lets encrypt
Right now lets encrypt is one of the best ways to have a SSL certificate as Let’s Encrypt is a free and open certificate authority run for the public’s benefit. It is a service provided by the Internet Security Research Group (ISRG) and by that you can use it to secure your Pydio.

To set it up it use the following guides as a base and adapt it with your configuration (such as virutalhosts ...) : 

- [Apache Lets encrypt](https://www.digitalocean.com/community/tutorials/how-to-secure-apache-with-let-s-encrypt-on-ubuntu-16-04)
- [Nginx Lets encrypt](https://www.digitalocean.com/community/tutorials/how-to-secure-nginx-with-let-s-encrypt-on-ubuntu-16-04)

- [Apache/Nginx Lets encrypt](https://www.vultr.com/docs/setup-letsencrypt-on-linux)

It should be really easy if you follow the steps.

#### 2. Self Created certificate
For this example we will see how to use a Self create certificate to use HTTPS protocol.

>In this guide the example will be realized using our self created certificate but if you buy one or use lets encrypt it should be pretty much the same.

Then lets begin : 

To create your self certificate use this command : `sudo openssl req -x509 -nodes -days 365 -newkey rsa:2048 -keyout /etc/ssl/private/apache-selfsigned.key -out /etc/ssl/certs/apache-selfsigned.crt`.

When you put this command you will have prompts asking you for informations pretty much all of them are optionnal but there's one that you mush fill,
**Common Name (e.g. server FQDN or YOUR name) []:** and then put your IP Address or Domain.

You can also use a [Diffie-Hellman](https://en.wikipedia.org/wiki/Diffie%E2%80%93Hellman_key_exchange) group to secure it even more, to do so use this command `sudo openssl dhparam -out /etc/ssl/certs/dhparam.pem 2048`.

After that you will have to configure Apache to use SSL.
Create it using `sudo nano /etc/apache2/conf-available/ssl-params.conf`

```
# from https://cipherli.st/
# and https://raymii.org/s/tutorials/Strong_SSL_Security_On_Apache2.html

SSLCipherSuite EECDH+AESGCM:EDH+AESGCM:AES256+EECDH:AES256+EDH
SSLProtocol All -SSLv2 -SSLv3
SSLHonorCipherOrder On
# Disable preloading HSTS for now.  You can use the commented out header line that includes
# the "preload" directive if you understand the implications.
#Header always set Strict-Transport-Security "max-age=63072000; includeSubdomains; preload"
Header always set Strict-Transport-Security "max-age=63072000; includeSubdomains"
Header always set X-Frame-Options DENY
Header always set X-Content-Type-Options nosniff
# Requires Apache >= 2.4
SSLCompression off 
SSLSessionTickets Off
SSLUseStapling on 
SSLStaplingCache "shmcb:logs/stapling-cache(150000)"

SSLOpenSSLConfCmd DHParameters "/etc/ssl/certs/dhparam.pem"
```

Set the SSLOpenSSLConfCmd DHParameters directive to point to the Diffie-Hellman file.

Now enable this file on apache using `sudo a2enconf ssl-params`.

Next we need to modify your Pydio's conf file located in sites-available.

Your file should look like this. 

>This is a basic configuration.

```
<IfModule mod_ssl.c>
        <VirtualHost _default_:443>
                ServerAdmin webmaster@localhost
                ServerName <YourServerName>
                DocumentRoot /var/www/<YourPydioROOT>

                ErrorLog ${APACHE_LOG_DIR}/error.log
                CustomLog ${APACHE_LOG_DIR}/access.log combined
                SSLEngine on
                SSLCertificateFile /etc/apache2/ssl/apache.crt
                SSLCertificateKeyFile /etc/apache2/ssl/apache.keyy

                <Directory /var/www/YourPydioROOT>
                    Allowoverride ALL
                    Require all granted
                </Directory>    
                
                
                <FilesMatch "\.(cgi|shtml|phtml|php)$">
                                SSLOptions +StdEnvVars
                
                </FilesMatch>
                <Directory /usr/lib/cgi-bin>
                                SSLOptions +StdEnvVars
                </Directory>

                # BrowserMatch "MSIE [2-6]" \
                #               nokeepalive ssl-unclean-shutdown \
                #               downgrade-1.0 force-response-1.0

        </VirtualHost>
</IfModule>
```
after that enable your conf file using `sudo a2ensite <Pydio_Conf_File>`.


> If you want to test your Syntax on apache you can do it by using the following command :
 `sudo apache2ctl configtest`

Now you have to enable the headers & ssl mod in apache using :
- `sudo a2enmod ssl`
- `sudo a2enmod headers`

After all of that just restart apache `sudo systemctl apache2 restart`.

Now you can access your Pydio through HTTPS protocol like the following,
`https://YourPydio/`it can be an ip address or domain name of course.

You can also use HTTP, if you want to only be able to use HTTPS you have to create a redirection.
To do so first add this line to your virtualhost/pydio_conf_file :
`Redirect permanent "/" "https://your_domain_or_IP/"`
dont forget to check the syntax using the command above.

**Note : if you're using a firewall do not forget to allow access to apache full.**

For this example the certificate was self made so your browser doesn't trust it because it wasnt made by the **[trusted certificate authorities](https://en.wikipedia.org/wiki/Certificate_authority)** so just validate it yourself on your browser when you will be prompted.

You can use this as a base to how to put SSL for Pydio.
