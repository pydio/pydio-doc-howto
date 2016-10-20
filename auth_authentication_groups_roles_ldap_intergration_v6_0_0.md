In previous version, you used to work with ldap plugin for authentication on Pydio. In this article, we would like to introduce new functions on ldap plugin in Pydio version 6.0.0.

### The paged results control

The paged results control is very useful for ldap plugin which need to receive search results in a controlled manner limited by the page size. The page size can be configured on ldap plugin as per availability of its resources, such as bandwidth, processing capacity or number of records.

In ldap, the number of records in search result is limited and variable depend on ldap system. Such as on AD is 1000 and on openldap is 500. In early version, you have to modify the this value on ldap manually.

ldap plugin on Pydio v6 takes control on this value and you can configure it easily. By default is 500 means that 500 records for each page.

Note: paged result control support by PHP version >=5.4

[:image-popup:authentication/groups_and_roles_with_ldapad_integration_v6.0.0/ldapserverpagesize_v6.png]
### Search on multi attributes

When doing share files/folder, you have to search user list and select people who you are looking for. This action can be translated on Pydio to a ldap query to search your input text that match with only one attribute on ldap. Obviously, its result is limited.

Supposed that you are looking for a person who you remember only his name commence by ‘John’, but on displayName this attribute is ‘Kenedy John’ for example. You can not find his name on the list. On new version, you can find out this person if you specify the attributes to search for: displayName,givenName,cn

[:image-popup:authentication/groups_and_roles_with_ldapad_integration_v6.0.0/ldapsearchuserpyattrs_v6.png]

### Fake memberOf

Mapping ldap group to Pydio roles is very useful to intergrate Pydio in your system. A difficulty is some ldap system do not support memberOf attributes natively. So you have to install memberOf overlay on ldap. However, one of Pydio objective is minimizing the modification on your system. This is a reason why we develop a new ‘fake’ attribute memberof on ldap user object. With this technique, you can map memberof to roleId as memberof on AD. Of course, this can reduce the performance of Pydio.

[:image-popup:authentication/groups_and_roles_with_ldapad_integration_v6.0.0/ldapfakememberof_v6.png]

In configuration, ‘Fake memberOf from’ is an attribute of ldap group object that Pydio will use to build memberof. It can be ‘member’ or ‘memberUid’ depend on your ldap.

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

It aslo can help you to add prefix of all roleID who mapped from ldap for further purposes.

According to new feature, In Application core>Configuration management> Roles /groups Directory Listing session, you have more option

[:image-popup:authentication/groups_and_roles_with_ldapad_integration_v6.0.0/ldapMaprole_v6.png]

- Display roles and/or groups: For sharing, you need a list of user/role/group to select. You can limit this list for user only, or role, or group …
- Role prefix: should the same value in ldap configuration indicate that listing only roles who has prefix
- List roles by: If role listing is enable, there is two options are listing all role on Pydio, or only the roles that current user belong to
- Exclude roles/Include roles: can be a list with semicolon separator or a preg  
  ie: preg:^root|^system or root,system

