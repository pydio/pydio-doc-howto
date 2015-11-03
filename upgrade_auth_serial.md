### Technical Background
In Pydio 6, the authentication modules were split between auth backend and auth frontends. The idea is to decouple the user “credentials” method (frontend) and how we do use those credentials to authenticate (backend). The mix of both concepts in v5 was preventing to easily implement a new way of capturing credentials (e.g. One-Time-Password) without locking this to a predefined authentification backend (sql, serial, etc..).

For this reason, the following plugins where now replaced by “authfronts” plugins: serial_otp (get credentials by negociating a one-time-password with third party service), basic_http (retrieve standard HTTP auth credentials already retrieved by the web server) and CAS (negociate authentication with a CAS server).

The common process will be to first DISABLE the auth plugin, then upgrade, the enable the corresponding authfront plugin.

### Migrating auth.serial_otp to authfront.otp
[TODO]

### Migrating auth.basic_http to authfront.http_server
Make sure to be already logged in.

Disable auth.basic_http, and switch to either auth.serial or auth.sql depending on your configuration.

Perform upgrade, and then only, go to the Authfront plugins lists (under Features Plugins) and activate the new authfront.http_server plugin.

### Migrating auth.cas to authfront.cas
If you are working with auth.cas as authentication engine on your Pydio, you can switch to use authfront.cas.

#### _Step 1_.

Be sure that all configuration has already been safe by at least one backup version.
Infomration important:
– Database
– data/plugins/boot.conf/bootstrap.json (this file is very important, like masterboot of your HDD)
I supposed that you have a mysql server to temporarily switch to use auth.sql during the migration.

#### _Step 2: Active auth.sql_

In Global Configuration>Authentification> Secondary instance
Mode Master/Slave
Instance Type: select DB Auth Storage. Don’t forget to click on “Install SQL Tables” button
Transmit Clear Pass: Yes
Auto Create User: Yes

Don’t logout
Now, you create a user admintemp and set this user as administrative account. This to besure that once you turn off auth.cas, you still have an account to access to Pydio.

replace data/plugins/boot.conf/bootstrap.json by this file (replace some infos on your sql database)

    {
      "core.conf":{
        "UNIQUE_INSTANCE_CONFIG":{
          "instance_name":"conf.sql",
          "group_switch_value":"conf.sql",
          "SQL_DRIVER":{
            "core_driver":"core",
            "group_switch_value":"core"
          }
        },
        "DIBI_PRECONFIGURATION":{
          "mysql_username":"pydio",
          "mysql_password":"pydiopsss",
          "mysql_host":"localhost",
          "mysql_driver":"mysql",
          "mysql_database":"py525",
          "group_switch_value":"mysql"
        }
      },
      "core.auth":{
        "MASTER_INSTANCE_CONFIG":{
          "instance_name":"auth.sql",
          "group_switch_value":"auth.sql",
          "SQL_DRIVER":{
            "core_driver":"core",
            "group_switch_value":"core"
          }
        }
      }
    }

Clear cache and you can log into Pydio with admintemp created above.

#### _Step 3: Update pydio from 5.2.5 to 6.0.0_

#### _Step 4: Active authfront.cas_

Supposed that you got the success after upgrade to version 6.0.0
Enter Global Configuration> Feature plugins > Authentification frontends >
Enable CAS frontend: YES

+ ORDER: 11 (default)
+ Create user: YES
+ Protocol type: Sessions Only
+ Some of following item look like in auth.cas
+ CAS Server: server address or ipaddress
+ CAS Port: port server cas listens to
+ CAS URI: uri of server cas. i.e /cas
+ Logout URL: URL of server cas to call logout action

Modify login Page:

+ YES: The login screen will be modified and user have opportunity to select the way to do the authentication by CAS or by internal auth plugin.
If you were used to using auth.cas, you can see that once auth.cas is acive, the user cannot use another authentification method than CAS because of redirection. This option is ON by default
+ Additional roles for user …:
+ Certificate Path: path to certificate file of the chain to CA who issues certificate of CAS server
+ phpCAS mode
There are two modes for this plugin client mode and proxy mode.
– Client mode: This plugin works as a CAS client
– In Proxy mode, this plugin not only authenticate user in CAS server but also proxy authentification service to third service such as samba, imap. For further information please visit https://wiki.jasig.org/display/casc/phpcas
 