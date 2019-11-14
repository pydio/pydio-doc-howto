In a fresh install of Pydio Cells, the application creates 3 default datasources, namely:

- **pydiods1**: default data source for shared files: the `Common Files` workspace that exists at startup points here
- **personal**: contains "My Files"-like workspaces for each user
- **cellsdata**: is used to create _cells_ when no root node has been defined upon creation

By default, these are created under `<CELLS_WORKING_DIR>/data/`. Furthermore, 3 others datasources are implicitely created to store internal and technical data:

- **thumbs**: stores the thumbnails
- **versions**: stores the old versions of the files, when the versioning is enabled on a given DS
- **binaries**: service docstore, stores various things, FI: avatars, custom background...

In the default direct-to-disk setup, these 3 DSs are in fact distinct buckets exposed by the `minio` object service of the default `pydiods1` DS: you can see the corresponding folders created as siblings of the `pydiods1` folder, under `<CELLS_WORKING_DIR>/data/`.

If you want to change this layout, this is possible, but not straightforward; you have to understand what is hapenning and follow exactly the below steps. We plan on enhancing this in a future version soon.

So let us move all these 6 DSs to amazon S3.

### Create the required buckets

First, we have to create 6 buckets in our S3 remote storage.
Note that buckets names in AWS have to be globally unique, so you cannot use the names they have in the local vanilla setup. Let us use for instance:

- com-example-cells-personal
- com-example-cells-cells
- com-example-cells-common
- com-example-cells-thumbs
- com-example-cells-versions
- com-example-cells-binaries

Note that due to their implicit declaration (they retrieved their API key and secret from the pydiods1 DS), the thumb, versions and binaries must be buckets of the **same** storage than the ds that is defined as default DS (by default pydiods1): _they are just additional bucket in the same s3 storage as the main default DS_. Otherwise you will have to tweak the config even a little bit more.

Note also that in Cells ED, you can also create only one bucket for cellsdata, personnal files and Commnfile, and rather use a sub folders to setup the default common Files folder and the 2 template paths.

### Create the new datasources

Then, using the admin console of the web UI, we create new DSs pointing to these newly created buckets for personnal, cells and common.

| **default datasource** | **NewDatasource** | **Usage**            |
|------------------------|-------------------|----------------------|
| pydiods1               | s3common          | Common files         |
| personal               | s3personal        | My files for users   |
| cellsdata              | s3cells           | Root nodes for cells |

In Home distribution, as you cannot edit the default template path for cellsdata and personal files, you rather have to delete the existing default ds and recreate them **with the exact same name** pointing toward your newly created s3 buckets.

### Adapt the template paths to point to the new datasources

Template paths are the mechanism that enable to have dynamic workspaces, typically depending on the name of the loǵged in user for personnal files. In Cells ED you can modify this by going to:  
`Admin console >> show advanced parameters >> Left column menu >> Data management >> Template Path`

With the above naming, you should then modify the 2 default template path to reather have:

```conf
# Cells
Path = DataSources.s3cells + "/" + User.Name;
# My Files
Path = DataSources.s3personal + "/" + User.Name;
```

_To use auto-completion: first move your cursor after the dot, then press `CTRL + SPACE`_

### Adapt config by directly impacting pydio.json

The `pydio.json` file that is at the root of the `CELLS_WORKING_DIR` is the main configuration file of your instance/node and is also _dynamically_ updated by the app when an admin make some changes via the admin console. You should rather handle it with extra care and we always advise to:

- shutdown cells before editing the file
- do a proper backup of your file before modifying it.

Thus said, let's proceed to the next step by editing this file:

[:image-popup:/devops/change_default_ds.png]

Modify the higlighted part (by default it uses the default datasource `pydiods1`) to your new datasource, in our example `s3common`.

You must then also update the following properties in the `services` section to use the bucket names you have created for thumbs, versions and binaries implicit DSs:

Change these:

```json
...
"pydio.docstore-binaries": {
      "bucket": "binaries",
      "datasource": "default"
 },
 ...
"pydio.thumbs_store": {
      "bucket": "thumbs",
      "datasource": "default"
 },
 ...
 "pydio.versions-store": {
      "bucket": "versions",
      "datasource": "default"
 },
 ```

to rather have (using the names of our example, adpt to your use case):

```json
...
"pydio.docstore-binaries": {
      "bucket": "com-example-cells-binaries",
      "datasource": "default"
 },
 ...
"pydio.thumbs_store": {
      "bucket": "com-example-cells-thumbs",
      "datasource": "default"
 },
 ...
 "pydio.versions-store": {
      "bucket": "com-example-cells-versions",
      "datasource": "default"
 },
 ```

### Final STEP

You should be now fully setup. Just restart the app and perform a few tests. If everything works fine, you can now delete the default datasources that you do not use anymore.

### Troubleshooting

Note that the above procedure does not include migration of existing data, after restart you will then typically have lost all existing thumbnails that will be blank.

In order to double check everything is working correctly, you might perform following checks:

- Change your account avatar: it insures the `binaries` implicit DS runs OK
- Upload an image: this checks the `thumbs` implicit DS
- Turn versioning on for a workspace and modify a file: this validates `versions` implicit DS
- Create a Cell with no root folder
- Add a few files in both `My Files` and `Common Files` folder: you can then double check that the crorresponding buckets in your S3 account contain the expected tree structure.
