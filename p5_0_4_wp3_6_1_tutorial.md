This article is a contribution by Charmlander.

Recently Tested With

+ pydio5.0.4 : http://sourceforge.net/projects/ajaxplorer/files/latest/download?source=files
+ wordpress3.6.1 : http://wordpress.org/latest.zip
+ pydio_cms_4.0.1 : http://sourceforge.net/projects/ajaxplorer/files/ajaxplorer/cms-bridges/ajaxplorer-cms-bridges-4.0.1.zip/download
Contents [hide]

## Basic installation
1. First install WordPress
2. Install pydio with Mysql database
3. Check both Wp and pydio has admin login.(Same admin username is preferred)
4. I do not know what is the exact problem but I had some problems with different admin usernames maybe some browser cache problems.

## WordPress Side
1. Login to admin panel in WordPress and install pydio_cms_4.0.1 via Plugins/Addnew option
2. Be aware that the original file you downloaded from sourceforge.net has all the cms plugins.So extract that file and select the directory wordpress/ajaxplorer only.You can either zip it again and upload to wordpress or copy the whole directory (only Pydio in the wordpress dir) to wp-content/plugins directory.Warning !!! Be sure that there is no recurring directories otherwise the plugin install will return with error.
3. Activate the plugin.
4. On the dashboard select Settings/Pydio
5. In the Ajaxplorer path enter the full installation path something like : /home/user/public_html/ajaxplorer
6. Choose a secret key
7. Be sure that Auto Create (Create Ajxp users when they login) option is Checked to Yes.And do not forget to save changes.
8. We finished the WordPress side.Now it’s time for Pydio part.

## Pydio Side
1. After all these done login to your pydio with admin account and open Settings/Global Configurations/Core Configs/Authentication panel.
2. On the panel goto Main Instance Section and do the followings;
    - Instance Type : Remote Authentication
    - CMS Type : WordPress
    - WordPress url : This is where your logged out users will be redirected to
    - Login uri : /wp-login.php
    - Exit action : Selecet either goto specific page(to wp url above) or only perform logout.
    - Local Prefix : Enter something like wpajax_ (or similar)
    - Roles Map : Leave it blank
    - Secret Key : Your wordpress plugin secret key
    - Users : AJXP_DATA_PATH/plugins/auth.serial/users.ser
    - Transmit Clear Pass : No
    - Auto Create Users : Yes
    - Login Redirect : Leave it blank
    - Admin Login : Leave it blank
3. Now goto wordpress dashboard and activate register users in settings/general —>Membership select Anyone can register.So your register action is activated you have to install a plugin in wordpress in order to send confirmation (send password) e-mails to users being registered if you are on a shared hosting.In this scenario I use Easy WP SMTP.Install it and configure it for sending mails.
4. After you had configured your Easy WP SMTP plugin in WordPress and allow users to register, I think the most tricky part comes against us.If you try to register an user right now you’d encounter with an error like below :
5. “Fatal error: Cannot redeclare ajxp_gluecode_updateRole() (previously declared in /home/user/public_html/test/pydio/plugins/auth.remote/glueCode.php:79) in /home/user/public_html/test/pydio/plugins/auth.remote/glueCode.php on line 103”
To get rid of this notorious error message open your Pydio plugin file located in wordpress/wp-content folder something like /home/user/public_html/wordpress/wp-content/plugins/ajaxplorer/class.WpAjxp.php
6. On the line 36:
`add_action('user_register', array(&$this, 'createUser'), 1, 1);`
7. replace it with :
`add_action('action=register', array(&$this, 'createUser'), 1, 1);`
8. Now Pydio can hook the register actions within the WordPress installation.

## Result
After all changes are applied now you can login or register a new user in wordpress and maybe use a menu on homepage or redirect him automatically to pydio page after login.It depends on your WordPress knowledge beyond this section.All I have to say is when an user logged in WordPress, he can automatically login to Pydio without another login screen.

And this is the test site where I installed while writing this tutorial.I didn’t tested it deeply but practised some login and register actions and it’s fulfilly working.If you want to try just register the site and on homepage click pydio menu on top. http://www.zonomo.com/test/wp

Hope I could help you with my share.Charmlander.22.10.2013