*Note: NOT updated for Pydio 7 & 8, only compatible with P6*

### Introduction
Documentation detailing how [administrators can use custom vars in repository paths](https://pydio.com/en/docs/kb/workspaces/custom-variables-workspaces) briefly describes what is required to pull your custom variables from an external data source and use them to dynamically assign repository paths to any user that logs on.

This is an extremely useful function as it eliminates having to create a new repository for each user when the users all require separate repositories that are linked to an existing directory structure external to the application..

One of the limitations of the default implementation is that when using the built in serial authentication the only variable that is exposed is the user id. If you wanted to map the users folders to an existing directory structure you have to do one of the following:

+ Rename all their root folders at the file system level.
+ Give the user a login ID that was identical to their existing root folder name.
+ Fetch the data from an external database. (If you were using the built in serial authentication you would be limited to naming the user exactly as named in the external data source so that the query could filter out your user.)
The following How -To will demonstrate how to add variables to the users profile within the application and then expose them for use either directly or as a reference to enable the linking to an external data source without the limitation to the users login ID outline above.

### Create your own plugin
The first step is to enable the action.skeleton plugin, or **better  create a duplicate** as described in the included plugin documentation. This will be used as the container class for our new filter function.

I called my plugin php class “pluginToolbar”.

###Create an input in your profile side
We can add more input in the profil side to add and get more info of the current user. We want his company name so we add a new param in the manifest of the plugin like this :

    <server_settings>
        <param name="user_company" type="string" scope="user" description="CONF_MESSAGE[Your Company name]" label="CONF_MESSAGE[Company]" expose="true" mandatory="true" default="" />
    </server_settings>

One of the most important thing is the "scope" parameter. When we set "user" it add the "user_company" parameter in the user profil we can get this in the profile side :

[:image-popup:workspaces/going_further_with_custom_variable/going_further_with_custom_variable.png]

Don't forget to change your plugin name and description !!
### Handling the CUSTOM_DATA server-side
We add a filter function to the class.pluginToolbar.php file. This function will retrieve the custom variable from the user (in this case the “Company” variable) and assign it to a global variable similar AJXP_USER that can be used in repository paths or included in the application header / footer via the skeleton plugin. Here we created a new global variable called USERS_COMPANY_NAME and assigned the value stored against the user.

    public static function filterVars(&$value)
    {
        if (is_string($value) && strpos($value, "USER_COMPANY_NAME") !== false) {
            if(AuthService::getLoggedUser() != null) {
                $user_id = AuthService::getLoggedUser()->getId();
                $userObject = ConfService::getConfStorageImpl()->createUserObject($user_id);
                $user_company = $userObject->mergedRole->filterParameterValue("action.toolbar", "user_company", AJXP_REPO_SCOPE_ALL, "");
                if (empty($user_company)) {
                    $user_company = "empty_company";
                }
                $value = str_replace("USER_COMPANY_NAME", $user_company, $value);
            } else {
                $value = str_replace("USER_COMPANY_NAME", "", $value);
            }
        }
    }

So first we check $value before the user because when we check the user, Pydio check some variable filter including this one, so to avoid any error, we check in this order. After that we get all info of the connected user and get the "user_company" variable from the plugin "action.toolbar" ( we see that later ). We do a final check if his company name isn't empty.

The next thing required to get it working is the so called “hook”. Then in the config file (conf/bootstrap_context.php), you will register this function as a “vars.filter” hook ( Note we have also registered the xml.filter. This is only required if you want to use the newly exposed variables within the manifest.xml file of the skeleton plugin ) :

    require_once AJXP_INSTALL_PATH . "/plugins/action.toolbar/class.pluginToolbar.php";
    AJXP_Controller::registerIncludeHook("vars.filter", array("pluginToolbar", "filterVars"));
    AJXP_Controller::registerIncludeHook("xml.filter", array("pluginToolbar", "filterVars"));

Once you have made these changes you need to clear the application cache. This is done by deleting the root/data/cache/plugins_requires.ser and root/data/cache/plugins_cache.ser files. I like to restart the webserver and clear the cache in my browser at the same time.

Now you can create a repository with a path that includes the USERS_COMPANY_NAME variable ( don't forget the read and write option !!) :

[:image-popup:workspaces/going_further_with_custom_variable/going_further_with_custom_variable2.png]

### Using your new variables to customise the interface.
Once you have created and exposed your variables you can use them to customise the user interface. By modifying the manifest.xml file in the skeleton plugin as shown below your users will have their company name displayed on the right hand side of the main toolbar and your company name will be shown in the bttom right hand corner of the interface. The DIV code can be modified as required to get your logo and/or any other information you want to include onto the interface.

We are going to split the components into 3 separate template elements for clarity and add some javascript to refresh the interface when a user logs on or off.

    <client_configs uuidAttr="name">
    	<template name="head" element="ajxp_desktop" position="top"><![CDATA[
    		<script type="text/javascript">
    	document.observe("ajaxplorer:user_logged", function(){
	    	var reg = pydio.getXmlRegistry();
	    	var tplNode = XPathSelectSingleNode(reg, 'client_configs/template[@name="dynamic_head"]');
	    	$("dynamic_header_div").remove();
	    	$("ajxp_desktop").insert({top:tplNode.firstChild.nodeValue});
           });
			</script>
		]]></template>
		<template name="dynamic_head" element="ajxp_desktop" position="top"><![CDATA[
		<div id="dynamic_header_div" align="right" style="position:absolute;right:150px;top:7px;font-family:Arial Narrow;font-size:30px;padding:3px;z-index:10000">USER_COMPANY_NAME</div>
		]]></template>
	</client_configs>

### Using the “skeleton” plugin options
The custom Footer and Toolbar button target can be set from within the GUI using the plugin settings. (NB: I found that the button taget URL set in the GUI did not work as expected and so the button target can also be set directly in the manifest.xml file – as well as disabling the confirmation pop up.

 	var confs = pydio.getPluginConfigs("action[@name='toolbar']");
 	 var target = "http://yourbutton.target.com" window.open(target, "my_popup");
 	 
NOTE: The filterVars function should have some checking / verification added (array_key_exists?) to make it error proof. If the code is added without a correctly named CUSTOM_DATA array element defined an error will likely occur.