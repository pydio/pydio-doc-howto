### ADVANCED SETTINGS OF THE "LDAP/AD" AUTHENTIFICATION
In this guide we will look at the specifics that Pydio offers when you're authenticating with an LDAP/AD server and give you examples to make it easier to use it.

First and foremost you have enabled the LDAP Authentication and you **[binded your server](https://pydio.com/en/docs/v8/binding-ldapad-server)**, second you have basic knowledge of the **[Groups & Roles](https://pydio.com/en/docs/v8/inheritance-groups-and-users-roles)** feature.

### GROUP MAPPING

[:image-popup:/authentication/auth_group_LDAP.png]

As you can see the Group Schema is sort of self explanatory but dont worry i will go in details and give you an example to help you have an up and running Pydio with LDAP/AD authentication.

+ **Groups DN** : you can map groups with a Distinguished Names (you can add as many as you want)
```
for example : OU=Sales,DC=Volvo,DC=COM
CN stands for Common Name,OU for Organizational Unit and DC for Domain Component
```
+ **LDAP Groups Filter** : you can specify the group scheme that you want Pydio to load/filter
```
for example : OU=groups
OU stands for Organizational Unit and is here the object class
*Object Class Specifies the LDAP object class value that defines groups in the 
directory service.
```

+ **Group Attribute** : it's the main attribute that you'll be using as a label
```
for example : CN=group
```

+ **Role Prefix ( for memberOF)** : you can put a prefix to make it easy to search, when you're mapping memberOF
```
put the prefix of your choice to make it easy to know which LDAP/AD Users are part of what in Pydio
```

### ATTRIBUTES MAPPING

[:image-popup:/authentication/auth_attributes_LDAP.png]

Here you can map attributes in a more precise way so that they match your Pydio's attribute.

> I will give you an example of one of the many possibilities that you can achieve with the attribute mapping

+ **LDAP attribute** : the ldap attribute of your choice that you're going to map
```
for example : ON=Vendors
```
+ **Mapping Type** : the TYPE of the Pydio parameter/path/... that you want the mapping to target
``` 
for example : Role ID
(we want Vendors to be a role in Pydio)
```
+ **Plugin parameter** : the VALUE of the Pydio parameter/path/... that you're targeting 
``` 
for example : ExampleROLE
(the ID of the role that we want to be mapped with our LDAP attribute)
```
### ADVANCED PARAMETERS

[:image-popup:/authentication/auth_adv_param_LDAP.png]

You can set advanced parameters to have a more personalized experience with your Pydio

+ **Fake Member from** : if you dont have the memberOF attribute/overlay you can enter a group attribute that has the members ids to fake one on Pydio.

+ **Search MemberOF recursively ( with AD only)** : if you want to recursively search for values of memberOF.

+ **Fake MemberOF value of member/memberUid attribute of group** :  if you enable it the user attribute will be DN and if its disabled it will be CN.

+ **Search Users by Attribute** :  you can specify the parameter that you want to be used for autocompletion when searching instead of the user ID

+ **LDAP Server page size** : the size of the LDAP Server page

+ **Use referral bind** : The Bind operation allows credentials to be exchanged between the client and server to establish a new authorization state.

+ **Cache Users Count ( in hours )** : if you want to locally cache the number of users for X hours, if you have huge directories it can be really a good to enable it



### AUTH DRIVER COMMONS

[:image-popup:/authentication/auth_driver_common_LDAP.png]

+ **Auto Create User** : if its enabled it will automatically create a user when you're using a remote authentication system.

+ **Login Redirect** : you can choose to perform a redirection to a given URL after a login operation.

+ **Administrator Login** : you can choose a user to be a default admin by specifying a User ID.  

+ **Auto apply role** : you can choose to automatically apply a role to a user whenever he's authenticating through this driver.



> And thats just a sample of what you can achieve with Pydio, i let you try and custom everything to fit what you want.

### TROUBLESHOOTING LDAP

+ **PHP-LDAP extension** : you should make sure that you have php-ldap 
you can use `php -m | grep ldap` to check.
If you do not have the ldap extension you can install it through
aptitude, yum, curl ... by using `install php-ldap`.

> If you want to get more informations about the extension you can visit the [PHP-LDAP](http://php.net/manual/en/ldap.installation.php) page.
    
    
### IF YOUR ISSUE WAS NOT LISTED

You should re-check if the test between your LDAP and Pydio was successful when you were setting it up, you have the **'Try to connect to LDAP'** Button to make sure that you gave the right informations. Otherwise i advise you to reconfigure it from scratch in case you missed a setting.

If none of the above worked you should make sure that your Pydio installation was rightfully done, you can also go check **[Pydio's Troubleshooting](https://pydio.com/en/docs/v8/troubleshooting)** page.

Else you can go on our **[Forum](https://forum.pydio.com/)** and search if someone had a similar issue or you can create a post.