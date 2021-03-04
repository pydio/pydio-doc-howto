The Cells Client makes it easy to communicate with your Pydio Cells Server instance from the command line of your workstation or from automated scripts in a client server.

We explain here how to install and configure the tool. We also introduce a few of the most iomportant commands to get you started quickly.

The current guide presents examples using a Linux Workstation, please adapt if you are using another operating system.

## Overview

The Cells Client a.k.a `cec` works like every other tool integrated in most linux distributions, for instance **scp**.

Using the `cec` command, you can list, download and upload directly to your remote Cells server.

We try our best to be backward compatible, yet you will have a better user experience if your Pydio Cells server is up-to-date (typically in version 2.2+) and use the latest client (2.1+ at the time of writing).

Typically, we have introduced the Personal Access Token that is easier to use and more secure in version 2.2 of the Pydio Cells server.

## Installation

We do not provide a packaged installer for the various OSs.  
Yet, Cells Client is a single self-contained binary file and is easy to install.

We provide binaries for the following amd64 architectures:

- [Linux](https://download.pydio.com/latest/cells-client/release/{latest}/linux-amd64/cec)
- [MacOS](https://download.pydio.com/latest/cells-client/release/{latest}/darwin-amd64/cec)
- [Windows](https://download.pydio.com/latest/cells-client/release/{latest}/windows-amd64/cec.exe)

After download, you have to give execution permissions to the binary file, typically on Linux: `chmod u+x cec`.

We strongly advise to add the command to your `PATH` environment variable, to makes it easy to call the command from anywhere in the system.

On Linux, you can for instance add a symlink to the binary location (replace below with correct path):

```sh
sudo ln -s /path/to/your/binary/cec /usr/local/bin/cec
```

To verify that `cec` is correctly installed, simply run for instance:

```sh
cec version
```

## Connecting To Your Server

The Cells Client is just another client to talk to your Cells server instance to manage your files.  
Thus, it needs to establish a connection using a valid user with sufficient permission to achieve what you are trying to do, typically:

- you will not be able to download a file from a workspace where you do not have read access
- you need write access in the workspace where you want to upload

Once you have a valid user, you have 2 choices:

- Pass the necessary connection information at each call (Non Persistent Mode)
- Go through a configuration step and persists necessary information on the client machine (Persistent Mode)

### Non Persistent Mode

This is typically useful if you want to use the Cells Client in your CICD pipe or via cron jobs.  
In such case, we strongly advise that you create a Personal Access Token on the server and use this.

Let's say that you have created a user `robot` that has sufficient permissions for what you want to do.  
To create a token that is valid for 90 days, log via SSH into your server as `pydio` (a.k.a. as the user that **runs** the `cells` service) and execute:

```sh
$ cells admin user token -u robot -e 90d
✔ This token for robot will expire on Tuesday, 01-Jun-21 16:46:40 CEST.
✔ d-_-x3N8jg9VYegwf5KpKFTlYnQIzCrvbXHzS24uB7k.mibFBN2bGy3TUVzJvcrnUlI9UuM3-kzB1OekrPLLd4U
⚠ Make sure to secure it as it grants access to the user resources!
```

You can then use environment variables (or the corresponding flags) to configure the connection. Typically, in our case:

```sh
export CEC_URL=https://files.example.com
export CEC_TOKEN=d-_-x3N8jg9VYegwf5KpKFTlYnQIzCrvbXHzS24uB7k.mibFBN2bGy3TUVzJvcrnUlI9UuM3-kzB1OekrPLLd4U
```

You can then directly talk to your server, for instance:

```sh
cec ls common-files 
```

### Persistent Mode

In your local workstation, you can also interactively configure your connection once and store credential locally.

If you have a keyring that is correctly configured and running on your machine, we transparently use it to avoid storing sensitive information in clear text.  
You can simply test if the keyring is present and usable with:

```sh
cec configure check-keyring 
```

Calling the `cec configure` command let you then choose between the available authentication mechanism. For persistent mode, we advise to use the default OAuth _Authorization Code_ flow.

```sh
cec configure oauth
```

You are then guided through a few steps to configure and persist your connection. Mainly:

- Enter your server address: the full URL to access your Cells instance, e.g.: `https://files.example.com/`
- Choose OAuth2 process either by opening a browser or copy/pasting the URL in your browser to get a valid token
- Test and validate the connection.

The token is then automatically saved in your keychain and will be refreshed and stored again as necessary.

### Authentication Modes Pros and Cons

As seen above, you might use one of 3 authentication methods to establish the connection:

#### Personal Access Token

A token can be generated on the server for a given user. It can be limited in time or you might choose the auto-refresh mode.

Note about the autorefresh option: let's say you have given a validity of 10 days.
If you connect to your server within 10 days, the token's validity is extended for 10 more days _on the server side_: on the client side the token _string_ remains unchanged.  
Thus, if you have a cron job that runs once a week during the night, typically to push some backups to your server, the token remains valid undefinitly.
But if your server is down and _misses_ a week, the upload will fail the week after, because 14 days have passed and the token expired.

**Pros**:

- Secure
- Non Interactive
- The best solution if the client machine is a headless server, that has no Keyring and must communicate with your server with daemon processes, typically `cron` jobs.
- On client side, you only have to give the URL and the token to establish the connection

**Cons**:

- To created a token, you must either have access to the server as privileged user or ask your sysadmin

#### OAuth2 Credential Flows

This is the recommended strategy for persitent mode on your local workstation.
Calling `cec configure oauth` will guide you through a quick process to securely generate an ID token and a refresh Token.

Under the hood, `cec` will watch the validity of the token. When necessary, it will issue a refresh request and stores the updated tokens in your keychain without you even noticing it. For the record, in Cells 2.2, the default validity period of the refresh token is 60 days.

As tokens are represented as unique (random) complicated strings, this approach makes it difficult to steal your token by only looking at it, even if it ends up in clear text shown to third persons.

**Pros**:

- Secure
- Any user can use her own account to configure a connection, without asking the sysadmin.

**Cons**:

- You must go through an interactive process configure your connection.

#### Client Credential Flows

This legacy method is not recommended and might disapear in a future version.

**Pros**:

- You only have to enter URL, login and password
- Can be used via configure process or directly using flags / env variable at each call
- Any user can use her own account to configure a connection, without asking the sysadmin.

**Cons**:

- The user password ends up stored in clear text in case no keyring is present
- The process will fail if your server relies on external user repository to manage authentication (typically LDAP or SSO).

## Usage

Use the `cec --help` command to know about available commands. Below are a few interesting ones for manipulating files:

- `cec ls`: List files and folders on the server, when no path is provided, it lists the workspaces that the current user can access.
- `cec scp`: Upload/Download file to/from a remote server.
- `cec cp`, `cec cp` and `cec rm`: Copy, move, rename and delete files **within the server**.
- `cec mkdir`: Create a folder on the remote server
- `cec clear`: Clear authentication tokens stored in your keychain.

For your convenience, below are a few examples.

### 1/ Listing the content of the personal-files workspace

```sh
$ cec ls personal-files
+--------+--------------------------+
|  TYPE  |           NAME           |
+--------+--------------------------+
| Folder | personal-files           |
| File   | Huge Photo-1.jpg         |
| File   | Huge Photo.jpg           |
| File   | IMG_9723.JPG             |
| File   | P5021040.jpg             |
| Folder | UPLOAD                   |
| File   | anothercopy              |
| File   | cec22                    |
| Folder | recycle_bin              |
| File   | test_crud-1545206681.txt |
| File   | test_crud-1545206846.txt |
| File   | test_file2.txt           |
+--------+--------------------------+
```

### 2/ Showing details about a file

```sh
$ cec ls personal-files/P5021040.jpg -d
Listing: 1 results for personal-files/P5021040.jpg
+------+--------------------------------------+-----------------------------+--------+------------+
| TYPE |                 UUID                 |            NAME             |  SIZE  |  MODIFIED  |
+------+--------------------------------------+-----------------------------+--------+------------+
| File | 98bbd86c-acb9-4b56-a6f3-837609155ba6 | personal-files/P5021040.jpg | 3.1 MB | 5 days ago |
+------+--------------------------------------+-----------------------------+--------+------------+
```

### 3/ Uploading a file to server

```sh
$ cec scp ./README.md cells://common-files/
Copying ./README.md to cells://common-files/
 ## Waiting for file to be indexed...
 ## File correctly indexed
```

### 4/ Download a file from server

```sh
$ cec scp cells://personal-files/IMG_9723.JPG ./
Copying cells://personal-files/IMG_9723.JPG to ./
Written 822601 bytes to file
```

## Command Completion

Cells Client provides a handy feature that provides completion on commands and paths; both on local and remote machines.

_NOTE: you **must** add `cec` to you local `PATH` if you want to configure the completion helper (see above)._

### Bash completion

To enable this feature, you must have `bash-completion` third party add-on installed on your workstation.

```sh
# on Debian / Ubuntu
sudo apt install bash-completion

# on RHEL / CentOS
sudo yum install bash-completion

# on MacOS (make sure to follow the instructions displayed by Homebrew)
brew install bash-completion
```

_MacOS latest release changed the default shell to ZSH_.

Then, to add the completion in a persistent manner:

```sh
# Linux users
cec completion bash | sudo tee /etc/bash_completion.d/cec
# MacOS users 
cec completion bash | sudo tee /usr/local/etc/bash_completion.d/cec
```

You can also only _source_ the file in current session, the feature will be gone when you start a new shell.

```sh
source <(cec completion bash)
```

Note: if you want to use completion for remote paths while using `scp` sub command, you have to prefix the _remote_ path with `cells//` rather than `cells://`; that is to omit the column character before the double slash. Typically:

```sh
cec scp ./README.md cells//com <press the tab key>
# Completes the path to
cec scp ./README.md cells//common-files/
...
```

Note: when you update the Cells Client, you also have to update the completion file, typically on Linux machines:

```sh
cec completion bash | sudo tee /etc/bash_completion.d/cec
source /etc/bash_completion.d/cec
```
