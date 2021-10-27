This guide shows how to install and run Pydio Cells on Windows 10.

The binary also work on other version of Windows Desktop (8, 11) and of the Windows Server. Yet, please note that due to the majority of UNIX-like boxes in the enterprise server world and also the lack of feedback from the community, the Windows version of our application might still have unknown glitches and is not officially suported.

Please feel free to join our [community](https://forum.pydio.com) to improve this. 

## Install Cells on Windows 10 

The only hard requirement is a recent MySQL database. If not yet present on your machine, you can refer to:  

- [MySQL 8.0](https://dev.mysql.com/doc/refman/8.0/en/windows-installation.html)
- [MariaDB 10.4](https://mariadb.org/download/)


You can then download the Pydio Cells executable from [our download server](https://download.pydio.com/latest/cells/release/{latest}/windows-amd64/cells.exe).

Open your terminal (powershell by default on Windows) then proceed to install with the following command:

- `.\cells.exe configure`

> Note: on powershell (Windows terminal), with legacy version of Cells or Windows, if the arrows keys do not seem to work, you can try with H-J-K-L (J: Up, K: Down).

At first prompt, you can choose how you want to go on with the installation:

- **Browser based**: opens a tab in your local browser with an intuitive installer.
- **command line interface**: for advanced users, pretty straight forward.

Once installed, you can find the application working folder with data, configurations and logs under %APPDATA%.  
For instance: `C:\Users\pydio\AppData\Roaming\Pydio\cells`.

> Note: you may have to explicitly allow displaying **hidden files/folders** in your settings to see this folder.

You can now start Cells and access it at `https://localhost:8080` or `https://<server ip or domain>:8080`

```
.\cells.exe start
```

By default, Cells start on port 8080 with a self-signed certificate. To change this and use a different domain, port or protocol, run:

```
.\cells.exe configure sites
```
