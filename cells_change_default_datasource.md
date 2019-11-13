In a fresh install of Pydio Cells, the application creates 3 datasources (DS) that are used to store internal and technical data, namely:

- **pydiods1**: stores thumbnails
- **personal**: contains "My Files"-like workspaces for each user
- **cellsdata**: is used to create _cells_ when no root node has been defined upon creation

By default, these are created under `<CELLS_WORKING_DIR>/data/` but you can change this default location, even to switch to another storage.

In the following example, we describe the exact steps you must execute to move all those default DSs to S3.

### Create the datasources and required buckets

First we replicate the 3 default DSs on our S3 remote storage.

We basically create the required buckets - you must use a different bucket for each DS - and add a S3 connection for each one of them.

**The 3 following buckets must be created on your storage**:

- **thumbnails** (is used to store the thumbnails).
- **versions** (is used for the versioning).
- **binaries** .

### Create the New datasources

The array above gives you an idea to replicate the exact functionning default datasources.

| **default datasource**  | **default workspaces using the datasource**  | **NewDatasource**  |
|---|---|---|
| common files  | pydiods1  | datasourceNew  |
|  personal files | my-files  | personalNew  |
| cellsdata | *see below for details*|cellsdataNew

**_cellsdata is used for when you create a cell without a root node, it's used for it's default node_**

### Change template paths to point to the new datasources

Template path, by default they look like this.
![Screenshot 2018-12-18 at 15.15.08](https://i.imgur.com/c56ifQB.png)

_You can press CTRL + SPACE to prompt auto-completion, first move your cursor to the dot, on it's right, then press the auto-completion shortcut_.

We need to change the paths to the new datasources to whom they are pointing.

Let's start with CELLS, the part that we need to modify is this one:

| **Before** | **After** |
| --- | --- |
| Path = DataSources.**cellsdata** + "/" + User.Name;  |  Path = DataSources.**cellsdataNew** + "/" + User.Name; |

> For instance if we want to use our new datasources we will change it to this ( you can refer to the array above ):

And for MY-FILES:

| **Before** | **After** |
| --- | --- |
| Path = DataSources.**personal** + "/" + User.Name;  |  Path = DataSources.**personalNew** + "/" + User.Name; |

> (the bold part is the datasource name that has to be changed)

Now let's proceed to the next step.

Inside the `pydio.json` file that is located in your `~/.config/pydio/cells/pydio.json` folder ( the _~_ being the home of the user that launched the cells binary )

[:image-popup:/devops/change_default_ds.png]

Modify the higlighted part (by default it uses the default datasource `pydiods1`) to your new datasource for instance `pydiods1new`.

### Change the workspaces default datasource

For the personal-files you must choose the template path and not the datasource by name as seen on the screenshot below:

![Screenshot 2018-12-18 at 15.29.47](https://i.imgur.com/AsSImrK.png)

For common files you can change the datasource to to the new one.
For instance by default it uses `pydiods1`, change it to `datasourceNew` ( refer to the array at the begining to see the default ds matching it )

### Final STEP

You can now delete the default datasources if you wish not to use them.

### Troubleshooting

- if you don't have thumbnails refer to the 1st paragraph about the mandatory buckets, you might have forgotten to create them.
