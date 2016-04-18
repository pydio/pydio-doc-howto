### Workspace Templates
[:image:image/workspaces/delegating_workspace_creation/display_template.png]

There are two ways to let some non-administrator users create their own workspaces. In both cases, you first have to create a “Template”, which is a workspace where you prefill most of the necessary parameters, and only a couple of parameters are let to the user.

When you create such a template, you have the options as when creating a Workspace, plus an additional “Template Options” section, where you can manually choose whether group administrators and / or simple users will be able to use this template to create a workspace. That way, templates can also be used “internally” for the admin convenience, when you find yourself creating the workspaces with the same parameters over and over.

#### _Template specific variable_

Templates parameters can use exactly the same parameters (and custom variables) as would Workspace do, plus an interesting **AJXP_ALLOW_SUB_PATH** keyword, that can be appended to an absolute path on the server. For example, you can use the following Path value for a standard access driver :

	/var/lib/tpldata/AJXP_ALLOW_SUB_PATH

In that case, if a user creates a workspace from this template, any value set to the “Path” parameter will be actually prepended by the /var/lib/tpldata/ path.

## Using groups & group administrors

Creating groups, you can then assign the “Administrator” Profile to a user of a group. This user will be able to access to a lightweight version of the Settings panel. She will only see the users, workspaces and roles of her group. And she will only be able to create a workspace based on an authorized template, not from scratch. Here is below a screenshot of the group admin Settings panel.

[:image-popup:workspaces/delegating_workspace_creation/group_admin.png]

 

### Letting users create their own workspaces
Generally, delegating some administrative task to a group administrator should be sufficient, but in some case, you’ll even want every users to be able to create a workspace.

Take for example a “Drobpox” based workspace : as admin, you will want to describe the API Key used to access the Dropbox API, and share it with all accounts, but then you will want each user to manually enter her own login/password to access dropbox. They will do that by using a predefined Dropbox template. Same case for accessing a mailbox for example.

First, make sure the templates you have defined have the “Let Users create workspace” option set to “Yes”. And also, in the global “Configurations Management”, enable the “Let users create repositories” options. Once all this is enabled, users should see a “Create workspace” action appear at the bottom of their workspace list. From there, they will be able to choose a template, and fill the remaining options not included in the template.