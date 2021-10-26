_This guide describes the steps required to have Pydio Cells running on Windows_

## Install Cells on Windows 10

### Install a Database

- [MySQL 8.0](https://dev.mysql.com/doc/refman/8.0/en/windows-installation.html)
- [MariaDB 10.4](https://mariadb.org/download/)

### Install Cells

Download the Pydio Cells executable [Download Server](https://download.pydio.com/latest/cells/release/{latest}/windows-amd64/cells.exe).

Open your terminal (powershell by default on windows) then proceed to install with the following command :

- `.\cells.exe configure`

> Note: on powershell (Windows terminal), with legacy version of Cells or Windows, if the arrows keys do not seem to work, you can try with H-J-K-L ( J: Up, K: Down).

You are first asked to choose how you want to run the installation:

- **Browser based**: opens a tab in your local browser with an intuitive installer.
- **command line interface**: for advanced users, pretty straight forward.

After succesfully completing installation, the application working folder with data, configurations and logs is located under %APPDATA%, for instance: `C:\Users\pydio\AppData\Roaming\Pydio\cells`.

> Note: you may have to explicitly allow displaying **hidden files/folders** in your settings to see this folder.

You can now start Cells and access it at `https://localhost:8080` or `https://<server ip or domain>:8080`

```
.\cells.exe start
```

By default the port 8080 is used, to configure a different URL and/or port, run:

```
.\cells.exe configure sites
```
