This section will describe how to use the auth.remote plugin to create a bridge with an existing CMS. At first we’ll assume that the cms has it’s own authentification mechanism, and one just want to “inform” Pydio that a user is logged indeed. Then, we’ll see how to create an external login page and still  use Pydio’s own mechanism to check the users credentials (for example on a simple website that does not have users management at all).
One of the best way to see the auth.remote plugin work in real life is to download and inspect the existing bridges with external cms : wordpress, drupal, joomla and cmsms. They all do exactly what is described here.

### Auth.remote plugin basics
**SSO Mechanism**

Auth.remote plugin is designed to create bridges between application. We won’t go here through the plugins basics, please go to the Plugins chapter of the Administrator Guide if you don’t know what we are talking about here. The master directory is considered to be the third party CMS.

When accessing Pydio, the plugin configurations will allow you to either automatically redirect non-logged users to the CMS login page (unique login page), or to log in Pydio, which would log the user in both the CMS and Pydio (bi directionnal login). Logout is recommanded to be performed on the CMS side, by using a “Logout Redirect” action.

**Communication**

The base of this plugin will be to call from your external cms a little piece of Pydio code to trigger Pydio authentication events (the process is detailed below), and thus registering a logged user in both the CMS Session and Pydio Session. When calling this code, we want to make sure that the code is really called by the CMS, not by another code on the same server, so there is a basic security implemented by a “secret” key that is shared by both the CMS and by Pydio.

**Users auto-creation**

One of the interest of using auth.remote with another CMS can be to avoid duplicating users management on both parts of the system (pydio & cms) : for this, there is an **autoCreate** option that will trigger automatic creation of a user account in Pydio when logged in by auth.remote : that way, users creation/deletion is managed only on the CMS side, and if the CMS tells a user is valid, Pydio will create automatically its account, even if it’s not already registered on the Pydio side. This option is passed programmatically when calling the glueCode.

**_Note_**: autoCreate would probablye be useless if not used in conjunction with the “DEFAULT_RIGHTS” option of the repositories. Setting a non-empty value for this option in a given repository “Repo” (value can be r or rw as read or read/write), and setting auth.remote auto create option to true, any user identified by the external CMS will be automatically created in Pydio AND have directly access to “Repo” with read or read/write access.

**Auth.remote configuration**

Now we can have a closer look at the auth.remote parameters

[:image-popup:authentication/authentication_w_your_cms/screenshot-2013-05-13-at-11-45-57.png]

You can choose the CMS type amongst the most well-known types (Drupal, WordPress, Joomla) or Custom, where you will enter all parameters manually. Basically, the main URL and login URL will point to the CMS root, and to a CMS page containing a login submission form.

Auth Form ID is a specific parameter that you may have to check: the login page of your CMS MUST CONTAIN an html Form element with the id indicated here. The default value used here should worked by default, but you may verify that typically your CMS custom theme did not change this value. It is necessary if you want to have a chance the Pydio can actually authenticate users on the Pydio-side. Which is necessary to have the Mobile Clients working.

### Communicating between the two apps
**The glueCode API**

Let’s digg how the glueCode works : its role is to be inserted inside an already existing system (cms or not, but assuming they are written in PHP of course) and be able to let this system “call” Pydio authentication functions (login, logout, create/delete users, etc).

Necessarily, if you want to call a function of such a framework as Pydio, you can’t just “require()” a file and call the function : you have to initialize the framework first, by requiring all necessary dependencies, starting the right PHP session, etc, etc. This can be very tedious, it’s a lot of work to learn the given framework, what classes are needed etc. That’s why all this code is in fact performed directly by the glueCode! It can be seen as a parenthesis in your code, injecting and initializing all the Pydio framework, executing one instruction, and going out. The glueCode is located inside auth.remote plugin, in the file glueCode.php (good choice no?). So all you’ll have to do is to know the absolute (ore relative) path on your server to this file, and you’ll be ok!

How can we pass parameters to this piece of code, like the name of a user? We use the PHP global scope to communicate between the two applications : before including the glueCode, you just have to declare a global array() named **$AJXP_GLUE_GLOBALS** that will contain all necessary arguments : the target function and its arguments, but also the secret key, the auth.remote plugin configuration, etc.

**Calling glueCode sample**

So as an example, let’s take the simplest action logout : when a user disconnects from the CMS, we want to be sure that he will also be disconnected from ajaXplorer, so we add the following lines in the CMS. **Please note the AJXP_EXEC constant** : it’s important, otherwise you’ll have an “Access not allowed” error when including glueCode.php to your code.

    $glueCode = "path/to/pydio/plugins/auth.remote/glueCode.php";
	$secret = "secret_key_passed_to_auth.remote_by_configuration";
	define('AJXP_EXEC', true);

	global $AJXP_GLUE_GLOBALS;
	$AJXP_GLUE_GLOBALS = array();
	$AJXP_GLUE_GLOBALS["secret"] = $secret;
	$AJXP_GLUE_GLOBALS["plugInAction"] = "logout";
   	include($glueCode);

