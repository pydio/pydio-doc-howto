### INSTALLING LINUX + NGINX AND PYDIO
After a fresh install of a Linux distribution we will see all the steps that will help you run Pydio on a LINUX/Nginx.
**All the command lines where used on Ubuntu but you have the same thing pretty much on every Linux Distriution so dont forget to adapt it to your configuration if required**

> I advise you to read everything even if you are not using the same configuration, it can help you figure out issues with your installation.

#### CONFIGURATION BEFORE INSTALLING
+ **PHP** : check with this command line if you have any php installed `php -v`if it shows you a line with a php version and that php version is not **php-7.0** or above,
what you can do is remove php using `apt remove php* & apt purge php*`
after that use `apt autoremove`. Now if you re-use `php -v`it should show you no version of PHP.
+ **Apache2** : as usual use `apache2 -v` if it shows you any version of apache use `apt remove apache* & apt purge apache*`
and then `apt autoremove`, now re-check it using `apache2 -v`.

> Dont forget to use SUDO if you're not on root user.

#### INSTALLATION

##### 1. Main Packages
+ **PHP** : to install PHP use `apt install php-7.0`.
+ **NGINX** : to install Nginx use `apt install nginx`.
+ **Mysql** : you can install MariaDB using `apt install mariadb-server`.


##### 2. Additional Packages (Required)

+ **PHP-Mysql** : PHP-Mysql allows php to execute sql, to install this one use `apt install php-mysql`.
+ **PHP-FPM** : PHP-FPM is needed when you're using NginX because it doesn't contain native PHP Processing,
to install PHP-FPM `apt install php-fpm`.
After that open this file with any text editor `/etc/php/7.0/fpm/php.ini`and uncomment this line `cgi.fix_pathinfo=0`.

**For this Part you should go below to Virtual Hosts if you have no idea about how it works**
Don't forget to configure Nginx so that he can use PHP, to do so you need to add the following line :

Open with a text editor your Pydio's sites-available conf file located in `/etc/nginx/sites-available/` then,
You can either uncomment the lines if they are in the file (should be if you used the default one as a template) that will be marked :
```
	location ~ \.php$ {
	#	include snippets/fastcgi-php.conf;                                  THERE'S THIS ONE to uncomment
	#
	#	# With php7.0-cgi alone:
	#	fastcgi_pass 127.0.0.1:9000;
	#	# With php7.0-fpm:
		fastcgi_pass unix:/run/php/php7.0-fpm.sock;                         THIS ONE to UNCOMMENT
		try_files $uri =404;                                                THIS ONE to UNCOMMENT   
		fastcgi_param SCRIPT_FILENAME $document_root$fastcgi_script_name;   THIS ONE to UNCOMMENT
		include fastcgi_params;		                                        THIS ONE 
	}

	# deny access to .htaccess files, if Apache's document root
	# concurs with nginx's one
	#
	#location ~ /\.ht {                                                     YOU CAN ALSO UNCOMMENT THIS ONE
	#	deny all;                                                           and THIS ONE 
	#}                                                                      and THIS ONE
}

```

Otherwise you can add them as the following :
```
location ~ \.php$ {
        include snippets/fastcgi-php.conf;
        fastcgi_pass unix:/var/run/pydio-fpm.sock; (it's your php socket the name can be different)
    }

    location ~ /\.ht {
        deny all;
    }
```

**You can use `nginx -t`to look if the syntax is correct.**

You can also configure PHP to support larger files edit with a text editor `/etc/php.7.0/fpm/php.ini`
```
file_uploads = On
post_max_size = 20G
upload_max_filesize = 20G
max_file_uploads = 20000
output_buffering = Off
```
and edit those values to what suits ur usage.  


> Those Additonal Packages are required to have a working server, but if you know other alternatives you can use them.

##### 3. Install Pydio
Dont forget to create a Database for Pydio `mysql -uroot -p`
and when you're logged in use `CREATE DATABASE <databasename>;`.
Give it privileges with `GRANT ALL PRIVILEGES ON <databasename>.* TO <username>@<server> IDENTIFIED BY '<password>' ;`.

Then to install Pydio you have multiple ways check them **[here](https://pydio.com/en/docs/v8/installation-guide)**.


##### 4. Virtual Hosts Nginx
This part is important if you're going to use Virutal Hosts.

First you need to give the rights to your Pydio `sudo chown -R $USER:$USER /var/www/YourPydio`
it can also be `www-data:www-data`,
then `sudo chmod -R 755 /var/www`.

Now you have to create your config file, you can copy the default one to use it as a template 
`cp /etc/nginx/sites-available/default /etc/nginx/sites-available/YourPydio`.

now open this file with any text editor `/etc/nginx/sites-available/YourPydio`
and change the following lines :
```
server {
        listen 80 default_server; (remove 'default_server' because you're going to have multiple ones)
        listen [::]:80 default_server;(same as ABOVE)

        root /var/www/html; (change it to your Pydio's webserver location webroot /pydio/core/src/core )
        
        index index.html index.htm index.nginx-debian.html; (add index.php allowing the index to also be a php page)

        server_name _; (you can give it a server name such as example.com www.example.com (you can put both entries))

        location / {
                try_files $uri $uri/ =404;
        }

IMPORTANT DONT FORGET TO ADD YOUR PHP SOCKET 
location ~ \.php$ {
        include snippets/fastcgi-php.conf;
        fastcgi_pass unix:/var/run/pydio-fpm.sock; (it's your php socket the name can be different)
    }

    location ~ /\.ht {
        deny all;
    }
}

```
Now create a link to sites-enabled :
`sudo ln -s /etc/nginx/sites-available/YourPydio /etc/nginx/sites-enabled/` 
Use `nginx -t`to verify syntax errors.

>What he have here is just an example you're free to modify it as you want

##### 5. Security
For security you have a lot of choices so identify your needs and refer to guides that can give you details about it,
with Pydio you already have security but you can add a lair to your WebServer, "better safe than sorry".
You can add SSL and such.

#### IF YOU'RE HAVING AN ISSUE
If there's something that wasn't clear or that you may have missed
you can check guides such as :

+ **[Virtual Hosts](https://www.digitalocean.com/community/tutorials/how-to-set-up-nginx-server-blocks-virtual-hosts-on-ubuntu-16-04)**
+ **[LEMP Stack DEBIAN 8](https://www.digitalocean.com/community/tutorials/how-to-install-linux-nginx-mysql-php-lemp-stack-on-debian-8)**
+ **[Nginx/Linux](https://www.digitalocean.com/community/tutorials/how-to-install-nginx-on-ubuntu-16-04)**

Or visit our **[Forum](https://forum.pydio.com/)**.

