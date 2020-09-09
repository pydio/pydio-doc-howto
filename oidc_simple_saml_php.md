## About SimpleSAMLphp

SimpleSAMLphp is written in native PHP and deals with authentication. For more information, please visit [this link](https://simplesamlphp.org/)

## Install and configure SimpleSAMLphp server

Download the source code [here](https://simplesamlphp.org/download).

Install following [this guide](https://simplesamlphp.org/docs/stable/simplesamlphp-install).

Setup SimpleSAMLphp as an [Identity Provider](https://simplesamlphp.org/docs/stable/simplesamlphp-idp)

## Register Cells as a Service Provider in SimpleSAMLphp

metadata/saml20-sp-remote.php
```php
/*
 * Example SimpleSAMLphp SAML 2.0 SP
 */
$metadata['https://cells.lab.py/auth/login/callback'] = [
    'AssertionConsumerService' => '<your_callback_url>',
    'SingleLogoutService' => '<your_logout_url>,
];
```

*:warning: The callback url used here is generated in the next step. The format of the url may vary depending on the version of Cells so please refer to your admin console.*

## Add a SimpleSAMLphp connector in Cells

Navigate to the Admin Console &gt; Authentication &gt; OAUTH2 / OIDC

Use the following configuration example to create a new connector :

### Connector Options
- Connector type : ```SAML```
- Id: ```your_id_here```
- Name: ```your_name_here``` (the name will appear to the end user in the Login dialog)

### SAML Options
- SSO URL user for POST Value: ```https://<your_saml_url_here>/saml2/idp/SSOService.php```
- CA to use when validating the signature of the SAML response : ```<your_certificate_path_on_the_cells_server>```
- Callback URL : (**generated - use it to register cells as a relying party trust in adfs**)
- Name of the attributes to map in the ID Token Claims: (this will depend on your configuration - see excerpts below)
  - Username: ```uid```
  - Email: ```email```
- Manually specify issuers value:
  - Entity issuer: ```https://<your_cells_url_here>```
  - SSO issuer: ```https://<your_saml_url>/saml2/idp/metadata.php```

[:image:connectors/connector_simplesaml_01.png]

## Excerpts from local testing configuration

Enable Idp protocol in config/config.php
```php
    'enable.saml20-idp' => true,
    'enable.shib13-idp' => false,
    'enable.adfs-idp' => false,
    'enable.wsfed-sp' => false,
    'enable.authmemcookie' => false,
```

config/authsources.php
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

metadata/saml20-idp-hosted.php
```php
    // X.509 key and certificate. Relative to the cert directory.
    'privatekey' => 'key.pem',
    'certificate' => 'cert.pem',
```