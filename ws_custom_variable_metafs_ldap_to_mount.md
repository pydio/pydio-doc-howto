Customization a variable on Pydio is a interested feature that helps you resolve some technical problems in installation and integration of Pydio in your organization. But the combination of Custom Variable and others plugins is complicated with typical user. In this ‘how-to’, I would like take an example that may helps you can easily understand Custom variable.
The objective of this article is mount automatically user’s home directory in to Pydio each time user login and umount when he/she logout.
The value of home directory is mapped from ldap

## Create a workspace for mounting
We create a new workspace to content the user’s home directory in Pydio
Create a folder for mounting. Be sure that this folder is empty (otherwise, the mount command will generate an error)
For example:

    # mkdir /mnt/metafs

set permission

    # chown www-data:www-data -R /mnt/metafs

In Pydio, go to Globall Settings >> Workspaces >> Click “new workspace”
Workspace Name: HomeDirectory
*Note: specify the PATH of this workspace is folder you’ve created.

[:image-popup:miscellaneous/custom_variable_metafs_ldap_to_mount/CV_wsp_metaFS.png]

## Add meta.fs (FS mount) into this workspace
In Pydio, go to Globall Settings >> Workspaces >> >> Select “HomeDirectory” >> Click Edit
Then click on “Workspace Features” tab
In “Meta Plugin” drop down list, select FS Mount
In the Dialog:
– FS Type: default cifs
– SUDO: Yes
– RemotePath: MAP_USER_HOME_DIR (this value will be replaced in custom variable)
– **Mount point: /mnt/metafs/AJXP_USER**
– Mount Options: default
– Pass Password via … : Yes (recommended)
– User: blank
– Password: blank
– Session credentials: YES
*Note: Click on “Add Source”

[:image-popup:miscellaneous/custom_variable_metafs_ldap_to_mount/CV_metafs_config.png]

## Enable store credential in session
In Pydio, go to Global settings >> Application Core >> Authentication >> Select YES for “store credentials in session”

[:image-popup:miscellaneous/custom_variable_metafs_ldap_to_mount/CV_enable_cred_session.png]

## Mapping homeDirectory from LDAP
In this article, I supposed that you’re configured Pydio to use LDAP.
Map attribute “homeDirectory” as in the screenshot:

[:image-popup:miscellaneous/custom_variable_metafs_ldap_to_mount/CV_mapHomeDirectory.png]

the “homeDirectory” attribute from LDAP will be mapped to Pydio into variable core.conf/USER_HOME_DIR. We will use the value of this variable to replace the string MAP_USER_HOME_DIR in configuration of workspace created above

## Script for replacement
In Pydio folder, create a folder myCode

    # mkdir /usr/share/pydio/myCode
    # touch /usr/share/pydio/myCode/mapUserHome.php
    # chown www-data:www-data -R /usr/share/pydio/myCode

the file mapUserHome.php:
In this file, we will looking for variable whose value is MAP_USER_HOME_DIR and replace its value by core.conf/USER_HOME_DIR

    <?php
    function myFilter(&$value)
    {
    if ((!is_string($value)) || strcmp($value, "MAP_USER_HOME_DIR") !== 0) return;
    AJXP_Logger::debug(__CLASS__, __FUNCTION__, "Start custom variable");
    $currentUser = AuthService::getLoggedUser();
    if (is_null(ConfService::getRepository())) return;
    if ($currentUser != null) {
    $repoObject = ConfService::getCurrentRepositoryId();

    AJXP_Logger::debug(__CLASS__, __FUNCTION__, "Found logged user");
    $currentUser = AuthService::getLoggedUser();
    //-------------------
    // get attribute of ldap user via core.conf/USER_HOME_DIR
    //-------------------
    $custom = $currentUser->mergedRole->filterParameterValue("core.conf", "USER_HOME_DIR", $repoObject, '');
    AJXP_Logger::debug(__CLASS__, __FUNCTION__, "custom: ".$custom);
    if (empty($custom)) return;
    $custom = str_replace("\\", "/", $custom);

    //-------------------
    //replace Remote path in configuration of FS Mount by ldap attribute
    //-------------------
    $value = str_replace("MAP_USER_HOME_DIR", $custom, $value);
    AJXP_Logger::debug(__CLASS__, __FUNCTION__, "After replacing: ".$value);
    }

    }
    ?>

## Enable apache to launch mount/umount
In /etc/sudoers, add following line:
`www-data ALL = NOPASSWD: /bin/umount,/bin/mount`

## Troubleshooting
The best way to troubleshoot is use mount command line manually
`mount -t cifs //homeDirectory /mnt/metafs -o username=name,password=pass,.....`
Depend on your ldap, the user home directory attribute is variable, such as home, userhome, homeDirectory

## Links
https://pydio.com/forum/f/topic/access-protected-var-for-homedrive/