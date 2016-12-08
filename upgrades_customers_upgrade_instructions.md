# Customers Upgrade instructions

If you are currently running Pydio 6.0.8 along with our existing set of enterprise plugins (core.licence, custom.gui_customize), you can migrate to our new Enterprise Distribution. After the migration, you will not need to maintain custom plugins anymore, as they are directly part of the distribution.
You can also choose to upgrade to the latest community edition and stick to this edition. See at the bottom of the page for that operation.

Upgrading to 6.2.0 Enterprise Distribution
---

This will be a two-step upgrade : first upgrading from your current version to the 6.2.0 « community », and then migrating to « enterprise distribution ». You just have to make sure **not to reload your web interface during the process**, otherwise you will end up in an unstable state.

## Step 1
1.	Grab your API Keys that you will find in your new dashboard on Pydio.com. You will use it later during this process.
2.	Make sure that your whole installation is writeable by the web server during the upgrade.
3.	Login to your admin interface, and open the updater options under Parameters > Features > Actions Plugins > Updater Engine.
4.	Change the update site url from https://pyd.io/update to https://update.pydio.com/pub/. Save and close the plugin parameters.
5.	Click on the upgrade button in the top-right zone of the screen : you should see a package available for upgrading from 6.0.8 to 6.2.0.
6.	Run the upgrade. From now on, make sure to NOT RELOAD YOUR WEB INTERFACE, otherwise you will have a conflict with the previous version of core.licence.

## Step 2
7.	After upgrade, re-open the Updater Engine parameters and switch the update site from https://update.pydio.com/pub/ to https://update.pydio.com/auth/
8.	Change the update channel from « Stable » to « Install Enterprise Distribution ».
9.	Under the update site section, you should now see 2 new fields to set up a login/password for the update site. Here enter your API KEY and SECRET that you got from your dashboard. Save parameters and close.
10.	Click on the upgrade button in the top-right zone of the screen : you should now see anoter package available for migrating to 6.2.0. You may see a popup window asking your for authorization, use the same API KEY / API SECRET
11.	Run this new upgrade. Now you should be able to logout and reload the interface. When logging in again, in the Settings panel, you should discover your new Dashboard.
12.	Enjoy !


Upgrading to 6.2.0 and sticking to community edition
---
If you do not want to install the enterprise distribution, you will need to upgrade the core.licence plugin to a newer version to be compatible. 
Please follow the steps :

1.	Make sure that your whole installation is writeable by the web server during the upgrade.
2.	Download the new core.licence plugin from your new pydio.com account. You should find pydio-core.licence-6.2.0.zip in your dashboard.
3.	Prepare the plugin folder , but DO NOT REPLACE it yet.
4.	Login to your admin interface, and open the updater options under Parameters > Features > Actions Plugins > Updater Engine.
5.	Change the update site url from https://pyd.io/update to https://update.pydio.com/pub/
6.	Click on the upgrade button in the top-right zone of the screen : you should see a package available for upgrading from 6.0.8 to 6.2.0
7.	After upgrade, replace the plugins/core.licence folder with the new version you downloaded before.
