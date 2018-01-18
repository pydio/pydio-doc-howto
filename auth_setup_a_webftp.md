## Introduction
Pydio provides drivers for accessing a remote FTP server, and also specific settings of the auth.ftp driver to let you setup a “webftp” like configuration: user will be presented with a full-fledged FTP Connexion screen instead of a login screen, including the FTP Server connexion data (Host, Path, Port, etc.). This is typically handy for hosters who want to provide a FileZilla alternative to their customers.

[:image-popup:authentication/setup_a_webftp/webftp.png]

**Disclaimer**: when used in the “webftp” mode and accessed through a mobile browser (iOS / android), Pydio will falsely propose the usage of the native applications, but they **currently do not support this feature**, as the login screen must be extended inside the application. Please make sure to inform your users of this limitation.

## Configuration
To achieve this setup, you need two things : a specific “dynamic” FTP-based workspace, and the correct configuration of the “FTP Authentication” driver in "Other Plugins" .

### FTP Workspace
To be able to query the FTP server for the user authentication, the Auth Driver will refer to an underlying FTP workspace, that you need to set up in preamble. The workspace should be configured as follow:

+ Connection : The usual settings, your host,port,path and optional settings all up to you, You have the ability to test the validity of your connection.
+ FTP Server Tweaks : User id ( use ls or dir most of the time)and the first of the two last numbers as the user id.
+ Dynamic FTP : Set “Pass Ftp Data Through Auth Driver” turn this setting ON.
+ Default Rights : Set the default rights to Read/Write, as once logged, it will be the actual workspace the user will use to access their remote FTP server.

### FTP Auth Driver
Once the dynamic workspace is created, you will be able to edit the Menu > Authentification , and select as main instance  the “FTP Authentication” driver. Then, set up the driver as in the image below.
[:image-popup:authentication/setup_a_webftp/webftp2.png]

The “My FTP” here is the workspace created at previous step. The “Admin Login” should be a user of one accessible FTP server, that will be detected by Pydio as an admin if this user logs in.

**Disclaimer: If you want to switch from the "ftp login" to the "old login" be sure that "admin login" and "ftp login" are the same**

Make sure not to set a secondary auth driver, otherwise the dynamic login screen mechanism will not properly work. 	