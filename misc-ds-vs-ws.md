When deploying Cells and building your DataSources and Workspaces, you have to pick the right layout both in terms of data security and system scalability. This article gives a couple of recommendation for having a correct balance.


### Datasources "Cost"

Datasource is the level where Cells maintains the consistence. It means that the performance of the service depends on the size of the datasource (the number of files). In the other hand, each datasource runs in a private service and requires a table in the database. More datasources in Cells means more resources for the Cells' services. The ratio between the number of datasource and the size of each one should be selected carefully to optimize the resource usage.

### Using Workspaces

Being defined on the top of datasources, Workspaces are the right level to define access control rules.

A datasource per project is absolutely not a good solution. Instead, you may organize several projects in a datasource. For example, you have a directory /data/cells which stores users' data. The directory structure would be:

- /data/cells/datasources/ds1
  - /data/cells/datasources/ds1/project1
  - /data/cells/datasources/ds1/project2
  - /data/cells/datasources/ds1/project3
- /data/cells/datasources/ds2
  - /data/cells/datasources/ds2/project21
  - /data/cells/datasources/ds2/project22
  - /data/cells/datasources/ds2/project23        


Over the time, as the number of project grows, you can add more new datasources as well as a sub folder for each new project

- /data/cells/datasources/dsx
  - /data/cells/datasources/dsx/projectx1
  - /data/cells/datasources/dsx/projectx2
  - /data/cells/datasources/dsx/projectx3

    
### Caveats

Please keep in mind that you can create new datasource only on the first-level of /data/cells/datasources directory. A new datasource in any second-level sub-directory of /data/cells/datasources/ will result in a nested-service-object error.

For further information, you can see .minio.sys directory in /data/cells/datasources. This is the data for the object service, all datasources that are then defined in this location depend on it.

See also: https://pydio.com/en/docs/cells/v4/datasources-overview