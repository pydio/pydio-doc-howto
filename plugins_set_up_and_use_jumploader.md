## Install Jumploader
Installing Jumploader is quite simple. All you have to do is to download the Jumploader jar file at this address: http://jumploader.com/jumploader_z.jar.

Now that you have you jar file you just have to copy it to the following directory ajaxplorer_install_path/plugin/uploader.jumploader .

 

## Enable Jumploader
To enable Jumploader you have to go to the GUI in the section Settings->Global Configuration->Plugins->Uploader and enable Jumploader by double clicking on it and disable all the other uploaders. The result should be as follow:

[:image-popup:plugins/set_up_and_use_jumploader/Screenshot-from-2013-07-26-101641.png]
 

 

## Use Jumploader
### Upload simple files
The upload of simple files can be divided into two parts: the files under the php_upload_limit and the ones above this limit (need partitioning). When you want to upload a file you just have to click on Send->From Computer. It will launch the Java applet that will allow you to choose and upload your files.

[:image-popup:plugins/set_up_and_use_jumploader/jumploader.png]

You then have two ways to add files: use the “add” button of the applet or drag and drop the files one by one into the applet.

[:image-popup:plugins/set_up_and_use_jumploader/Screenshot-from-2013-07-26-105737.png]

**Known issues:**

+ When using a WebDAV repository the files above the php upload limit are not uploaded well.

### Upload directories
Jumploader allows you to upload entire folders by drag and drop or, when using the “add” button, left click once on a folder and type enter.

When you’re uploading a folder it will recreate the entire folder tree and will place every file in its relative position inside the folder.

**Known issues:**

+ When using a FTP repository it will not recreate the folder tree and will copy each file in the current directory.
+ When using a WebDAV repository you will have the same issue with partitioned files and sometimes some files’ name will be altered.

### Cross session resuming
When you begin an upload session you have, now, the possibility to resume this session. In order to do this you have to select and upload the same file for which the upload has been interrupted (works only for partitioned files). Then instead of uploading this file from the begining it will resume from where it has been interrupted.

**Known issues:**

+ Works only with FS based repositories (FS, SMB and SFTP)