### Auth.remote actions reference
We’ll list here the available actions made available by the glueCode.php file. Of course, it’s up to you to extend and write your own glueCode.php if you need more interactions with Pydio. As described in the previous section, the array $AJXP_GLUE_GLOBALS defined in the PHP global scope is used to pass both the action name (key “plugInAction”) and the action paramaters (other keys described). The output of each action is set by the glueCode as a boolean inside AJXP_GLUE_GLOBALS[“result”].

+ **login**
    - This will log a user in Pydio. **At the moment, the password check is bypassed**, assuming the CMS is responsible for the credentials. See the next section if you still want to rely on Pydio user/password checks. If autoCreate is true, the user will be created anyway, even if it’s not already existing on the Pydio side. If the admin right is passed, the user is automatically assigned “admin” right. A SessionSwitcher is used to temporary close the current php session, open the Pydio session and log the user.
    - **_Parameters_**
        * “plugInAction” : “login”
        * “login” : array(“name” => user_name, “password” => user_password, [optional] “right” => “admin”)
        * “autoCreate” : boolean true or false
+ **logout**
    - Destroy the Pydio session (disconnecting any logged user)
    - **_Parameters_**
        * “plugInAction” : “logout”
+ **addUser**
    - Adds a user inside Pydio. Can specify whether the new user is admin or not.
    - **_Parameters_**
        * “plugInAction” : “addUser”
        * “user” : array(“name” => username, “password” => userpassword”, [optional] “right” => “admin”)
+ **delUser**
    - Deletes an Pydio user
    - **_Parameters_**
        * “plugInAction” : “delUser”
        * “userName”  : the user name.
+ **updateUser**
    - Updates an existing user, can be used to update both password and admin right.
    - **_Parameters_**
        * “plugInAction” : “addUser”
        * “user” : array(“name” => username, “password” => userpassword”, [optional] “right” => “admin”)
If not already done, please download the pydio “External Bridges” distribution and have a  look for example at the wordrpress WpAjxp.php class : you’ll now be able to understand what’s happening : beside all the pure WordPress work necessary to hook a given function inside the available wordpress extensions points, the functions are all really simple and just make use of this API to trigger the necessary actions inside Pydio.

So if either your CMS implement extension points, or you can easily determine where to place the calls in your code (just after successfull login, just after successfull logout, etc, etc), include the glueCode.php having set the right arguments and it should work.

### Create your own login page but still using Pydio users management
Now we will describe how to create a custom login page, but that will still rely on Pydio users/password checks. You don’t use a specific CMS and still want to use Pydio full-featured auth system. We’ll have to hack some part of the glueCode.php, so **you must have some basic PHP knowledge** to understand what you are doing when editing the files!

**Auth.remote necessary changes**

The main problem is that in the standard glueCode.php, when authenticating a password with the credentials passed by an external CMS, we in fact assume that the user has legitimity, and bypass the password check. So we’ll have to change this to make sure that the password is indeed checked. This can be done by changing the AuthService::logUser call inside plugins/auth.remote/glueCode.php, line 99 :

Replace : $result = AuthService::logUser($login[“name”], $login[“password”], true); // the true here specify “bypass password check”.

By : $result = AuthService::logUser($login[“name”], $login[“password”], false, false, -1); // Note the last arguments they are important

**_[NOTE : i’ll probably add a parameter to the standard glueCode.php to avoid this hack, but as for now it’s necessary (release 3.1.1)]_**

Also since there is a kind of challenge/response using a seed mechanism to encode the password between Pydio GUI and Pydio server, and since you won’t use this feature, we’ll first have to **switch the TRANSMIT_CLEAR_PASS option to TRUE.**

**Creating a form and submitting it to a php script**

Now, assuming you have configured your Pydio installation to use auth.remote, with TRANSMIT_CLEAR_PASS to TRUE, and the hack of glueCode at line 99, how would your login form look like? Well, like this !

    <?php
    // Here the PHP code for handling the form and the HTML code
    // for displaying it are in the same file "login.php"
    // but it's not necessary!

    if(isSet($_POST["login"]) && isSEt($_POST["password"])){

    	// Necessary to make "connection" with the glueCode
    	define("AJXP_EXEC", true);
    	$glueCode = "absolute/path/to/ajxp/plugins/auth.remote/glueCode.php";
	    $secret = "my_secret_key";

    	// Initialize the "parameters holder"
    	global $AJXP_GLUE_GLOBALS;
    	$AJXP_GLUE_GLOBALS = array();
    	$AJXP_GLUE_GLOBALS["secret"] = $secret;
    	$AJXP_GLUE_GLOBALS["plugInAction"] = "login";
    	$AJXP_GLUE_GLOBALS["autoCreate"] = false;

    	// NOTE THE md5() call on the password field.
     	$AJXP_GLUE_GLOBALS["login"] = array("name" => $_POST["login"], "password" => md5($_POST["password"]));

    	// NOW call glueCode!
       	include($glueCode);
    }
    ?>
    <html>
    	<body>
    		<form action="login.php" method="POST">
    			Login : <input name="login"><br>
    			Password : <input name="password" type="password"><br>
    			<input type="submit" value="SUBMIT">
    		</form>
    	</body>
    </html>
    
Well, that’s all for today folks!