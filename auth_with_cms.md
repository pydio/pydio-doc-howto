### AUTHENTICATION USING A CMS
In this guide we will see how to do a basic remote authentication with a CMS.
First and foremost if you're here it will be assumed that you have finished your Pydio installation and that you can log and use it.

### REMOTE AUTHENTICATION PLUGIN

Go to **Application Parameters > Available Plugins > Authentication Backend** and enable the plugin if it wasn't already done.
This plugin will create a bridge between Pydio and an application.

*If you want more details about the plugin you can go to **[this](https://pydio.com/fr/docs/references/plugins/auth/remote)** page*

Now you can install **[Pydio-Bridge](https://pydio.com/en/get-pydio/downloads/cms-bridges)**, first download it then go to your cms's extensions/addons/(whatever it is called) settings and install it.

> The supported versions are Wordpress **all versions**, Joomla **1.5 to 3.0** and Drupal **6.X, 7.X**

After that you just need it to be enabled on your cms and set with the following parameters :
+ **Pydio's installation path** : you must give an absolute path such as for example `/var/www/pydio` or wherever you have it on your webserver.
+ **Secret Key** : this secret key allows your Pydio and CMS to recognize each other.
+ **Create Users**: Usually this should be enabled, what it does is that if you're logging in with a user that is not in Pydio's database it will automatically create an entry for it.


> These are the most common settings but if your configuration is different from the ones that i listed, i highly recommand you to go look at your CMS'S documentation.

Now on Pydio's side you have to go to **Application Parameters > Authentication > Master Driver > Main Instance > and choose remote authentication**

I will give you a basic configuration for those cms's.

#### Joomla!
[:image-popup:/authentication/auth_remote_joomla_CMS.png]

+ **Joomla! URL** : link to your joomla `http://host/path` or `/path`.
+ **Home node** : Joomla's main page where the login form is located.
+ **Auth Form ID** : the default value for Joomla! is `login-form`.
+ **Local Prefix** : if you create a user using this prefix he will be stored in the local filesystem.
+ **Secret Key** : this is the key that you get on your Joomla's extension Pydio-Bridge. 
+ **Users** : this is the users list you should let the default value if you're not used to it.


#### Wordpress
[:image-popup:/authentication/auth_remote_wordpress_CMS.png]

+ **Wordpress URL** : link to your wordpress installation `http://host/path`or`/path`.
+ **Login URL** : if you're not in slave mode let it by default else it will redirect you to a given URL.
+ **Exit Action** : you can choose an action what will be performed when a user quits Pydio
you have the choice between `back to main page`or`logout`.
+ **Local Prefix** : if you create a user using this prefix he will be stored in the local filesystem.
+ **Secret Key** : this is the key that you get on your Wordpress' extension Pydio-Bridge.
+ **Users** : this is the users list, you should let the default value if you're not used to it. 

#### Drupal
[:image-popup:/authentication/auth_remote_drupal_CMS.png]

+ **Drupal URL** : link to your drupal installation `http://host/path`or`/path`.
+ **Login URL** : Drupal's main login page.
+ **Auth Form ID** : by default the value is `user-login-form`.
+ **Exit Action** : you can choose an action what will be performed when a user quits Pydio either `back to main page`or`logout`.
+ **Local Prefix** : if you create a user using this prefix he will be stored in the local filesystem.
+ **Secret Key** : this is the key that you get on your Drupal's extension Pydio-Bridge.
+ **Users** : this is the users list, you should not change the default value if you're not used to it. 

#### Custom
[:image-popup:/authentication/auth_remote_custom_CMS.png]

+ **Login URL** : to your custom login page.
+ **Logout URL** : Redirects you to this URL when you're logging out.
+ **Custom Auth Function** : PATH or URL to your custom authentifcation function.
+ **Local Prefix** : if you create a user using this prefix he will be stored in the local filesystem.
+ **Secret Key** : this is the key that you get on your Drupal's extension Pydio-Bridge.
+ **Users** : this is the users list, you should not change the default value if you're not used to it. 

>With the Pydio-Bridge file there is a basic login form that you can use as a template and modify as you please.


