### Php LDAP extension
The first thing to check when trying to connect to an LDAP-based directory is that your php does have the LDAP Extension active.

See http://php.net/manual/en/ldap.installation.php and particularly the comments for many ways of installing this extension, depending on your server OS.

 

### Binding or anonymous?
Depending on your directory configuration, you may have to provide an “administrator” user access to be able to query the directory. The following two parameters can be left empty or not, accordingly :

[:image-popup:authentication/troubleshooting_ldapad/screenshot-2013-06-04-at-15-16-35.png]

You should use the “Try to connect to LDAP” button, with the “Test” user field : it will try to connect to the server, and if successful, try to find the test user in the directory.

### Test user is OK but cannot login
This seems to indicate that the password is not correctly passed : did you set the “Transmit Clear Pass” parameter to true?

### I cannot share folder or create shared users!
You must set up a “secondary instance” in the Authentication panel, so that Pydio can handle a “multiple” configuration : reading the master users from the Directory, but also creating new shared users inside its “local” directory. Use either Serial or DB auth storage for this secondary instance.