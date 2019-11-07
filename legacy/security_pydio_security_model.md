##Security basics
### Architecture
Storing all your organization information in a SaaS-based Box service can prove to turn into a nightmare: are the provider’s servers located in a country that is compliant with your jurisdiction? Is the service provider trust-worthy? Can he actually read the data? And eventually let third-party read them (like NSA…)? And what if a trainee has opened a “box” last year, and no one can close it anymore? Will she still see all the documents forever??

Using Pydio is a simple solution to this: take back the control, by installing your enterprise box on your own servers. This does not necessarily means a server inside your office building, but simply leveraging your existing architecture. Maybe the enterprise already owns a piece of storage somewhere in a data-center, that has been negotiated with a local dealer, and why should you migrate everything to another storage mechanism? Or you have set up a public-cloud based infrastructure, but on which you decide to encrypt everything, keeping away the keys from the provider, and that you access only through a VPN…

As such, by construction, Pydio provides a “passive” security model that will raise the standards in most cases.

### Open Source. Mature.
What open source guarantees, is a better quality and a more secure code: there can’t be any spyware or malware delivered with the solution, as a whole community is watching the code closely, and it would just be disclosed in no time. Furthermore, security experts regularly audit the code for vulnerabilities, and when they are found, they are disclosed through the public channels. This forces the contributors to be always listening and reactive, to avoid losing the trust of the community.

Pydio is a 5 years old project, with more than 500 000 downloads. Its maturity is a strong gage of security.

### Encryption
Encrypting the data during communication and at rest is a recommended procedure. The first is easier to provide that the latter, and should always be set up.

By using TLS encryption ( = accessing Pydio through HTTPS instead of HTTP ), you will defeat the most easiest flaws that can happen. When connecting from a public Wi-Fi, it’s really easy to intercept the communication, and sending everything in clear is the best way to get information stolen.

For encryption Pydio currently provides an EncFS implementation, that is based on a encryption of the data on the file system, and the decryption is done only during the user session. Full client-side encryption is still a hard one, as it must be in that case provided by the HTML clients themselves (= the browser), requiring a lot of computer processing.

## Common security flaws
### Web-App protection
Pydio is created with security in mind: you will put your IP in there, you HAVE to be protected. Beyond the common authentication features and protections barriers that every software like this provides, the web application has been regularly tested and coded with the common security flaws in mind.

Below are some of the most current and how they are handled.

+ XSS: Sanitization of many parameters, included filenames and metadata.
+ CSRF: a secure token must be provided for all requests
+ Brute-force login: After three wrong logins, user must prove she’s a human by filling a CAPTCHA form
+ XPath Injection: heavily relying on XML, the registry is protected from injections by a sanitization mechanism.

### Mobile Clients
The mobile applications (iOS and Android) store the passwords using the standard “KeyChain” mechanism of each platform. They both support HTTPS connection (see below). The iOS application also provides a way to PIN-lock the whole application, this is in the roadmap for the Android version.

## How to report a vulnerability?
Write directly to security@pydio.com

## Permissions on pydio's files and/or folders
The setting permissions on Pydio’s files/folders is an easy but very important task. Correct permissions can reduce the risk of modifications from other entities than web server and system administrator. We highly recommend you to use the most restricted permissions for Pydio data.

Under the concept of linux

– All files/folders must be owned by root account

– Everyone have no access

– Web server (apache) has permission via group ownership

– Web server has read and execution on .php files and read/write on user data

Following this criteria, the solution below could be applied. When you install Pydio from deb or rpm, Pydio’s files/folders are located in several places such as ***/etc/pydio*** for configuration, ***/usr/share/pydio*** for .php files … and it could be slightly difficult because you will have to look for each file separately . In this example, we obviously can see that Pydio has group **www** and has no permission for everyone while it is owned by root. Note that data, configuration files and logs may be changed and permission must be r/w for group.

Example:

      # www is for httpd group name
    chown -R root:www PYDIO_INSTALL_DIR/
    cd PYDIO_INSTALL_DIR/
    find ./ -type d -exec chmod u=rwx,g=rx,o= '{}' \;
    find ./ -type f -exec chmod u=rw,g=r,o= '{}' \;

    #Mod data dir for config changes and logs
    find data -type d -exec chmod ug=rwx,o= '{}' \;
    find data -type f -exec chmod ug=rw,o= '{}' \;

    # fix permission on .htaccess to read-only for apache
    find /var/lib/pydio -name .htaccess -exec chmod 640 '{}' \;
    find /var/cache/pydio -name .htaccess -exec chmod 640 '{}' \;
    find /usr/share/pydio -name .htaccess -exec chmod 640 '{}' \;
    
When Pydio is installed from the archive on your webserver you just have to :    

```
    chown -R root:www-data /var/www/PYDIO_INSTALL_DIR/
    cd /var/www/PYDIO_INSTALL_DIR/
    find ./ -type d -exec chmod u=rwx,g=rx,o= '{}' \;
    find ./ -type f -exec chmod u=rw,g=r,o= '{}' \;
    
    #Mod data dir for config changes and logs
    find data -type d -exec chmod ug=rwx,o= '{}' \;
    find data -type f -exec chmod ug=rw,o= '{}' \;
    
    # fix permission on .htaccess to read-only for apache
    find /var/www/PYDIO_INSTALL_DIR -name .htaccess -exec chmod 640 '{}' \;
```

## Ensuring your security setup
As described in the security model, Pydio comes fully secured, and you should not have to worry about the major vulnerabilities brought by web applications. Still, there are a couple of tests you should make to be sure that your underlying configuration is not introducing problems.

## Checking web access to sensitive data
**_Note: this chapter is valid for Zip/Tar packaging only. In the Linux Packages (Deb & Rpm) the sensitive folders are spread elsewhere in the system, and by default never accessible via http._**

### Allowed / Forbidden folders
The most common mis-configuration of your web server that can lead to an information leakage lies in the “htaccess” (for Apache) or equivalents permission files configuration. When you deploy Pydio using the Zip or TarGz packages, the application is structured as follow. The folders marked with “red” must NOT be accessible through the web. This means for example that if you can read the content of http://yourserver/your_web_access/conf/, without having a “Forbidden” error, you HAVE a problem.

+ web_access/
    - `conf/`
    - core/
    - plugins/ 
    - data/
        * `cache/`
        * `plugins/`
        * `….`
        * public/
    - `+various php scripts`

    The data/ folder is a bit specific: it must be accessible only to let the user access *data/public/* folder. If you want to change this, you can change the **Application Core > Sharing**, using for example another folder, and declaring an alias in your web server. That way, you can safely disable all access for data/

### Apache: .htaccess
The most basic way of preventing access is (in Apache) to use a .htaccess file containing the `Deny From All` instruction. This is actually packaged in the software by default, and will work “as it's” in 95% of cases. But sometimes Apache is not configured to handle this : make sure that the VirtualHost or Directory configured for your website has the `AllowOverride All` instruction, otherwise .htaccess files will not be taken into account.

### IIS: web.config
The .htaccess are not understood by IIS. You have to use the equivalent using Request Filtering to prevent access to the described folders.

## Use HTTPS!
As recommended in the first page warning when you install the software, the basic security measure when you “http-ize” your information is to always use an SSL-encrypted connection! Buying a certificate for your domain is not very hard, and will make sure that all communications between the web application or the mobile clients and the server are encrypted. This is a norm that should be respected everywhere, for example if you're connecting to a public Wi-Fi, it is REALLY EASY to intercept sensitive information, such as users credentials or documents contents when you're downloading them.
