This is an example of connecting Pydio Cells with an open-source saml server. SimpleSAMLphp is written in native PHP that deals with authentication. For more information, please visit [this link](https://simplesamlphp.org/)

## Install and configure SimpleSAMLphp server

It's possible to configure SimpleSAMLPhp to serve as a Service Provider or as Idetity Provider. In this how-to, we are going to setup a Identity Provider which provider services to Cells - a service provider.

Following the docs of simplesamlphp, you can easily download source code and setup a web server on a linux box. For demonstration only, a simple authsource is set in simplesamlphp by copying the sample accompanied source code to config/authsources.php. 
For further information, please visit [this link](https://simplesamlphp.org/docs/stable/simplesamlphp-idp)

```php
<?php
$config = [
    // This is a authentication source which handles admin authentication.
    'admin' => [
        // The default is to use core:AdminPassword, but it can be replaced with
        // any authentication source.
        'core:AdminPassword',
    ],
    'example-userpass' => [
        'exampleauth:UserPass',
        'student:studentpass' => [
            'uid' => ['student'],
            'email' => ['stu01@lab.py'],
            'eduPersonAffiliation' => ['member', 'student'],
        ],
        'employee:employeepass' => [
            'uid' => ['employee'],
            'email' => ['emp001@lab.py'],
            'eduPersonAffiliation' => ['member', 'employee'],
        ],
    ],
];

```

Enable Idp protocol in config/config.php

```php
    'enable.saml20-idp' => true,
    'enable.shib13-idp' => false,
    'enable.adfs-idp' => false,
    'enable.wsfed-sp' => false,
    'enable.authmemcookie' => false,

```

Configure certificate/key for idp in metadata/saml20-idp-hosted.php

```php
    // X.509 key and certificate. Relative to the cert directory.
    'privatekey' => 'key.pem',
    'certificate' => 'cert.pem',
```

Add Cells - Service Provider into metadata/saml20-sp-remote.php
```php
/*
 * Example SimpleSAMLphp SAML 2.0 SP
 */
$metadata['https://cells.lab.py/auth/dex/callback'] = [
    'AssertionConsumerService' => 'https://cells.lab.py/auth/dex/callback',
    'SingleLogoutService' => 'https://cells.lab.py/auth/dex/logout',
];
```

## Add new connector in Cells

You've done the configuration of a saml server and registered cells with id "https://cells.lab.py/auth/dex/callback". The saml server are able to accept the request comming from you Cells and return back the result of authentication to callback url.

[:image-popup:connectors/connector_simplesaml_01.png]
