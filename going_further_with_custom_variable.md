### Introduction
Documentation detailing how [administrators can use custom vars in repository paths](https://pyd.io/defining-custom-variables-in-workspaces/) briefly describes what is required to pull your custom variables from an external data source and use them to dynamically assign repository paths to any user that logs on.

This is an extremely useful function as it eliminates having to create a new repository for each user when the users all require separate repositories that are linked to an existing directory structure external to the application..

One of the limitations of the default implementation is that when using the built in serial authentication the only variable that is exposed is the user id. If you wanted to map the users folders to an existing directory structure you have to do one of the following:

+ Rename all their root folders at the fiile system level.
+ Give the user a login ID that was identical to their existing root folder name.
+ Fetch the data from an external database. (If you were using the built in serial authentication you would be limited to naming the user exactly as named in the external data source so that the query could filter out your user.)
The following How -To will demonstrate how to add variables to the users profile within the application and then expose them for use either directly or as a reference to enable the linking to an external data source without the limitation to the users login ID outline above.

### Create your own plugin
The first step is to enable the action.skeleton plugin, or **better  create a duplicate** as described in the included plugin documentation. This will be used as the container class for our new filter function.

I called my plugin php class “pluginToolbar”.

### Handling the CUSTOM_DATA server-side
Next we need to open up root/conf/bootstrap_plugins.php and save a copy as a back up in case something goes wrong and you want reinstate the original configuration.

Now in the active bootstrap_plugins.php we can add variables to the CUSTOM_DATA array as shown below. These will show up on the users ‘personal data’ tab.

    $PLUGINS = array(
    	"CONF_DRIVER" => array(
	    	"NAME"		=> "serial",
	    	"OPTIONS"	=> array(
	    		"REPOSITORIES_FILEPATH"	=> "AJXP_DATA_PATH/plugins/conf.serial/repo.ser",
	    		"ROLES_FILEPATH"		=> "AJXP_DATA_PATH/plugins/auth.serial/roles.ser",
	    		"USERS_DIRPATH"			=> "AJXP_DATA_PATH/plugins/auth.serial",
	    		"CUSTOM_DATA" => array(
	    			"Fname" => "First Name", 
		    		"Lname" => "Last Name",
		    		"Company" => "Company Name",
		    		"Department" => "Department",
		    		"telephone" => "Tel:",
		    		"Email" => "Email",
		    		"Country" => "Country"
		    		)
	    		)
    	),

The next thing to do is add a filter function to the class.pluginToolbar.php file. This function will retrieve the custom variable from the user (in this case the “Company” variable) and assign it to a global variable similar AJXP_USER that can be used in repository paths or included in the application header / footer via the skeleton plugin. Here we created a new global variable called USERS_COMPANY_NAME and assigned the value stored against the user.

    public static function filterVars(&$value){
    	if(AuthService::getLoggedUser() != null){
	    	$custom = AuthService::getLoggedUser()->getPref("CUSTOM_PARAMS");
	    	$customCompany = $custom["Company"];
    		$value = str_replace("USER_COMPANY_NAME", $customCompany, $value);
    	}else{
	    	$value = str_replace("USER_COMPANY_NAME", "", $value);
	    }
    }

The next thing required to get it working is the so called “hook”. An example is provided at the top of the bootstrap_plugins.php file. We need to modify it as shown below. (Note we have also registered the xml.filter. This is only required if you want to use the newly exposed variables within the manifest.xml file of the skeleton plugin.

    /********************************************
     * CUSTOM VARIABLES HOOK
     ********************************************/
    /**
     * This is a sample "hard" hook, directly included. See directly the PluginSkeleton class
     * for more explanation.
     */
    require_once AJXP_INSTALL_PATH."/plugins/action.toolbar/class.pluginToolbar.php"; AJXP_Controller::registerIncludeHook("vars.filter", array("pluginToolbar", "filterVars")); AJXP_Controller::registerIncludeHook("xml.filter", array("pluginToolbar", "filterVars"));
    /*********************************************************/

Once you have made these changes you need to clear the application cache. This is done by deleting the root/data/cache/plugins_requires.ser and root/data/cache/plugins_cache.ser files. I like to restart the webserver and clear the cache in my browser at the same time.

Now you can create a repository with a path that includes the USERS_COMPANY_NAME variable.

### Using your new variables to customise the interface.
Once you have created and exposed your variables you can use them to customise the user interface. By modifying the manifest.xml file in the skeleton plugin as shown below your users will have their company name displayed on the right hand side of the main toolbar and your company name will be shown in the bttom right hand corner of the interface. The DIV code can be modified as required to get your logo and/or any other information you want to include onto the interface.

We are going to split the components into 3 separate template elements for clarity and add some javascript to refresh the interface when a user logs on or off.

    <client_configs uuidAttr="name">
    	<template name="head" element="ajxp_desktop" position="top"><![CDATA[
    		<script type="text/javascript">
    	document.observe("ajaxplorer:user_logged", function(){
	    	var reg = ajaxplorer.getXmlRegistry();
	    	var tplNode = XPathSelectSingleNode(reg, 'client_configs/template[@name="dynamic_head"]');
	    	$("dynamic_header_div").remove();
	    	$("ajxp_desktop").insert({top:tplNode.firstChild.nodeValue});
           });
			</script>
		]]></template>
		<template name="dynamic_head" element="ajxp_desktop" position="top"><![CDATA[
		<div id="dynamic_header_div" align="right" style="position:absolute;right:16px;top:7px;font-family:Arial Narrow;font-size:30px;padding:3px;">USER_COMPANY_NAME</div>
		]]></template>
	</client_configs>

### Using the “skeleton” plugin options
The custom Footer and Toolbar button target can be set from within the GUI using the plugin settings. (NB: I found that the button taget URL set in the GUI did not work as expected and so the button target can also be set directly in the manifest.xml file – as well as disabling the confirmation pop up.

 	var confs = ajaxplorer.getPluginConfigs("action[@name='toolbar']");
 	 var target = "http://yourbutton.target.com" window.open(target, "my_popup");
 	 
NOTE: The filterVars function should have some checking / verification added (array_key_exists?) to make it error proof. If the code is added without a correctly named CUSTOM_DATA array element defined an error will likely occur.