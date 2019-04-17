
This how-to will show you how to configure and use different webdav clients, be advised that each client has different it's unique behaviour.

## MacOS Finder

Do the following: **go to your finder > right click > connect to a server**.

[:image-popup:miscellaneous/webdav/macos_webdav_1.png]

In the menu that pops up, add your address like in the examples above, for instance: `http://192.168.0.1:8080/dav/`

[:image-popup:miscellaneous/webdav/macos_webdav_2.png]

You are then prompted to put in your login informations.

You can now access to your files located in your pydio server through the macos finder. It is mounted as a remote disk: if you the connection is lost you also loose your ability to see/interact with the finder.

**Do not forget to refresh the finder if there are changes on cells' side**

## Ubuntu (or any other Linux users)

Ubuntu or other linux users can use the Nautilus file manager.

As seen in the below screenshot (ubuntu 18), if you click on `Other locations`, you open a field on the bottom allowing you to enter the address of the webdav.  
[:image-popup:miscellaneous/webdav/ubuntu_webdav_1.png]

For instance: `dav://192.168.0.1:8080/dav/personal-files`. You are then prompted for login and password and you are good to go.

> if you are using ssl replace `dav` by `davs`  ---> `davs://192.168.0.1:8080/dav/personal-files`

[:image-popup:miscellaneous/webdav/ubuntu_webdav_2.png]

> For command line user you can use `mount` and mount the volume using the same path.

## Windows Explorer

Windows 10 users must:

[:image-popup:miscellaneous/webdav/windows_webdav_1.png]

- Go to **This pc** 
- go in the top menu to **Map a network drive**
- choose a letter type the url such as `http://192.168.0.122:8080/dav/personal-files` (do not forget to tick **connect using different credentials**)

_in the example we show a specific workspace, but you can browse all of them by only typing `/dav/`._

[:image-popup:miscellaneous/webdav/windows_webdav_2.png]

With windows you might encounter an issue when accessing a webdav that is running not with ssl,
to remediate this issue you will have to edit a value in the registry:

* Look for this inside the registry: `HKEY_LOCAL_MACHINE\SYSTEM\CurrentControlSet\Services\WebClient\Parameters`
* Then select this `BasicAuthLevel`
* and set it's value to **2**

_the value to 2 means that it's no more restricted to only secure connection(https)_

It's done. You can now browse your workspaces and cells from your explorer.

**Do not forget to sometimes refresh the explorer (F5 key) if you did operations on the webui directly**

## Cyberduck client (macos & windows)

Cyberduck is a free storage browser and it also have the ability to act as a webdav client, you can downlooad and find more informations about it on their website.

https://cyberduck.io/

Now to configure cyberduck with pydio Cells, 

* First create a connection:
[:image-popup:miscellaneous/webdav/cyberduck_webdav_1.png]

To illustrate the example, pydio cells is running and accessed through this address `http://192.168.0.122:8080:

* depending on if you server is running on http or https choose the matching webdav line.

[:image-popup:miscellaneous/webdav/cyberduck_webdav_2.png]

* Server: the address where your server is running (can also be a domain name for instance example.com)
* Port: the port on which it is accessed, usually it 443 or 80 but for our example we use 8080.
* Username: Your Pydio Cells username
* Path: you can either point to a specific workspace by typing `/dav/workspace-name` or `/dav/` to have a list of all workspaces.
