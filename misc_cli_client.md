In this how to we will take a look at the new cells command line tool and how it enables you to perform actions such as **cp, ls, ...** on your cells instance.

### What is the Cells command line tool

The cells command line tool works like every other tool integrated in most linux distributions, for instance **scp**.
The tool gives you the ability to list, download and upload directly to your remote cells instance into your (workspace/folder/cell) data with the ease of a command on your terminal.

### Installation

Download source code and use the Makefile to compile binary on your os

```
$ go get -u github.com/pydio/cells-client
$ cd $GOPATH/github.com/pydio/cells-client
$ make dev
```

You should have a `cec` binary available, make it executable with `chmod u+x cec`.

Put it in your path or add a symlink to the binary location, typically:
`sudo ln -s /<path-to-bin>/cec /usr/local/bin/cec`

This last step is **required** if you want to configure the completion helper (see [cec completion](https://pydio.com/en/docs/developer-guide/cec-completion)).

Otherwise, you can also do `./cec ls` directly (in such case, adapt the suggested commands to run the examples).

You can verify that `cec` is correctly installed and configured by launching any command, for instance:
`cec version`

### Configuration

You must first configure the client to connect to the server. 

```
$ cec configure
```

You will be prompted with the following informations : 

- Server Address : full URL to Cells, e.g. `https://cells.yourdomain.com/`

- Client ID / Client Secret: this is used by the OpenIDConnect service for authentication. Look in your server `pydio.json` file for the following section (see below), **Id** is the Client ID and **Secret** is the client Secret.

```json
         "staticClients": [
           {
             "Id": "cells-front",
             "IdTokensExpiry": "10m",
             "Name": "cells-front",
             "OfflineSessionsSliding": true,
             "RedirectURIs": [
               "http://localhost:8080/auth/callback"
             ],
             "RefreshTokensExpiry": "30m",
             "Secret": "Nqjuhpzl839618VrbLrnEPyn"
           }
         ],

```

- User Login and password

### Usage

Use the `cec --help` command to know about the available commands. There are currently two interesting commands for manipulating files : 

- `cec ls` : list files and folders on the server, when no path is passed, it lists the workspaces that use has access to.

- `cec cp` : Upload / Download file to/from a remote server.

Other commands are available for listing datasources, users, roles, etc... but it is still a WIP.

## Examples

**1/ Listing the content of the personal-files workspace**

```shell
$ cec ls personal-files
```

it will display an array of this form:

|  TYPE | NAME  |
|---    |  ---  |
|  Folder | personal-files  |
| File  | Huge Photo-1.jpg  |
| File  | Huge Photo.jpg  |
| File  | IMG_9723.jpg  |
| File  | P5021040.jpg  |
| Folder  | UPLOAD  |
| File  | anothercopy  |
| File  | cec22  |
| Folder  | recycle_bin  |
| File  | test_crud-1545206681.txt  |
| File  |  test_crud-1545206846.txt |
| FIle  | test_file2.txt




**2/ Showing details about a file**

```shell
$ cec ls personal-files/P5021040.jpg -d
Listing: 1 results for personal-files/P5021040.jpg
```

it will display an array of this form:

| TYPE | UUID | NAME | SIZE | MODIFIED |
| --- |  ---  |  --- | ---  |       --- |
| File| 98bbd86c-acb9-4b56-a6f3-837609155ba6 | personal-files/P5021040.jpg | 3.1 MB | 5 days ago |
| --- | --- | --- | --- | --- |

**3/ Uploading a file to server**

```sh
$ cec cp ./README.md cells://common-files/
Copying ./README.md to cells://common-files/
 ## Waiting for file to be indexed...
 ## File correctly indexed
```

**4/ Download a file from server**

```sh
$ cec cp cells://personal-files/IMG_9723.JPG ./
Copying cells://personal-files/IMG_9723.JPG to ./
Written 822601 bytes to file
```