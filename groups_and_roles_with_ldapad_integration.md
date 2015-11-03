### Introduction
When you’re using AjaXplorer with [LDAP/AD authentication](https://pyd.io/administrator/configuring-global-parameters/setup-authentication-driver/binding-to-an-ldapad-server/) and your LDAP directory supports the “memberOf” attribute (i.e. you have Microsoft Active Directory, or an LDAP server with the “memberOf” overlay set up), you can use your Active Directory groups in AjaXplorer.

This is a special case of mapping LDAP attributes : you use the mapping mechanism of the LDAP/AD Authentication plugin to map LDAP/AD groups that a user is member of (in LDAP/AD) to groups or roles in Ajaxplorer. A typical use case would be that you’re using a CIFS or SMB storage backend with ACL based on LDAP/AD groups, you want to expose those files trough AjaXplorer with your workspace ACL in Ajaxplorer mirroring your filer’s ACL. Retrieving LDAP/AD group membership of your users would make that possible.

You will need a good understanding of [Ajaxporer’s Groups and Roles](https://pyd.io/administrator/users-managements-roles/inheritance-groups-and-users-roles/) to make good use of this featurel. Most notably, you’ll find that for managing ACL on workspaces, AjaXplorer uses **roles** while in a Windows environment you’d expect to be using AD **groups** for file ACL. So, depending on what it is you’re trying to achieve, you might want to map  LDAP/AD **groups** to  AjaXplorer **roles** in some cases, while in other situations you ‘d want to map between groups.
It is also  possible to do both.

Note :

+ in AjaXplorer (at the time of writing) a user can only be member of 1 group.
+ When retreiving group membership from a Directory server where users can be member of several groups, or where groups can in turn be member of other groups (so-called nested groups), AjaXplorer will only map one group.
It’s rather unpredictable which group AjaXplorer will end up using.
+ an AjaXplorer user, however, can have several different roles
 

**update (december 2014)** : Pydio 6 has newer features re. LDAP and AD integration : [Groups and Roles with LDAP/AD integration [v6.0.0]](https://pyd.io/groups-and-roles-with-ldapad-integration-v6-0-0/)

 

 

### Mapping Ldap Attributes (revisited)
You map LDAP/AD attributes to AjaXplorer parameters in the Global Configurations > Core Configs > Authentication panel, using a group of parameters  that work in triplet : {ldap attribute, mapping type, plugin parameter}. This group is repeatable, so that you can retrieve and map several distinct LDAP/AD attributes, or you can map the same LDAP/AD attribute to multiple parameters.

Mapping type can take 4 values :

+ **_Plugin parameter_** : will enter the retrieved LDAP value to any arbitrary plugin parameter of the current user.
+ **_Role id_** : The value of the LDAP attribute will be mapped to a Role ID. This role will be assigned to the current (logged on) user. Creates the role if it does not exists.
+ **_Group Path_** : Similar to role ID : the retrieved LDAP value is mapped to the “group path” for the current user, making the user member of that group in AjaXplorer.
+ **_Profile_** : set the specific profile to the current user (to be used with caution).
The ones we’re interested in are **_“Group Path”_** and **_“Role ID”_**.

 

### Mapping LDAP/AD Groups to AjaXplorer Groups
In the Authentication panel, you can add some values to the LDAP/AD config :

+ **Group DN** : (optional ) .  if you have collected your LDAP/AD groups in a dedicated OU. This improves the performance when AjaxPlorer queries the Directory, and can also (maybe ?) be used to limit the groups AjaXplorer will see.
+ **Group Attribute** : the attribute that ajaxpplorer should use as name/label for the group.
For “Group attribute”, chose something that identifies the group in a human-friendly way. In Microsoft AD, “sAMAccountname” appears to work well.

[:image-popup:authentication/groups_and_roles_with_ldapad_integration/mapgroupsroles.png]

Then, you add

1. Ldap attribute: `memberOf`
2. Mapping type: `Group path`
3. Plugin Parameter:  (remains empty)
 

That’s all.
When a user logs on, AjaXplorer will check which groups the user is member of (in LDAP/AD), and create the corresponding group and membership in AjaXplorer.

 

### Mapping LDAP/AD Groups to AjaXplorer Roles
This is entirely similar to mapping groups :

1. Ldap attribute: `memberOf`
2. Mapping type: `Role id`

“Plugin parameter” remains blank. Group DN can remain the same as for group mapping.

The main difference is that AjaXplorer seems to ignore the “Group attribute” setting and use the group’s LDAP DN as its label in Ajaxplorer.

### Work in progress
This Howto only outlines the basics of integrating LDAP/AD group membership into AjaXplorer Groups and Roles.
Likewise, the integration of LDAP/AD groups in AjaXplorer is also a little rough around the edges. This may change in the future.

Meanwhile, Search the forums for LDAP/AD Groups for additional information.