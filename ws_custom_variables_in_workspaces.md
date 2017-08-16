*Note: updated for Pydio 8*

It’s a common need, especially when you handle a great number of users, to try to dynamically create the repositories for all users, instead of creating manually each repository for each users. The “Custom Variables”, in conjunction with specific PHP hooks, can be a smart way to implement such a feature.

### Usecase : “I want that each user have access to their own folder (/home/username)”.
The trick here is that you won’t create 5000 repositories, even dynamically, but instead you’ll create one repository “My Home” (for example), that will dynamically adapt its path to the currently logged user. To do this, you’ll just have to define the “Path” of the repository with the specific AJXP_USER keyword. 

This keyword will be replace at runtime by the currently logged user. Also to make sure all users have now access to this personal workspace, you can edit the root role and grant a R/W access to it. 

[:image-popup:workspaces/custom_varibales_in_workspaces/custom_variable_in_workspaces.png]	[:image-popup:workspaces/custom_varibales_in_workspaces/custom_variable_in_workspaces2.png]

### Usecase : “Users storage is 'sharded' inside various folders, based on their login and my organization rules.”

#### Custom Variable

That’s were the custom variable will appear. The trick here will be two-step : name a specific variable, similar to AJXP_USER used in the previous case, that will be your department name, like CUSTOM_USER_PATH, and then insert a hook in the code, that will be called by Pydio at runtime to replace this CUSTOM_USER_PATH by the right value. First we create the Workspace with the custom variable, don't forget to set good right to all users.

[:image-popup:workspaces/custom_varibales_in_workspaces/custom_variable_in_workspaces3.png]	[:image-popup:workspaces/custom_varibales_in_workspaces/custom_variable_in_workspaces4.png]

#### Filter function

We assume that at one point, you have the skill to get the correspondance table between users and departments in one form or another : interrogating a database, loading an XML file, or a simple CSV file, whatever. You must be able to create your own PHP routine like “**CustomVarFilter()**“, that can give the correct folder path based on e.g. the user ID.   

The filter function signature takes a $value argument by reference, and modify it as necessary, and a ContextInterface $ctx that contains information about the current request context (namely the logged user object and the current workspace object.). 

Your function could look like the following. This one creates a shared tree of folders based on the two first letters (like storing admin in /a/d/admin/), with a first level of folder hardcoded (so admin would in fact be under /fs1/a/d/admin/).

    define("CUSTOM_DATA_BASE", AJXP_DATA_PATH . "/files/custom-var-tests");
    
    function CustomVarFilter(&$value, \Pydio\Core\Model\ContextInterface $ctx){
    
        if(!is_string($value) || "CUSTOM_USER_PATH" != $value){
            // NOT THE EXPECTED VARIABLE
            return;
        }
    
        $userObject = $ctx->getUser();
        if($userObject == null){
            // NO USER LOGGED
            return;
        }
            
        $ref =  array(
            "01279a"	=> "fs1",
            "bc"		=> "fs2",
            "defg"		=> "fs3",
            "hijk"		=> "fs4",
            "lmn"		=> "fs5",
            "opqr"		=> "fs6",
            "stuvwxyz"	=> "fs7"
        );
        
        $userId = $userObject->getId();
        $firstChar = $userId[0];
        $secondChar = $userId[1];
        
        $server = "other";
        foreach($ref as $key => $address){
            if(strpos($key, $firstChar) !== false){
                $server = $address;
            }
        }
        
        $value = strtolower(CUSTOM_DATA_BASE."/".$server."/".$firstChar."/".$secondChar."/".$userId);
    
    }


Basically, what it does is retrieving the current user from the context, parsing it's first letters and building the full path from that information. Then it replaces the passed value (CUSTOM_USER_PATH) by the new value (e.g. /var/www/pydio/files/custom-var-tests/fs1/a/d/admin).

#### Registering the filter function

Finally in the config file (conf/bootstrap_context.php), you will register this function as a “vars.filter” hook :

	require_once  "the/file/containing/your/own/code";
	AJXP_Controller::registerIncludeHook("vars.filter", "CustomVarFilter");
	