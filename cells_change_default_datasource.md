You can change the default datasources with another storage, but you will need to execute those exact steps to make it work.

### Create the datasources and required buckets.

First create the datasources that you wish to use, for our example we are going to use a remote S3 datasource.
Let's replicate the exact datasources that where by default, but on our S3 remote storage.

Basically create the required buckets and create the s3 connection to each one of them depending on your datasource. (for each datasource, use a different bucket)

**The 3 following buckets must be created on your storage** ():

* **thumbnail** (is used to store the thumbnails).
* **versions** (is used for the versioning).
* **binaries** .

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

* if you don't have thumbnails refer to the 1st paragraph about the mandatory buckets, you might have forgotten to create them.
