### BASIC SETTINGS TO CHECK
+ **php-ldap extension** : you should make sure that you have php-ldap 
you can use `php -m | grep ldap` to know.
If you do not have the ldap extension you can install it through
aptitude, yum, curl ... by using `install php-ldap`

    *If you want to get more informations about the extension you can visit the          [PHP-LDAP](http://php.net/manual/en/ldap.installation.php) page.*
    
    
### IF YOUR ISSUE WAS NOT LISTED
You should re-check if the test between your LDAP and Pydio was successful when you were setting it up, you have the 'Try to connect to LDAP' Button to make sure that you gave the right informations. Otherwise i advise you to reconfigure it from scratch in case you missed a setting.

If none of the above worked you should make sure that your Pydio installation was rightfully done, you can also go check **[Pydio's Troubleshooting](https://pydio.com/en/docs/v8/troubleshooting)** page.

Else you can go on our **[Forum](https://forum.pydio.com/)** and search if someone had a similar issue or you can create a post.