
You find below instuctions to configure and use each of the webdav client we have been testing. Be advised that each client has a different behaviour, unique to itself.

## MacOS Finder

Do the following: **Go to your Finder > right click > connect to a server**.

[:image-popup:miscellaneous/webdav/macos_webdav_1.png]

In the menu that pops up, add your server public address (e.g. `http://192.168.0.1:8080/dav/`).

[:image-popup:miscellaneous/webdav/macos_webdav_2.png]

You are then prompted to enter your credentials.

You can now access the files located in your Pydio Cells server using the MacOS Finder. Note that your server is mounted as a remote disk: if the connection is lost you also loose your ability to see/interact with your files using the Finder.

**Do not forget to refresh the Finder if there are changes on Pydio Cells' side**

## Ubuntu (or Linux like distributions)

Linux users can use the Nautilus file manager.

As seen in the below screenshot (Ubuntu 18), if you click on `Other locations`, a field on the bottom of the screen is opened, allowing you to enter the address of the webdav.  
[:image-popup:miscellaneous/webdav/ubuntu_webdav_1.png]

For instance: `dav://192.168.0.1:8080/dav/personal-files`. You are then prompted for login and password and you are good to go.

> if you are using SSL replace `dav` by `davs`  ---> `davs://192.168.0.1:8080/dav/personal-files`

[:image-popup:miscellaneous/webdav/ubuntu_webdav_2.png]

> For command line user you can use `mount` and mount the volume using the same path.

## Windows Explorer

Windows 10 users must:

[:image-popup:miscellaneous/webdav/windows_webdav_1.png]

- Go to **This pc** 
- go in the top menu to **Map a network drive**
- choose a letter type the url such as `http://192.168.0.122:8080/dav/personal-files` (do not forget to tick **connect using different credentials**)

_In the example, we show a specific workspace, but you can browse all of them by only typing `/dav/`._

[:image-popup:miscellaneous/webdav/windows_webdav_2.png]

With windows you might encounter an issue when accessing a webdav that is not running with SSL.  
To solve this, you have to edit a value in the registry:

* Look for this inside the registry: `HKEY_LOCAL_MACHINE\SYSTEM\CurrentControlSet\Services\WebClient\Parameters`
* Then select this `BasicAuthLevel`
* and set it's value to **2**

_the value to 2 means that it's no more restricted to only secure connection(https)_

It's done. You can now browse your workspaces and cells from your explorer.

**Do not forget to refresh the explorer (F5 key) to get an up to date state each time you make a modification in the GUI directly**: auto refresh and synchronisation is not yet implemented.

## Cyberduck client (MacOS & Windows)

Cyberduck is a free storage browser. It can also be used as a webdav client.

You can downlooad it and find a complete documentation on [Cyberduck website](https://cyberduck.io).

To configure Cyberduck with Pydio Cells: 

* Create a connection (in the below example, Pydio Cells is accessed at `http://192.168.0.122:8080`)

[:image-popup:miscellaneous/webdav/cyberduck_webdav_1.png]


* Choose relevant protocol (HTTP or HTTPS) depending on your setup.

[:image-popup:miscellaneous/webdav/cyberduck_webdav_2.png]

* Server: the address where your server is running (can also be a domain name for instance example.com)
* Port: the port on which it is accessed, usually it 443 or 80 but for our example we use 8080.
* Username: Your Pydio Cells username
* Path: you can either point to a specific workspace by typing `/dav/workspace-name` or `/dav/` to list all workspaces.
