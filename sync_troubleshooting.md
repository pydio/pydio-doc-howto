This how-to will gather common problems encountered with the sync client.

+ **Pydio Server requirements**:
    - v6 or later
    - SQL Database available
    - API properly configured
    - If using SSL, make sure to use a properly validated certificate
+ **Workpsaces requirements**:
    - Meta.syncable plugin active (via Workspace Features)
    - FileHasher plugin active
    - Metastore plugin active (to cache file hashes)
    - If pydio <= 6.0.3, Index.lucene must be active as well
+ **Indexation**:
on the server-side, the changes are continuously indexed so that the sync client only receives the modification of the tree when required. For this reason, if you are planning to modify the data of a workspace from outside pydio, you have to set up a specific strategy to make sure to keep pydio in sync. See How-to on this topic. If you are creating a workspace pointing to an already filled folder, make sure to manually trigger the first indexation as well (see action More…> Index Content in web interface).