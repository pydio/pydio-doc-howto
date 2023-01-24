<div style="background-color: #fbe9b7;font-size: 14px;">
<span style="background-color: #fae4a6;padding: 10px;">WARNING</span>
<span style="padding: 10px;display: inline-block;">This article is for Pydio 8 (PHP). Time to move to <a href="https://pydio.com/en/docs/administration-guides">Pydio Cells</a>!</span>
</div>


In this how-to, you will find the steps to do the migration and upgrade from Pydio 6 to pydio 7 on another machine base on CentOS 7.

## Contents
1. Install latest version Pydio on new machine
2. Migrate Database
3. Migrate data folder
4. Test and finalize

## Step 1.
Install Pydio PHP latest version on CentOS 7:

Another option is using virtual machine image (OVF). You can also download this file from https://pydio.com

> Tip: On CentOS 7, there are a lot of issues with selinux, it could be better if you disable selinux during installation & testing. Use this command: `setenforce 0`

## Step 2.

##### 2.1
On new machine, create a new database using following command:

    create database pydioold;
    grant all privileges on pydioold.* to 'pydio'@'localhost';
    flush privileges;

##### 2.2

> Before exporting the db, please try to set Settings > Main Options > Server URL to blank value

On old machine, export all data in DB by using `mysqldump`

    mysqldump -u username -p databasename > pydio6.mysql

##### 2.3

Copy pydio6.mysql file to new machine

##### 2.4

Import to **pydioold** database:

    mysql -u pydio -p pydioold < pydio6.mysql

##### 2.5

Execute some scripts to update **pydioold** database from version 6 to version 7:

https://raw.githubusercontent.com/pydio/pydio-core/develop/dist/php/6.2.0.mysql

https://raw.githubusercontent.com/pydio/pydio-core/develop/dist/php/6.4.0.mysql

https://raw.githubusercontent.com/pydio/pydio-core/develop/dist/php/6.5.3.mysql

https://raw.githubusercontent.com/pydio/pydio-core/develop/dist/php/7.0.0.mysql

https://raw.githubusercontent.com/pydio/pydio-core/develop/dist/php/7.0.2.mysql

> Just copy and paste to terminal of mysql

## Step 3

If you install Pydio by using yum, all data is located in: /var/lib/pydio

On new machine, backup this folder `mv /var/lib/pydio /var/lib/pydio.latest`

Copy recursively /var/lib/pydio from old machine to /var/lib/pydio on new machine.

(Visit this link to set permission correctly: https://pydio.com/en/docs/kb/security/permission-pydio%E2%80%99s-filesfolders)

##### 3.1 

Delete some cache file on new machine: `rm -rf /var/cache/pydio/plugins_*`

##### 3.2

Backup and edit /var/lib/pydio/plugins/boot.conf/bootstrap.json file to use new mysql connection (server, username, password, and database name - **pydioold**) or rename pydioold to other convenient name.

## Step 4

Try to browse pydio by using new machine.
If you have any problem with url, maybe this link is useful: https://pydio.com/en/docs/kb/miscellaneous/fix-public-link-after-upgrading-pydio-7#content

