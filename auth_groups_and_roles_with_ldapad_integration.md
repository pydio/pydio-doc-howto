### Introduction
When you’re using Pydio with [LDAP/AD authentication](https://pydio.com/en/docs/v8/binding-ldapad-server) and your LDAP directory supports the “memberOf” attribute (i.e. you have Microsoft Active Directory, or an LDAP server with the “memberOf” overlay set up), you can use your Active Directory groups in Pydio.

This is a special case of mapping LDAP attributes : you use the mapping mechanism of the LDAP/AD Authentication plugin to map LDAP/AD groups that a user is member of (in LDAP/AD) to groups or roles in Ajaxplorer. A typical use case would be that you’re using a CIFS or SMB storage backend with ACL based on LDAP/AD groups, you want to expose those files trough Pydio with your workspace mirroring your filer’s ACL. Retrieving LDAP/AD group membership of your users would make that possible.

You will need a good understanding of [Pydio’s Groups and Roles](https://pydio.com/en/docs/v8/inheritance-groups-and-users-roles#content) to make good use of this featurel. Most notably, you’ll find that for managing ACL on workspaces, Pydio uses **roles** while in a Windows environment you’d expect to be using AD **groups** for file ACL. So, depending on what it is you’re trying to achieve, you might want to map  LDAP/AD **groups** to  Pydio **roles** in some cases, while in other situations you‘d want to map between groups.
It is also possible to do both.

Note :
+ When retreiving group membership from a Directory server where users can be member of several groups, or where groups can in turn be member of other groups (so-called nested groups), Pydio will only map one group.
It’s rather unpredictable which group Pydio will end up using.
+ a Pydio user, however, can have several different roles
 

### Mapping Ldap Attributes
You map LDAP/AD attributes to Pydio parameters in the **Menu > Authentification** you choose instance type (master or secondary driver) LDAP/AD Directory,
using a group of parameters  that work in triplet : {ldap attribute, mapping type, plugin parameter}. This group is repeatable, so that you can retrieve and map several distinct LDAP/AD attributes, or you can map the same LDAP/AD attribute to multiple parameters.


[:image-popup:authentication/ldap_auth/Auth_LDAP.png]

Mapping type can take 4 values :

+ **_Plugin parameter_** : will enter the retrieved LDAP value to any arbitrary plugin parameter of the current user.
+ **_Role id_** : The value of the LDAP attribute will be mapped to a Role ID. This role will be assigned to the current (logged on) user. Creates the role if it does not exists.
+ **_Group Path_** : Similar to role ID : the retrieved LDAP value is mapped to the “group path” for the current user, making the user member of that group in Pydio.
+ **_Profile_** : set the specific profile to the current user (to be used with caution).
The ones we’re interested in are **_“Group Path”_** and **_“Role ID”_**.

 

### Mapping LDAP/AD Groups to Pydio Groups
In the Authentication panel, you can add some values to the LDAP/AD config :

+ **Group DN** : (optional ) .  if you have collected your LDAP/AD groups in a dedicated OU. This improves the performance when Pydio queries the Directory, and can also (maybe ?) be used to limit the groups Pydio will see.
+ **Group Attribute** : the attribute that Pydio should use as name/label for the group.
For “Group attribute”, chose something that identifies the group in a human-friendly way. In Microsoft AD, “sAMAccountname” appears to work well.

[:image-popup:authentication/groups_and_roles_with_ldapad_integration/group_ldap.png]

Then, you add

1. Ldap attribute: `memberOf`
2. Mapping type: `Group path`
3. Plugin Parameter:  (remains empty)
 

That’s all.
When a user logs on, Pydio will check which groups the user is member of (in LDAP/AD), and create the corresponding group and membership in Pydio.

 

### Mapping LDAP/AD Groups to Pydio Roles
This is entirely similar to mapping groups :

1. Ldap attribute: `memberOf`
2. Mapping type: `Role id`

“Plugin parameter” remains blank. Group DN can remain the same as for group mapping.

The main difference is that Pydio seems to ignore the “Group attribute” setting and use the group’s LDAP DN as its label.


### The paged results control

The paged results control is very useful for ldap plugin which need to receive search results in a controlled manner limited by the page size. The page size can be configured on ldap plugin as per availability of its resources, such as bandwidth, processing capacity or number of records.

In ldap, the number of records in search result is limited and variable depend on ldap system. Such as on AD it's 1000 and on openldap it's 500. In early version, you had to modify this value on ldap manually.

Ldap plugin on Pydio takes control of this value and you can configure it easily. By default it's 500 meaning 500 records for each page.

Note: paged result control is supported by PHP version >=5.4

[:image-popup:authentication/ldap_auth/server_size_page_LDAP.png]
### Search on multi attributes

When doing share files/folder, you have to search the user list and select people who you are looking for. This action can be translated on Pydio to a ldap query to search your input text that match with only one attribute on ldap. Obviously, its result is limited.

Assuming that you are looking for a user that you only remember by his name starting with ‘John’, but on displayName this attribute is ‘Kenedy John’ for example, you can find this user if you specify the attributes to search for: displayName,givenName,cn

[:image-popup:authentication/ldap_auth/search_users_by_attribute_LDAP.png]

### Fake memberOf

Mapping ldap group to Pydio roles is very useful to intergrate Pydio in your system. A difficulty is that some ldap system do not support memberOf attributes natively. So you have to install memberOf overlay on ldap. However, one of Pydio's objective is minimizing the modification on your system. That's why we developed a new ‘fake’ attribute memberof on ldap user object. With this technique, you can map memberof to roleId as memberof on AD. Of course, this could reduce the performances of Pydio.

[:image-popup:authentication/ldap_auth/fake_member_from_LDAP.png]

In configuration, ‘Fake memberOf from’ is an attribute of ldap group object that Pydio will use to build memberof. It can be ‘member’ or ‘memberUid’ depending on your ldap.

In this example, the ‘member’ should be taken for configuration ‘Fake memberOf from’

<code>
ldapsearch -x -h 10.11.5.2 -b ou=groups,dc=vpydio,dc=fr "(&(objectClass=groupOfNames)(cn=groupTest))"

dn: cn=groupTest,ou=groups,dc=vpydio,dc=fr
member: uid=test1,ou=people,dc=vpydio,dc=fr
member: uid=test2,ou=people,dc=vpydio,dc=fr
member: uid=test4,ou=people,dc=vpydio,dc=fr
member: uid=test5,ou=people,dc=vpydio,dc=fr
member: uid=test78,ou=people,dc=vpydio,dc=fr
</code>

It also can help you add prefixes to all roleID that mapped from ldap for further purposes.

According to the new feature, In **Configuration Backends > Roles / Groups Directory Listing session**, you have more options

[:image-popup:authentication/ldap_auth/listing_LDAP.png]

- Display roles and/or groups: For sharing, you need a list of user/role/group to select. You can limit this list to users only, or roles, or groups …
- Role prefix: should be the same value as in the ldap configuration, indicates that to list only the roles that have this prefix
- List roles by: If role listing is enabled, there are two options that are listing all the roles on Pydio, or only the roles that current user belongs to
- Exclude roles/Include roles: you can filter which roles you wheter or not want to be listed, can be a list with a semicolon separator or a preg  
  ie: preg:^root|^system or root,system






### Work in progress
This Howto only outlines the basics of integrating LDAP/AD group membership into Pydio Groups and Roles.
Likewise, the integration of LDAP/AD groups in Pydio is also a little rough around the edges. This may change in the future.

Meanwhile, Search the forums for LDAP/AD Groups for additional information.