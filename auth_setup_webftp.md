### SETUP A WEBFTP REMOTE AUTHENTIFICATION
With Pydio you can have FTP Workspaces implying that you will have to authenticate one way or another to access the content and so we offer you a plugin that allows you to have users log in this workspace as if they were logging in to a FTP server such as *FileZilla*.
#### Requirements

1. Pydio installed and fully functionnal.
2. FTP Workspace, you can go **[here](https://pydio.com/en/docs/v8/workspaces-drivers)** to learn how to set an FTP Workspace.
3. FTP Authentification Plugin.

**FTP Authentification Plugin**

First you need you enable this plugin go to **Application Parameters > Available Plugins > Authentification Backends > _FTP Authentification_**.

Then you can start to set it up, i will explain what every field stands for and it's usage.
[:image-popup:/authentication/auth_FTP.png]

+ **FTP Login Screen** : if you want your users to have dialog allowing them to enter host/port data, but in most cases you should disable it.
+ **Workspace** : The Workspace that you are using to check if the credentials are right.
+ **Auto Create User** : you can enable this, it will automatically create a user when someone logs, in Pydio's database.
+ **Login Redirect** : if you want a specific action to happen when someone is logging in you can give a Specific URL.
+ **Admin Login** : Give a user that is considered as an Admin by default. 
+ **Auto apply role** : If you want a role to be applied to every user that is authentificating through this driver.