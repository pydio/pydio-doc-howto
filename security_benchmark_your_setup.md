As described in the security model, Pydio comes fully secured, and you should not have to worry about the major vulnerabilities brought by webapps. Still, there are a couple of tests you should make to be sure that your underlying configuration is not introducing problems.

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
As recommanded in the first page warning when you install the software, the basic security mesure when you “http-ize” your information is to always use an SSL-encrypted connexion! Buying a certificate for your domain is not very hard, and will make sure that all communications between the webapp or the mobile clients and the server are encrypted. This is a norm that should be respected everywhere, for example if you're connecting to a public Wi-Fi, it is REALLY EASY to intercept sensitive informations, such as users credentials or documents contents when you're downloading them.
