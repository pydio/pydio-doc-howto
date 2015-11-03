The setting permission on Pydio’s files/folders is an easy task but very important. Correct permission can reduce the risk of modification from others entities than web server and system administrator. We highly recommend to use the most restricted permission for Pydio data.

Under the concept of linux

– All files/folders must be owned by root account

– Everyone have no access

– Web server (apache) has permission via group ownership

– Web server has read and execution on .php files and read/write on user data

Following this criteria, the below solution could be applied. When you install Pydio from deb or rpm, Pydio’s files/folders are located in several places such as /etc/pydio for configuration, /usr/share/pydio for .php files … and it could be slightly more difficult for this task. In this example, we can obviously see that Pydio has group www and has no permission for everyone while it is owned by root. Note that data, configuration files and logs may be changed  and permission must be r/w for group.

Example:

    # www is group name for httpd
    chown -R root:www PYDIO_INSTALL_DIR
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
    
For further information please visit: https://pyd.io/f/topic/recommended-file-and-folder-permissions/