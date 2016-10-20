# Ubuntu 14.04 (trusty)

## Step 1: Preparation before upgrade
**1.1. Add repo source.**

Firstly,  install some package
`sudo apt-get install apt-transport-https`

Secondly, please disable old pydio source if it existes in /etc/apt/source.list of your server. Then add the source according to the version you are going to install:

*repo source pydio community*:

`deb https://download.pydio.com/pub/linux/debian/ trusty main`

*repo source pydio enterprise*:

`deb https://API_KEY:API_SECRET@download.pydio.com/auth/linux/debian/ trusty non-free`

> Note: Enterprise version requires two repositories.

where you would replace API_KEY / API_SECRET by the values retrieved from your pydio.com account.

**1.2 Manually update DB**

Use this command to connect to MySQL database:

`mysql -u pydio -p`

> Tip: You can get the current password for *pydio* account in mysql by executing following command in your VM:
`sudo cat /var/lib/pydio/data/plugins/boot.conf/bootstrap.json | grep mysql_password`

Then execute the script in this link to upgrade your database:

https://raw.githubusercontent.com/pydio/pydio-core/develop/dist/php/6.2.0.mysql

and this script if you want to upgrade to Enterprise version
<pre>
CREATE TABLE IF NOT EXISTS `ajxp_session` (
  `id` int(11) NOT NULL AUTO_INCREMENT,
  `session_name` varchar(50) NOT NULL,
  `session_id` varchar(60) NOT NULL,
  `user` varchar(255) NOT NULL,
  `crt_repository` varchar(32) NOT NULL,
  `ip` varchar(45) NOT NULL,
  `ua` mediumtext NOT NULL,
  `status` int(11) NOT NULL DEFAULT '1',
  `updated` int(11) NOT NULL,
  PRIMARY KEY (`id`),
  UNIQUE KEY `session_name_2` (`session_name`,`session_id`),
  KEY `session_user` (`user`),
  KEY `session_status` (`status`),
  KEY `updated` (`updated`)
) DEFAULT CHARSET=utf8 COLLATE=utf8_unicode_ci;
</pre>

## Step 2: Upgrade

Commands:
- `apt-get update`
- `apt-get install pydio-all`
- `apt-get install pydio-enterprise`

## Step 3: After upgrade
3.1. Execute following command to disable pydio.conf (created after installation)

 `sudo a2disconf pydio.conf`

 `sudo service apache2 restart`

3.2. Go to Pydio >> Settings >> Application Core:  DOWNLOAD URL: /pydio/data/public (default value)
