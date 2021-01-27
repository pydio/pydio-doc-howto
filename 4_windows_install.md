_This guide describes the steps required to have Pydio Cells running on Windows_

## Install Cells on Windows 10

### Install a Database

- [MySQL 8.0](https://dev.mysql.com/doc/refman/8.0/en/windows-installation.html)
- [MariaDB 10.4](https://mariadb.org/download/)

### Install Cells

Download the Pydio Cells executable [Download Server](https://download.pydio.com/latest/cells/release/{latest}/windows-amd64/cells.exe).

Open your terminal (powershell by default on windows) then proceed to install with the following command :

- `.\cells.exe configure`

> Note, on powershell (terminal) the keys to navigate are H-J-K-L ( J = UP, K = DOWN).

The binary will ask you to choose how to run the installation :

- **Browser based**: will open a browser tab with an intuitive installer.
- **command line interface**: for advanced users, pretty straight forward.

Then, provide your database information and you are good to go.

Once the installation is done the folder containing your data and settings is located under %APPDATA%.

> To display the folder, this might require enabling the setting, display **hidden files/folders**

**Data Location** : `C:\Users\pydio\AppData\Roaming\Pydio\cells`

You can now start Cells and access through `https://localhost:8080` or `https://<server ip or domain>:8080`

```
.\cells.exe start
```

By default the port 8080 is used, to configure a different interface and port run the following command:

```
.\cells.exe configure sites
```




