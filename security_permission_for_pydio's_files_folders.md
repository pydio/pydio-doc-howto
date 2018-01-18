The setting permissions on Pydio’s files/folders is an easy but very important task. Correct permissions can reduce the risk of modifications from other entities than web server and system administrator. We highly recommend you to use the most restricted permissions for Pydio data.

Under the concept of linux

– All files/folders must be owned by root account

– Everyone have no access

– Web server (apache) has permission via group ownership

– Web server has read and execution on .php files and read/write on user data

Following this criteria, the solution below could be applied. When you install Pydio from deb or rpm, Pydio’s files/folders are located in several places such as ***/etc/pydio*** for configuration, ***/usr/share/pydio*** for .php files … and it could be slightly difficult because you will have to look for each file separatly . In this example, we obviously can see that Pydio has group **www** and has no permission for everyone while it is owned by root. Note that data, configuration files and logs may be changed and permission must be r/w for group.

Example:

      # www is for httpd group name
    chown -R root:www PYDIO_INSTALL_DIR/
    cd PYDIO_INSTALL_DIR/
    find ./ -type d -exec chmod u=rwx,g=rx,o= '{}' \;
    find ./ -type f -exec chmod u=rw,g=r,o= '{}' \;

    #Mod data dir for config changes and logs
    find data -type d -exec chmod ug=rwx,o= '{}' \;
    find data -type f -exec chmod ug=rw,o= '{}' \;

    # fix permission on .htaccess to read-only for apache
    find /var/lib/pydio -name .htaccess -exec chmod 640 '{}' \;
    find /var/cache/pydio -name .htaccess -exec chmod 640 '{}' \;
    find /usr/share/pydio -name .htaccess -exec chmod 640 '{}' \;
    
When Pydio is installed from the archive on your webserver you just have to :    

```
    chown -R root:www-data /var/www/PYDIO_INSTALL_DIR/
    cd /var/www/PYDIO_INSTALL_DIR/
    find ./ -type d -exec chmod u=rwx,g=rx,o= '{}' \;
    find ./ -type f -exec chmod u=rw,g=r,o= '{}' \;
    
    #Mod data dir for config changes and logs
    find data -type d -exec chmod ug=rwx,o= '{}' \;
    find data -type f -exec chmod ug=rw,o= '{}' \;
    
    # fix permission on .htaccess to read-only for apache
    find /var/www/PYDIO_INSTALL_DIR -name .htaccess -exec chmod 640 '{}' \;
```