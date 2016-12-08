As the legacy ajaxplorer[stable|testing|source) repositories for both Debian and Linux are now discontinued, if you have installed the application from these, you are invited to install Pydio instead. The procedure is fairly simple and should leave you with a migrated install of pydio.

In most case, this will happen when you want to migrate from AjaXplorer 5.0.4 (last stable in these repo) to Pydio 5.2.0 .


## Prerequisites
We assume that you have already installed the repositories on your machine. Make sure to update the repo lists to see both ajaxplorer & pydio listed in the available sources.

EL:

    sudo yum update

Debian :

     sudo apt-get update

You should see pydio-stable/testing repositories in the list.

If your current install is SQL-based, backup your database, and prepare the necessary credentials to connect to it (the ones you used in your ajaxplorer install).

## Install Pydio
EL:

     sudo yum install pydio

Debian :

     sudo apt-get install pydio

## AjaXplorer 5.0.4 >> Pydio 5.2.0 : upgrade database manually
Connect to your database

    $ mysql -u username -p
    > [enter password]
    > use ajaxplorer;
    > Copy / Paste the following lines:
    /* SEPARATOR */
    ALTER TABLE `ajxp_feed` ADD COLUMN `index_path` MEDIUMTEXT NULL;
    /* SEPARATOR */
    ALTER TABLE `ajxp_simple_store` ADD COLUMN `insertion_date` TIMESTAMP DEFAULT CURRENT_TIMESTAMP;
    /* SEPARATOR */
    ALTER TABLE  `ajxp_simple_store` CHANGE  `serialized_data`  `serialized_data` LONGBLOB NULL DEFAULT NULL ;
    /* SEPARATOR */
    ALTER TABLE  `ajxp_plugin_configs` CHANGE  `configs`  `configs` LONGBLOB NOT NULL ;
    /* SEPARATOR */
    ALTER TABLE  `ajxp_user_prefs` CHANGE  `val`  `val` BLOB NULL DEFAULT NULL ;
    /* SEPARATOR */
    ALTER TABLE  `ajxp_repo_options` CHANGE  `val`  `val` BLOB NULL DEFAULT NULL ;
    /* SEPARATOR */
    ALTER TABLE  `ajxp_log` CHANGE  `remote_ip`  `remote_ip` VARCHAR( 45 ) NULL DEFAULT NULL ;
    /* SEPARATOR */
    CREATE TABLE IF NOT EXISTS ajxp_user_teams (
    team_id VARCHAR(60) NOT NULL,
    user_id varchar(255) NOT NULL,
    team_label VARCHAR(255) NOT NULL,
    owner_id varchar(255) NOT NULL,
    PRIMARY KEY(team_id, user_id)
    );

## Copy AjaXplorer data to Pydio

    $ cp -R /var/lib/ajaxplorer/* /var/lib/pydio

## Fixing shared links
The generated PHP files for shared links (inside the public folder) do contain a reference to the full path of pydio: each file starts with the following line:
`require_once("/usr/share/ajaxplorer/publicLet.inc.php");`
But modifying these files will make them invalid as there is a sanity check of the file before execution.

So a fix should be to create a symlink from /usr/share/ajaxplorer pointing to /usr/share/pydio to avoid a require error. Rename the original /usr/share/ajaxplorer folder to something else for the sake of backups, and create a link here pointing to /usr/share/pydio

    cd /usr/share
    mv ajaxplorer ajaxplorer.backup
    ln -s /usr/share/pydio ajaxplorer

## Setup Apache aliases as required
Either you keep the default configuration, thus pydio will be available on http://yourserver/pydio, or you have to update the apache alias definitions files to replace make the legacy alias ajaxplorer point to pydio folder. See /etc/httpd/conf.d/ for EL6 or /etc/apache2/sites-enabled/ for Debian.

 

You should be all set!