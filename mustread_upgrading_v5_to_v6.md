This is a summary of common issues that can appear during upgrade. It is a copy of original forum thread https://pyd.io/f/topic/troubleshooting-upgrade-to-v6/

## Upgrade to v6 troubleshooting
Let’s use this forum thread to summarize and discuss potential issues while upgrading
First, a list of resources:
- Release note: https://pyd.io/pydio-core-6-0-0/
- Product tour: https://pyd.io/pydio-6-tour/

Please post as a comments your issues, i’ll reedit this thread accordingly.

**1/ I don’t see “upgrade available” in the app**

Make sure that your Update Engine plugin is correctly configured: if you are running 5.2.5, must be poiting to Stable channel. If you are running 5.3.4, must be pointing to dev channel.

**2/ I can’t upgrade because I’m using APT-GET or YUM – Or I did upgrade using apt-get/yum update, but I have a blank page**

Did you read https://pyd.io/upgrade-pydio-5-2-5-to-6-0-0/ ?

**2.1/ Error : The root install path is not writeable, no file will be copied! The archive is available on your server, you can copy its manually to override the current installation.**

The simplest way should be to temporarily change permissions on your Pydio install, perform upgrade from the interface, and revert them again. Basically, you can `chown -R www-data` (or httpd) the whole folder, do the upgrade, then revert to whatever the user was, and then again reopen the write permission on the /data/ folder.

**3/ After upgrading from a serial-based pydio5, i cannot access my workspaces anymore**

You see an error about “wrong SQL setup for meta plugins”. Open the file conf/bootstrap_repositories.php and comment out the lines of the default repositories where you see `"meta.syncable" => array()`, by simply adding a double slash at the beginning `// "meta.syncable" => array()`, . This should solve the problem.

Also, you’ll have to change the “Access.fs” plugin options. In the admin panel, in the plugins settings, look for Workspaces Drivers >> FileSystem Standard (access.fs), edit the global parameters of this plugin: remove “meta.syncable” from the Default Meta Sources here. Otherwise, each time you will create a workspace with this driver, the meta.syncable will be added automatically.

Please note that you then cannot use the PydioSync with such a setup.

**4/ After upgrading, I still see the old theme, and not the beautiful one as on your demo!**

Make sure to switch to the new ORBIT theme: under admin panel, in Features Plugins > Graphical User Interface > Client Driver, switch the theme to “Orbit”.

**4.1/ I updated my DEB/RPM and manually applied the DB upgrade as explained in the how-to, but I now see DB errors like ‘Unknown column ‘index_path’ in ‘where clause’’**

This is because another manual update was necessary when upgrading to v5.2.0, and you probably have not applied it. And it was probably transparent because the new columns were not used unless you activated some features, which we did in v6. So please go to https://github.com/pydio/pydio-core/blob/develop/dist/php/5.2.0.sql (or .sqlite) and apply this as well.

**5/ Help! White Page after upgrade and no errors in logs [FIXED in 6.0.1]**

If you see a blank page, and let’s say that your server is installed at http://yoururl.com/my_pydio , open the Web Developers console of your browser, and you see errors loading JS and CSS files. If you look carefully at the problematic urls, you can see that they are pointing to /plugins/gui.ajax/etc… instead of /my_pydio/plugins/gui.ajax etc.
You probably have manually set the Server URL in your pydio core options, and the base of the application is detected as /my_pydio instead of /my_pydio/ (the last slash is important).

Open the file plugins/gui.ajax/class.AJXP_ClientDriver.php and add the following line just after line 144:

    if(!empty($configUrl)){
        $root = '/'.ltrim(parse_url($configUrl, PHP_URL_PATH), '/');
        // ADD THIS LINE
        if(strlen($root) > 1) $root = rtrim($root, '/').'/';
        // END
    }else{

Now reload the interface.

**6/ New Shares are not working (404)**

New shares now REQUIRE that you have apache mod_rewrite enabled (or equivalent for other webservers). Also, the upgrade should have modified the .htaccess located in data/public/, but to make sure, you can remove the file data/public/grid_t.png inside this folder, and create a new share. This should regenerate the htaccess file.

**7/ Php big errors like “Parse error: syntax error, unexpected T_STRING, expecting T_CONSTANT_ENCAPSED_STRING …**

Make sure to use php5.3 or higher.

**8/ Various errors with ldap_control_paged_result_response()  [FIXED in 6.0.1]**

We introduced the usage of new PHP features to help browsing pages on an ad/ldap server (functions added in php5.4). But it seems some php versions have problems with that. We’ll add a parameter in next release, in between, you can hack the ldap plugin by forcing to ignore this feature.
In plugins/auth.ldap/class.authLdapDriver.php, replace existing line 312:
`$isSupportPagedResult = function_exists("ldap_control_paged_result") && function_exists("ldap_control_paged_result_response");`
by
`$isSupportPagedResult = false;`

**9/ Problems creating workspaces in Italian Language  [FIXED in 6.0.1]**

This should now be fixed in 6.0.1