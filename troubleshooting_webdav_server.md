Web Distributed Authoring and Versioning (WebDAV) is an extension of the Hypertext Transfer Protocol (HTTP) that facilitates collaboration between users in editing and managing documents and files stored on World Wide Web servers. A working group of the Internet Engineering Task Force (IETF) defined WebDAV in RFC 4918.
The WebDAV protocol makes the Web a readable and writable medium. It provides a framework for users to create, change and move documents on a server; typically a web server or web share.

The interest of this protocol is that nowadays, most Operating System implement its support by default, and provide the ability to “mount a network drive” mapping to a remote WebDAV server. But still, as each OS has its own implementation of a WebDAV client to do this, it can be quite cumbersome to setup on both server and client sides.

We assume that you have already read the [WebDAV Activation Guide](https://pyd.io/administrator/enriching-your-users-experience/webdav-server/) in the Admin Guide.

## Testing the server ability
### Check requirements
One important thing to understand is that AjaXplorer does not use WebDAV protocol internally (for its web-based client or mobile application). Thus it’s an optional feature you have to activate in the configuration, and everything can be running file in AjaXplorer, that does not mean your WebDAV is up and running.

Since version 5.0.0, we are now using the well-known SabreDAV component as a the WebDAV file server in AjaXplorer. This adds a dependency to the **php_mbstring** extension, and also that you have some kind of **Rewrite Engine** activated in your web server. Apache would for example require mod_rewrite to be loaded.

### Check .htaccess
As described in the Administration Guide, you must manually edit the **.htaccess** file (or equivalent for other webservers than Apache) to make sure the **RewriteBase** is the right one: if your installation if for example located at **http://mydomain.tld/my/files/**, you will have to make sure the RewriteBase is **/my/files**.

Also, the Authorization Header can in some server setup create problems. Try commenting/uncommenting the last two lines of the .htaccess to forward the header.

### Check the server is actually activated
Check both the global configurations (AjaXplorer Main Options), and that the user with which you are trying to access has indeed activated her account, and set up a password if necessary.

### Use the “Browser Access” to test
One of the most reliable testing client would be accessing the webDAV shares through the browser. You have to enable this feature in AjaXplorer Main Options, and then access your shares directly at **http://mydomain.tld/my/files/shares/** . You will be prompted for an HTTP-Authorization, and you must be able to list folders and files here.

## Fixing clients issues (mostly on Windows)

### Windows 7
#### _Slow Performances_

One fix that works for me is to **make sure that the proxy settings in IE are disabled**.  For some reason, at times, the settings “Automatically Detect Settings” In the connection tab reset itself. If it is disabled it works great.

#### _No Connexion at all on HTTPS_

If your SSL Certificate is self-signed, you will have to manually add it to the OS authorized certificates. To do this, access to the server first through your a browser, and there, do the necessary manipulation to tell IE to always trust this certificate.

Then make sure to your the standard “Map Network Drive” dialog, instead of “Connect to an http server…”.

### Mac OS Finder
**Mac OSX Finder** is using a specific Transfer-Encoding header for PUT requests (uploading files), which seem to be very poorly supported by servers, thus breaking the files uploads (file appear but is zero-byte sized).

### Basic-auth only clients
In some cases, you may be using a WebDAV client (typically a programming SDK) that is not able to handle Digest-HTTP Authentication, and only Basic-HTTP. In that case, you can enable Basic in the AjaXplorer configs.