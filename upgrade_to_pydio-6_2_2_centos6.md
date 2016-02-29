# Cent OS 6 / Redhat 6

On centos, it requires to use some extra repositories for php54 and its dependencies. Execute following command to add such repositories

`rpm -ivh https://www.softwarecollections.org/en/scls/rhscl/php54/epel-6-x86_64/download/rhscl-php54-epel-6-x86_64.noarch.rpm`

`rpm -ivh https://www.softwarecollections.org/en/scls/remi/php54more/epel-6-x86_64/download/remi-php54more-epel-6-x86_64.noarch.rpm`

`yum install epel-release`

### Install pydio repositories:

**Community version**

`wget https://download.pydio.com/pub/linux/centos/6/pydio-release-1-2.el6.noarch.rpm`

`rpm -i pydio-release-1-2.el6.noarch.rpm`

**Enterprise version**

`wget https://API_KEY:API_SECRET@download.pydio.com/auth/linux/centos/6/x86_64/pydio-enterprise-release-1-2.el6.noarch.rpm`

> Note: Enterprise version requires two repositories.

where you would replace API_KEY / API_SECRET by the values retrieved from your pydio.com account.

`rpm -i pydio-enterprise-release-1-2.el6.noarch.rpm`

`yum update`

> Note : If you're using CentOS v6.2 you may encounter yum update failure blaming “qpid-cpp”, this is actually a bug in yum, and it’s been fixed in newer versions of CentOS. All you need to do is to uninstall and install qpid-cpp packages.
- `yum erase qpid-cpp-server qpid-cpp-client`
- `yum install qpid-cpp-server qpid-cpp-client`


The following commands are applied for installation of new pydio and for doing upgrade from Pydio 6.0 to latest version

`yum install pydio-all`

`yum install pydio-enterprise`

Activate php54

`source /opt/rh/php54/enable`

Disable php5.3
`mv /etc/httpd/conf.d/php.conf /etc/httpd/conf.d/php.conf.bak`

Disable old php (5.3)

`mv /etc/httpd/conf.d/php.conf /etc/httpd/conf.d/php.conf.old`

> Note: the php.ini file is located in: /opt/rh/php54/root/etc/php.ini

Restart apache service

`service httpd restart`

**Manually upgrade Database**
Use this command to connect to MySQL database:

`mysql -u pydio -p`

> Tip: You can get the current password for *pydio* account in mysql by executing following command in your server:
`sudo cat /var/lib/pydio/plugins/boot.conf/bootstrap.json | grep mysql_password`

Then execute the script in this link to upgrade your database:

https://raw.githubusercontent.com/pydio/pydio-core/develop/dist/php/6.2.0.mysql
