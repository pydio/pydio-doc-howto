## Objective
In many usecases, Pydio is used to create an interface between the inside world and the outside world of an organization. To prevent information leakage and loss of control, many administrators will choose to automatically “expire” the documents: docs older than a given period are simply deleted. That way, users cannot forget and let confidential information stay in a semi-public area for too long.

## Set up Purge parameter
The PURGE parameter is a number of days after which docs are automatically removed, if they have not been modified. It’s a workspace parameter, and can be refined in many ways: on a per-workspace basis, and/or per-user and per-group basis.

Let’s say you want users of group1 to be able to keep documents for 30 days in their “Personal Files” workspace, and users of group2 to keep them only for 10 days in the same workspace. First piece is to set up the standard purge parameter in the workspace editor:

[:image-popup:plugins/set_up_an_automatic_purge/screenshot-2013-07-19-at-14-36-44.png]

We have set a default value of 10 days, thus we want to refine the value for users of group 1. Edit the “Group 1” using group editor, and go the “Parameters” tab. Search for plugin **access.fs** then parameter **PURGE_AFTER**, select the “**Personal Files**” workspace, and add it to the list of the customized parameters of this group. Set up a value and save the group.

[:image-popup:plugins/set_up_an_automatic_purge/screenshot-2013-07-19-at-14-39-36.png]

Now we’re done with the settings. Time to actually have the purge applied!

## Applying purge
### Direct call from CLI
You must be familiar with the PHP CLI way of activating the framework. If not, please read https://pyd.io/administrator/enriching-your-users-experience/command-line-version-of-the-server/ from the admin guide.

The action to trigger the purge is simply `<code>purge</code>`. When calling an action from the command line, you have to specify a target workspace (using it’s alias), and a user. If we want to trigger the purge for user1 , we will call something like:

    php /var/www/ajaxplorer/cmd.php -r=personal-files -u=user1 -p=u1password -a=purge

As you can see, the problem of such system is that you have to actually “log in as user1” to perform the purge. Using the task scheduler is a handy way to program the task execution to recurse on all users.

### Programming a scheduler task
The task scheduler is a component that was developed to centralized all the specific “maintenance” tasks, without having to add multiple lines in the external CRON-like system. The idea is to program a CRON to trigger the scheduler on a regular basis, and let the scheduler handle internal tasks. Please read the [“Using the task scheduler”](https://pyd.io/using-tasks-scheduler/) How-to if you are not already familiar with it.

What we want to achieve is program a task that will purge the “Personal Files” of all users. For that, we can use a wildcard in the scheduler task, “*” meaning all users. Add a new task with the following parameters:

+ Execution Context
    - **Label**: “Purge Personal Files”
    - **Schedule**: * 3 * * *
    - **User(s)**: *
    - **Repository**: personal-files
+ Parameters
    - **Action**: purge

This will trigger the purge every day at 3 AM

**WARNING**: You must have set up the necessary CRON to actually trigger the scheduler on a regular basis! See the How-to if necessary.