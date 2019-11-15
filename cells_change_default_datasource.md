In a fresh install of Pydio Cells, the application creates 3 default datasources, namely:

- **pydiods1**: default data source for shared files: the `Common Files` workspace that exists at startup points here
- **personal**: contains "My Files"-like workspaces for each user
- **cellsdata**: is used to create _cells_ when no root node has been defined upon creation

By default, these are created under `<CELLS_WORKING_DIR>/data/`.

Furthermore, there are three S3 buckets that are used to store internal and technical data:

- **thumbs**: stores the thumbnails
- **versions**: stores the old versions of the files, when the versioning is enabled on a given DS
- **binaries**: service docstore, stores various things, for instance: avatars, custom background...

By default, these 3 buckets are siblings of the `pydiods1` folder that can be found under `<CELLS_WORKING_DIR>/data/`. This is defined in the main config: for each one of these _technical buckets_ we specify a name and a DS. Under the hood, the system retrieves the object service that is linked to the specified DS and thus retrieves S3 connection information ( (more info on this here)[TODO] ).

In the default setup, these buckets are created during installation process. In an S3 setup, you must first insure the buckets have been created.

If you want to change the default layout, this is possible, but not straightforward; you have to understand what is hapenning and follow exactly the below steps. We plan on enhancing this in a future version soon.

## Change default location still using local file system

### Are you sure?

Please, first take a step back and ask yourself if the various configuration parameters offered by the application are not enough to reach your goal. Typically, if you want to change default location of all those object to somewhere else on your local file system, you migth just have to set the `CELLS_DATA_DIR` environment variable (that is by default set to `<CELLS_WORKING_DIR>/data/`) before launching the installation process.  
You will then have a fully working instance in no time.

If you want to have a more specific layout, go on reading.

### My Files and Cells Data

It is quite easy to tweak these:

#### Enterprise Distribution

You just have to edit the default template path by going to:
`Admin console >> show advanced parameters >> Left column menu >> Data management >> Template Path`

If you have created new datasources, you might just then impact the default template paths to use you newly created DS, for instance:

```conf
# Cells
Path = DataSources.cellsdatanew + "/" + User.Name;
# My Files
Path = DataSources.personalnew + "/" + User.Name;
```

_To use auto-completion: first move your cursor after the dot, then press `CTRL + SPACE`_

If you want to mutualise the DS, you might also implement a more complex layout. For instance, you might do the following:

- create a new datasource (let say `dynamicws`)
- create a temporary workspace that point to it and create 2 folders `personal` and `cellsdata` in it
- delete the temporary workspace
- impact the template path to have:

```conf
# Cells
Path = DataSources.dynamicws + "/cellsdata/" + User.Name;
# Personal Files
Path = DataSources.dynamicws + "/personal/" + User.Name;
```

#### Home distribution

As you cannot edit template paths, the only solution you have is to delete the existing default `cellsdata` and `personal` DSs and recreate them where it fits you most. You do not have to change anything else then.

### Common files

This is just a default workspace with Read/Write permissions for everyone. You can easily edit the workspace to point to another datasource.

### Technical buckets

The trick here is the way the buckets connection info are computed. It was our implementer choice to say:

- we will use a raw s3 bucket to store and retrieve our internal technical stuff (typically thumbnails)  
- these buckets do not need to be real datasources (they do not need neither index nor sync mechanism)
- yet, as each defined datasource has an `object` service that might expose more than one bucket, let use one of these
- so let's define a default datasource, retrieve its object service and assume it also exposes the additionnal buckets we need.

In a fresh vanilla install, the default datasource is `pydiods1` that exposes the `<CELLS_WORKING_DIR>/data/pydiods1` folder of your file system. The underlying object service exposes **all** then folders that are in`<CELLS_WORKING_DIR>/data/` as S3 buckets. So we just create the `thumbs`, `versions`, and `binaries` folders during installation.

If you were to change this, you can:

- create a new datasource that points towards the desired path (for instance `defaultds`)
- adapt your `pydio.json` file to have something like:

```json
"defaults": {
    "database": "...",
    "datasource": "defaultds",
    "url": "https://pydio.example.com",
    "urlInternal": "https://pydio.example.com:443"
  },
```

- save and restart the app.

## Switch to Amawon S3

You can also tune your configuration to rather have all your Datasources and Bucket on S3. We present here a simple setup that introduces well the steps you have to go through. Feel free to then adapt to your specific use case.

### Create the required buckets

The main difference here is that you (still) have to manually create your buckets in your S3 administration console before creating the DSs in Cells.

Note that buckets names in AWS have to be globally unique, so you cannot use the names they have in the local vanilla setup. Let us use for instance:

- com-example-cells-personal
- com-example-cells-cells
- com-example-cells-common
- com-example-cells-thumbs
- com-example-cells-versions
- com-example-cells-binaries

Note that due to their implicit declaration (see above) via the default DS, the thumb, versions and binaries must be buckets of the **same** storage than the ds that is defined as default DS. Otherwise you will have to tweak the config even a little bit more.

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

## Troubleshooting

Note that the above procedure does not include migration of existing data, after restart you will then typically have lost all existing thumbnails that will be blank.

In order to double check everything is working correctly, you might perform following checks:

- Change your account avatar: it insures the `binaries` implicit DS runs OK
- Upload an image: this checks the `thumbs` implicit DS
- Turn versioning on for a workspace and modify a file: this validates `versions` implicit DS
- Create a Cell with no root folder
- Add a few files in both `My Files` and `Common Files` folder: you can then double check that the crorresponding buckets in your S3 account contain the expected tree structure.
