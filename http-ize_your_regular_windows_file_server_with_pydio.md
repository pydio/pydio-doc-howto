author Koen Noens

[:image-popup:system/http-ize_your_regular_windows_file_server_with_pydio/Win2Pydio.png]Take a regular Windows file server (or any NAS that understands SMB or CIFS). Deploy Pydio. Map your Pydio workspaces to folders and shares on your Windows file server, and let Pydio retrieve user accounts and groups from your Active Directory. Wat you get is that all those files are now instantly available to users with nothing but a web browser and an internet connection, from a laptop, a tablet, or a smartphone. With all your original NTFS Security Access Control Lists still in effect. That’s what “Put your data in orbit’ means to me.

In this mini-howto, I’ll explain you how it’s done.

## Introduction and Context
If you have an ICT infrastructure dat dates back to the nineties or the beginning of this century, you probaly have tons of data (user files such as office or pdf documnts, spreadsheets, … ) on Windows file servers or a NAS that uses a Windows file sharing protocol (SMB, CIFS), with security based on NTFS ACL and Active Direcory user accounts.

As long as all your clients were Windws PC’s sitting on the corporate network, this was fine. But now your users are becoming mobile, and you’re looking for ways to make those files available to tablets or non-Windows laptops, maybe over the internet, and so on. A web based solution would cover most of that, but you don’t want to mirror all those files, and you definitely don’t want to duplicate the user authentication and authorisation effort.

That’s what Pydio can do for you : leverage your existing infrastructure to respond to these new uses cases.

