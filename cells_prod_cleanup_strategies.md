One of the main selling points of Pydio Cells is the ability to finely control and track usage of your business data. However, gathering a lot of information has a cost, especially in terms of disk usage. This page explains how it works under the hood and outlines strategies to help you keep things under control.

## [Ent] TL;DR the sysadmin check list

We list here most of the best practices that we usually setup when configuring a Pydio Cells server. You can find further explanation below.

- Configure and enable following jobs via the scheduler
  - Truncate Logs
  - Purge Files Activities
  - Purge Users Notification

**Warning**: if you are running an old instance of Pydio Cells that has been updated, you should use the `cells admin clean reset-job` command to insure you are using the latest version of the cleaning jobs.

- Monitor free space on your disk, do the cleaning before reaching 90% of disk space
- If you have a medium to large instance, rather use MongoDB for the KV stores
- Define reasonable quotas on the Datasources that are File System Based
- Monitor and prune rolled version of log files under `<CELLS_WORKING_DIR>/logs` 
- Define reasonable pruning strategies for versioned datasources
- Monitor and regularly clean the `<CELLS_WORKING_DIR>/data/.minio.sys/tmp` subfolder to remove parts for uploads that have failed.
  
## In depth explanation 

### Datasources

Before diving into the technical part, let's have a look at the way your documents are stored on the server. As explained in the [admin documentation](), your business data is stored in various datasources.

At installation time you can choose to use block storage for this. Typically, if you decide to use AWS S3, you won't hit any limit but your costs will grow with the amount of data stored.

Otherwise, the documents are stored by default on the server disk at the path defined by the `CELLS_DATA_DIR` environment variable (that is `<CELLS_WORKING_DIR>/data` by default).

#### Business Datasources 

- You should monitor the size of the data uploaded by your users to avoid filling up disks, typically in the `My Files` workspaces.
- [Enterprise] You can define quotas on workspaces to set limits.
- By default, trashed content is moved to recycle bins - these are simply folders at the root of each workspace that are created by the system when necessary - the disk space is not freed until the user explicitly empties the recycle bin.
- By default, when deleting a user, her `My file` folder gets renamed but the data is still there. One way to retrieve the space is to define a workspace for a manager at the root of the corresponding datasource and explicitly delete old data if necessary. 

#### Technical Datasources 

In addition to your business datasources, you should also be aware of technical datasources that store data at the same place:

- cellsdata: all data that is uploaded to cells that have been created from scratch
- thumbs: preview images that are created upon data upload
- versions: old versions of documents 
- binaries: a "fourre-tout" to store various binary files that are used by the server

Depending on:
- your number of users and they respective usage,
- the versioning policies you define,

these datasources might also grow big: you should monitor them and either adapt your configuration to spare space or grow your disks size.

#### Multipart Uploads

When you define a disk based datasource, typically at `<CELLS_WORKING_DIR>/data/mydatasource`, an object service is started that creates a working directory at `<CELLS_WORKING_DIR>/data/.minio.sys` (note that this service is mutualised between datasources that have the same parent folder).

When big files are uploaded, Pydio Cells splits the file in parts to improve reliability and efficiency. These parts are first uploaded under the minio working dir in a `tmp` subfolder. When a multipart upload fails, the parts are left there so that you don't have to upload them again on next try.

Thus, sometimes, if your users upload a lot of large files on unreliable connections, the `tmp` folder might grow big and you have to empty it manually as we cannot decide automatically which parts can be destroyed and which are still to be used. 

### Log Files

The log files can be found under the path defined by the `CELLS_LOG_DIR` environment variable (which is `<CELLS_WORKING_DIR>/logs` by default). These files are automatically rolled out when they reach a certain limit (see below).

You have:

- `pydio.log` (limit 10 MB): the main system logs; these will grow more quickly depending on the log level you define when you start the server. Be aware that using the DEBUG level makes the system extremely verbose.
- `tasks.log` (limit 10 MB): logs for the background jobs.
- `caddy_error.log` (limit 4 MB): error logs of the internal Caddy web server.
- [Enterprise only] `audit.log` (limit 100 MB): business logs to track user activity, typically for compliance and/or audit purposes. 

You should specifically monitor this location and set up external cleaning strategies to get rid of old rolled log files that you no longer need.

### Services

The various microservices all have a specific local working dir that is under the path defined by the `CELLS_SERVICES_DIR` environment variable (that is `<CELLS_WORKING_DIR>/services` by default) on each node where they are deployed.

#### Update Service

Each time you upgrade the server, the old binary is stored under `<CELLS_SERVICES_DIR>/pydio.grpc.update`. These files are each ~200MB and you can safely get rid of them when you are happy with the current running version. Furthermore, you can always download old versions again from our [download server](https://download.pydio.com/pub/)

#### In App Logs (Syslog, Audit, Tasks)

In addition to the log files that we have seen earlier, Pydio Cells also comes with a mechanism to display and search the various logs via the Admin console.

To provide this feature, the server uses microservices that ingest and index logs as they are produced by the other running microservices.
Under the hood, the log microservices parses log events that they receive via gRPC messages and index them in a key/value store.

These key value stores can grow big, especially on big instances. This has 2 consequences:

- more space used 
- longer response times when searching for specific logs in the Admin console.

##### Automatic cleaning 

Tasks logs 

The enterprise distribution also comes with a pre-defined job template that can be used to prune old logs from the KV stores.
Simply adjust the parameters to fit your needs and run it


##### Manual cleaning when using Bleve

By default, Cells uses Bleve as a KV store. 
These stores are files-backed and the corresponding data is stored in subfolders in  `<CELLS_SERVICES_DIR>/pydio.grpc.{log,tasks,audit}`

In worse case scenario, you can get rid of these stores by:

- stopping the app
- delete the `*.bleve` subfolder 
- restart the app

You won't be able to search for old logs, but the app can start restart and work without issues.
If you have kept the log files that are under `CELLS_LOG_DIR` (cf above) you can still dig them _manually_ and thus won't loose information.


##### With MongoDB






