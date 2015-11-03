Syncing against an workspace which data can be modified outside of Pydio can be a challenge. This article will describe the mechanisms involved and the possible strategies for that.

This article is applicable as well for search engine indexation issues, which rely on the same mechanism.

### How does sync/indexation work?
To perform syncrhonization, the desktop clients regularly “queries” the server to know if there were any changes inside the currently synced workspace. These requests are very light as if there were no changes since the previous one, the server just returns nothing, and computes nothing. To achieve that, we need to make the changes indexed “on the fly” when documents are modified in Pydio, and this is easily done when documents are added/deleted/modified directly throughout a Pydio interface (web, mobile, sync).

But sometimes, you will define a workspace that points to a storage which can be modified externally: files are posted by FTP to the filesystem, files are accessed through Samba shares, and theses shares are mounted directly via other protocols, etc… In that case, Pydio is basically not aware of the content changes, and you will observe issues during synchro (the web interface and mobile apps listing is not affected, as it’s always querying the storage in real time). So we have to make sure Pydio is “in sync” with its storage.

We will describe here the various options, starting with the most efficient ones, and ending with the more performance-greedy ones.

### Triggering unitary indexation
If the files that are modified inside the storage are handled by a bot, or an automatized process, or through a protocol where you can hook to some events (like an ftp server), the best option is to actually inform pydio directly when a file is modified. This can be done by calling the “lsync” action either by command line or rest API. It takes three parameters and covers all modifications types: old, new, copy: old is either a filepath (file modified or deleted) or null (file created), new is either a filepath (file created or modified) or null (file deleted), copy is true or false and used only when old and new are not null, to determine whether it’s a move or a copy.

### Triggering regular folders/worskapce re-indexation
If you cannot manage each file modification, you can on a regular basis trigger a re-indexation of some folders that you specifically want to monitor. Use the “index” action, again, either via command line or Rest API, and you can pass a “dir” parameter to limit the reindexation only to a given folder.

### Auto-detecting changes
We added an auto-detection feature to make the process easier. It is only implemeted for “syncable” workspaces, and is called asynchronously (triggers a php command line) when the sync client is asking for “changes”. This can imply a small delay to see the changes impacted in the local folder. It will browse the folders on the filesystem and compare their modification time with the one register in the DB index, so it will significantly burden the server, you should probably not use this feature for many workspaces and many users.

To enable that feature, switch it on in the Meta.syncable parameteres of the workspace. And additional timer allows to avoid to rescan everything on each “changes” request, but rescan only every X-minutes.