## Setting it up
Installing Pydio is a breeze, and I won’t go in the details here (refer to the [Administrator Guide](https://pyd.io/?p=66)).
Obviously, you do not install Pydio on your Windows file servers. You can run Pydio on a standard LAMP server and technically it will behave as a (CIFS, SMB) client to your file servers.

You may have some network design considerations to think about : if you want to make your Windows/CIFS storage available over the internet (for ‘road warriors’, teleworkers and such) you might want to put a Pydio server in your DMZ, and/or behind a reverse proxy. It’s beyond the scope of this howto to go into that. But it is entirely possible, and Pydio will work flawlessly behind a standard HTTP(S) server such as Apache with its standard proxy modules.

## Set up LDAP/AD Authentication
If you are already using Active Directory Services, your files will most likely have NTFS security (or share permissions) using AD accounts. So you’ll have to set up Pydio to use LDAP/AD authentication as well for Pydio to authenticate your users against the AD. We will also set up Pydio to access the storage back-end (your Windows file servers) using the account of the logged-on user – that’s how your existing file and directory NTFS ACL’s will remain in effect while the files are being accessed through Pydio.

refer to [“LDAP/AD authentication”](https://pyd.io/administrator/configuring-global-parameters/setup-authentication-driver/binding-to-an-ldapad-server/) in the Administrator Guide and the howto on [“Groups and Roles with LDAP/AD integration”](https://pyd.io/windows-file-server-to-web-based-file-repository/groups-and-roles-with-ldapad-integration-draft) (Authentication an User Management Sections of the Knowledge Base) for how to set up LDAP/AD Authentication.

When configuring LDAP/AD, note that you can map AD group membership to Pydio “Roles”. Do that; you will need those roles later on, but we’ll come back to that.


## Storage connectors
There’s a new meta connector but I haven’t used it yet. In this how-to, we explain the use of the “SMB” or “Samba” connector.

Setting it up is fairly straightforward : in “Settings” -> “Workspaces and Users” :

+ click Workspaces  -> New workspace
	- Wokspace label = Name for the workspace, as seen by the user.
	- Access driver = “Samba”
	- Server = server IP or hostname (you don’t need to preceed it with "\ \")
	- Uri = path to the share – not including the server hostname. e.g. if you want to map to a share `\\fileserver\share\subdir`, you enter  `/share/subdir`. **_note the direction of the slashes_**
	- Domain : default domain to use with session credentials. This is your AD Domain Name in `domain.tld` notation. Setting this will allow your users to log on with a short logon name (such as ‘pd’ in stead of ‘pd@domain.tld’ or ‘DOMAIN\pd’)
	- User Credentials  : **leave blank**  – you do not want all connections to this share to happen under the same generic account – you want Pydio to use the logged-on user’s credentials.
	Use Session Credentials : **YES**
	- Default Rights : you probably want “Read and Write” here
+ Save

That’s all for creating a workspace. You’ll still have to set Security on it.
To set security, go to “Roles” -> select the role that you want to set permissions for -> go to the tab ‘ACL’ -> check the applicable checkboxes.
To set security for individual users, go to “Users and Groups”, select the applicable user -> go to tab ‘ACL’, etc.

This will work the way a Windows administrator expects : Pydio ACL in combination with NTFS security of the underlying file server works the same way “share permissions” in Windows do : the Pydio ACL can not give more rights than the user already has (in NTFS ACL), but it can limit them further.

A simple approach to handle this would be to give reasonably wide rights in Pydio, while counting on the NTFS ACL of the underlying storage to limit it firther as needed. You can also make Pydio ACLs that are a close reproduction of the NTFS ACL. Or you can use Pudio ACL to impose further limits on top of your NTFS ACL, eg if you want to limit web access to a subset of the users that already have regular (windows file sharing) access.

### Groups And Roles
Use Pydio’s roles to manage the security on your workspaces.

On your Windows file server, you probably are using Security Groups to manage Access Controll on folders or shares. The Pydio feature that maps most closely to AD Security groups are Roles. In the AD/LDAP Authentication Plugin configuration, you can map AD groups to Pydio **_Roles_** and use those to manage the security of workspaces (that map to shares and their subdirectories).
This may sound bit confusing, but if you give it a try you’ll find it all comes together quite nicely.

When you map AD Security Groups to Pydio **‘Roles’**, you also get AD “nested” groups (groups that have other groups as member) represented in Pydio, and Pydio will deduce group membership and the corresponding permissions, so your NTFS ACL that use groups (even nested groups) will still work.

As explained earlier : to set security in Pydio, go to “Roles” -> select the role that you want to set permissions for -> go to the tab ‘ACL’ -> check the applicable checkboxes.

**_Note that Pydio Groups do not map very well to AD Security Groups. Use Roles instead._**

### Access Based Enumeration
Your Windows Access Based Enumeration still works in Pydio. I’m assuming this is because the smbclient (the program Pydio uses to implement its SMB connector) honors the ABE, so Pydio will only show the directories the logged-on user has access to, even if the workspace contains more than that. Jut like ABE on a Windows PC. This is very convenient if your working with top level shares (and thus larger Pydio workspaces) and rely on ABE and ACL to present subdirectories to the users ; on Pydio, your users will see only what they normally see in Windows Explorer.

## This is mechanism, not policy
LDAP/AD authentication and remote storage connectors (in this case : SMB) are mere mechanisms that, combined, form a web layer over a regular file server. But of course you can use them any other way you like. You don’t have to do it this way.

One of the things you’ll want to think about is what workspaces you’ll create, and to which parts of your Windows shares they will be mapped. It’s probably best you simply try to set up one, two or tree workspaces, mapped to different shares or subdirectores of shares, to see how that works and then decide what works for you.

## Using templates
If you have to do a lot of these “workspace mapped to file server”, you can create a storage template that includes the common settings (such as NAS server name, domain name, user credential settings, and miscellaneous defaults. To create a new workspace from a template, you then do as above but set `access driver: name_of_template`. The pre-set configuration of the template will apply, and you only have to set a few extra values (such as workspace name + corresponding URI path).

## Work Offline or with tablets
You can also extend your solution with other features of Pydio. One is the the local synchronization client, which lets you keep aan ofline mirror of a workspace, eg on a laptop. (At the time of writing, the offline sync client is still in beta). If you plan to use offline synchronization or personal workspaces, this may affect how you map those workspaces to the back-end.

For use on tablets, you can either use a regular browser, or use the Pydio App (on Android and iPad).