Using the meta.watch plugin along with the core.notifications features, the users will be able to precisely monitor some activities on any files or folder.


## Add watch feature to workspaces

[:image-popup:plugins/monitoring_files_or_folders/screenshot-2013-06-04-at-23-38-43.png]
 

 

## Activating the Notifications
If not already done at install, and particularly if coming from a previous version of Pydio, you may find yourself with a running install but no notifications active. This feature introduced in V5 a new requirement for a database support, so you must prepare a connexion to either an existing MySQL database, or, if you want to skip this step, you can let Pydio create a simple sqlite3 database. For this last solution, make sure that your PHP has the Sqlite3 extension loaded.

Check in the Global Configurations > Core Configs > Configurations Management that you have a “Core Connexion” configured, using either MySQL or Sqlite3 driver. Then in Global Configurations > Core Configs > Notifications Center, push the “Install SQL Tables” button, and set the notifications to “Enabled”.

## Setting a “watch” on a file or folder
Once all is correctly configured, users should be able to

+ Monitor a file for consultation : will trigger an alert if the file is downloaded or even previewed in the interface
+ Monitor a file for modification : will trigger an alert if the file is modified, either via the GUI or by being re-uploaded
+ Monitor a folder for consultation : will trigger an alert if the folder is “opened” for browsing its content
+ Monitor a folder for modification : will trigger an alert if a file is modified inside this folder. WARNING: currently, this will not be recursive in the subfolders.