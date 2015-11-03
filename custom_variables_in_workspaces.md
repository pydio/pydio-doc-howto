It’s a common need, especially when you handle a great number of users, to try to dynamically create the repositories for all users, instead of creating manually each repository for each users. The “Custom Variables”, in conjunction with specific PHP hooks, can be a smart way to implement such a feature.

### Usecase 1 : “I have 5000 users, and I just want that each user have access to their own folder (/home/username), and not the other users folders”.
The trick here is that you won’t create 5000 repositories, even dynamically, but instead you’ll create one repository “My Home” (for example), that will dynamically adapt its path to the currently logged user. To do this, you’ll just have to define the “Path” of the repository with the specific AJXP_USER keyword. This keyword will be replace at runtime by the currently logged user. Along with this usecase is often the fact that you don’t use the basic auth.serial plugin, but something more scalable like auth.ldap, or auth.sql. In that case, you won’t even have to manage users acesses via AjaXplorer : just use the “Default Rights” option of the “My Home” repository, set it to “rw”, and here you are! Any new user logging in will have it’s own personal space!

[:image-popup:workspaces/custom_varibales_in_workspaces/screenshot-2013-06-03-at-14-39-12.png]

### Usecase 2 : “I have 5000 users, each of them belonging to one of 40 department, and I want them to access to a given /home/department/ folder.”
**_Note : this feature is only available for version > 3.2.2_**

That’s were the custom variable will appear. The trick here will be two-step : name a specific variable, similar to AJXP_USER used in the previous case, that will be your department name, like CUSTOM_DEPT, and then insert a hook in the code, that will be called by AjaXplorer at runtime to replace this CUSTOM_DEPT by the right value.

We assume that at one point, you have the skill to get the correspondance table between users and departments in one form or another : interrogating a database, loading an XML file, or a simple CSV file, whatever. You must be able to create your own PHP routine like “**get_my_user_dept()**“, that will take a user and give back the dept name. Once you have that piece of code, you’re nearly done! All you’ll have to do will be to create a “filter” with a proper function signature, then tell AjaXplorer to call this filter each time it must grab a repository option (in fact more generally option). The function signature must simply take a $value argument by reference, and modify it as necessary. So your function will probably look like :

	function myFilter(&$value){
    	 $loggedUser = AuthService::getLoggedUser();
    	 $department = get_my_user_dept($loggedUser->getId());
    	 $value = str_replace("CUSTOM_DEPT", $value);
	}

Then in the config file (conf/bootstrap_context.php), you will register this function as a “vars.filter” hook :

	require_once  "the/file/containing/your/own/code";
	AJXP_Controller::registerIncludeHook("vars.filter", "myFilter");
	
And you’re done!

 

And to go further on this, see How-to: https://pyd.io/going-further-with-custom-variables/