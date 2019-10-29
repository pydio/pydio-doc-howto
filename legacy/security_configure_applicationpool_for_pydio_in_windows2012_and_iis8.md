In earlier version of IIS 8, it has sometimes been difficult to isolate web application pool from each other. If multiple web application pool are configured to run as the same identify (for example, Network Service), then code running inside one web application pool would be able to access to resources of others.
In this “How-to”, we will try to configure Pydio to run inside an Application Pool.
For further info: http://adopenstatic.com/cs/blogs/ken/archive/2008/01/29/15759.aspx

##### 1. Download pydio from internet:

https://pyd.io/download/

##### 2. Extract to C:\inetpub\wwwroot\pydio

##### 3. Create an Application Pool in IIS manager

Right-click on Application Pools and select Add.

Type pydio in Name and No Managed Code

[:image-popup:security/configure_applicationpool_for_pydio_in_windows2012_and_iis8/IIS8_AppPool_create-an-applicationpool.png]

##### 4. Convert Pydio folder to web application

Right-click on folder pydio in Default Web Sites to convert this folder to web application

[:image-popup:security/configure_applicationpool_for_pydio_in_windows2012_and_iis8/IIS8_AppPool_add-application.png]

##### 5. Select application pool
We select pydio in the application pools list

[:image-popup:security/configure_applicationpool_for_pydio_in_windows2012_and_iis8/3.connectAs-300x298.png]

##### 6. NTFS permission

In this step, we will set the permission on the Pydio folder. there are two groups of data: 1, php script and configuration infomration 2, repository for user upload/modify. Of cours, we have to have 2 security policy for them. In our situation we need Pydio running in isolated process. We have to remove default user and add Application Pool user in the NTFS access list.

– Disable inheritance: Because pydio folder is a subfolder of wwwroot, if we need to remove IIS_IUSRS, we must remove permission inheritance. Select “Convert inherited permissions into …”

[:image-popup:security/configure_applicationpool_for_pydio_in_windows2012_and_iis8/IIS8_AppPool_Disable-inheritance-300x200.png]
[:image-popup:security/configure_applicationpool_for_pydio_in_windows2012_and_iis8/IIS8_AppPool_convert-inerited-permission.png]

– remove IIS_IUSRS

select accout IIS_IUSRS and remove it

[:image-popup:security/configure_applicationpool_for_pydio_in_windows2012_and_iis8/IIS8_AppPool_remove-IIS_IUSRS.png]

– Add “iis apppool\pydio” in NTFS access list of pydio folder

The Locations must be “this computer” then type **“iis apppool\pydio”** to search pydio worker process.

This is a service account and it have no password.

[:image-popup:security/configure_applicationpool_for_pydio_in_windows2012_and_iis8/IIS8_AppPool_add-pydio-app-pool.png]

– Permission for Pydio scripts. We prevent any modification on the code and configuration carried out by IIS therefore the permission read/exec is sufficient

[:image-popup:security/configure_applicationpool_for_pydio_in_windows2012_and_iis8/IIS8_AppPool_permission-of-pydio-on-Pydio-folder.png]

– Permission for Pydio data. But for pydio/data, IIS need more permission to manipulate users’ data such as upload/download, create new folder/files…

[:image-popup:security/configure_applicationpool_for_pydio_in_windows2012_and_iis8/IIS8_AppPool_full-access-on-Data.png]

##### 6.1 Edit Anonymous Authentication Credentials

[:image-popup:security/configure_applicationpool_for_pydio_in_windows2012_and_iis8/Selection_018.png]

##### 7. Verify process worker for pydio

In task manager, you can see w3wp.exe is running with user pydio

[:image-popup:security/configure_applicationpool_for_pydio_in_windows2012_and_iis8/IIS8_AppPool_verify-process-worker.png]

##### 8. Pydio diagnostic result:

Now we are ready to launch Pydio web site on a web browser. But… there are a lot of “warning”. Don’t worry, we will fix them all.

[:image-popup:security/configure_applicationpool_for_pydio_in_windows2012_and_iis8/IIS8_AppPool_pydio-diagnostic-result.png]

Sometimes PHP in our system can not recognize the repository to locate its session files. We have to specify in conf/bootstrap_conf.php

open conf/bootstrap_conf.php and uncomment and do a small change lines:

//$AJXP_INISET[“session.save_path”] = AJXP_DATA_PATH.”/tmp/sessions”;

to

$AJXP_INISET[“session.save_path”] = AJXP_DATA_PATH.”/tmp/sessions”;

With this configuration, PHP will save all session files on AJXP_DATA_PATH.”/tmp/sessions”. Of course, we have to create tmp and tmp/sessions in AJXP_DATA_PATH.

For security reason, we highly recommend to locate php session out of Pydio folder. If you do that, please remember to set appropriate permission (r/w) on this folder for iis **apppool\pydio**.

When you upload data to server, firstly PHP will cache them on a temporary folder and relocate it to desired location when the upload is complete. We also help PHP where it can locate for temporary files by uncomment in conf/boot_conf.php

//define(“AJXP_TMP_DIR”, AJXP_DATA_PATH.”/tmp”);

to

define(“AJXP_TMP_DIR”, AJXP_DATA_PATH.”/tmp”);

Don’t forget to set permission on this folder for iis **apppool\pydio** account

 

##### 9. Enforce security for your system

After doing all above steps, please refer to https://pyd.io/installing-on-win8-iis8/ for:

###### 9.1. configure mysql

###### 9.2. apply security

###### 9.3. verify the result

 

##### 10. web.config (Pydio 6)

C:\inetpub\wwwroot\pydio\webconf

    <?xml version="1.0" encoding="UTF-8"?>
    <configuration>
        <system.webServer>
            <rewrite>
                <rules>
                    <rule name="Pydio-Rule-1" stopProcessing="true">
                        <match url="^shares" ignoreCase="false" />
                        <conditions logicalGrouping="MatchAll">
                            <add input="{REQUEST_FILENAME}" matchType="IsFile" ignoreCase="false" negate="true" />
                            <add input="{REQUEST_FILENAME}" matchType="IsDirectory" ignoreCase="false" negate="true" />
                        </conditions>
                        <action type="Rewrite" url="./dav.php" />
                    </rule>
                    <rule name="Pydio-Rule-2" stopProcessing="true">
                        <match url="^api" ignoreCase="false" />
                        <action type="Rewrite" url="./rest.php" />
                    </rule>
                    <rule name="Pydio-Rule-3" stopProcessing="true">
                        <match url="^user" ignoreCase="false" />
                        <action type="Rewrite" url="./index.php?get_action=user_access_point" appendQueryString="false" />
                    </rule>
                    <rule name="Pydio-Rule-4" stopProcessing="true">
                        <match url="(.*)" ignoreCase="false" />
                        <conditions logicalGrouping="MatchAll">
                            <add input="{URL}" pattern="^/pydio6/index" ignoreCase="false" negate="true" />
                            <add input="{URL}" pattern="^/pydio6/plugins" ignoreCase="false" negate="true" />
                            <add input="{URL}" pattern="^/pydio6/dashboard|^/pydio6/welcome|^/pydio6/settings|^/pydio6/ws-" ignoreCase="false" />
                        </conditions>
                        <action type="Rewrite" url="index.php" />
                    </rule>
                </rules>
            </rewrite>
        </system.webServer>
    </configuration>
    C:\inetpub\wwwroot\pydio\data\public\webconfig

    <?xml version="1.0" encoding="UTF-8"?>
    <configuration>
        <system.webServer>
            <rewrite>
                <rules>
                    <rule name="Pydio-Public-Folder-Rule-1">
                        <match url="^([a-zA-Z0-9_-]+)\.php$" ignoreCase="false" />
                        <conditions logicalGrouping="MatchAll">
                            <add input="{REQUEST_FILENAME}" matchType="IsFile" ignoreCase="false" negate="true" />
                            <add input="{REQUEST_FILENAME}" matchType="IsDirectory" ignoreCase="false" negate="true" />
                        </conditions>
                        <action type="Rewrite" url="share.php?hash={R:1}" appendQueryString="true" />
                    </rule>
                    <rule name="Pydio-Public-Folder-Rule-2">
                        <match url="^([a-zA-Z0-9_-]+)--([a-z]+)$" ignoreCase="false" />
                        <action type="Rewrite" url="share.php?hash={R:1}&amp;lang={R:2}" appendQueryString="true" />
                    </rule>
                    <rule name="Pydio-Public-Folder-Rule-3">
                        <match url="^([a-zA-Z0-9_-]+)$" ignoreCase="false" />
                        <action type="Rewrite" url="share.php?hash={R:1}" appendQueryString="true" />
                    </rule>
                </rules>
            </rewrite>
        </system.webServer>
    </configuration>