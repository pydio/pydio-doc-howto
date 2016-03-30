## Activate scheduler
The task scheduler is a component that was developed to centralized all the specific “maintenance” tasks, without having to add multiple lines in the external CRON-like system. The idea is to program a CRON to trigger the scheduler on a regular basis, and let the scheduler handle internal tasks.

You must first activate the plugin action.scheduler, that you will find under **Global Configs > Extensions > Action > Scheduler**. To activate, open the plugin editor and make sure it is set to “Enabled: Yes”. If you reload the GUI, you should now see the Scheduler node appear under Logs & Other Data

[:image-popup:plugins/using_the_tasks_sheduler/screenshot-2013-07-19-at-14-15-23.png]
 

## Programming tasks
Once the plugin is active, you will also see a new button in the toolbar (when switched on the “Settings” workspace). You can select “New Task” to define a task that will be triggered by the scheduler.

A task is defined by the following parameters:

+ Execution Context
    - **Label**: a readable label for your task
    - **Schedule**: a CRON-Like expression following the format **_minutes hours days dayWeeks monthes_**
    - **User(s)**: a comma-separated list of users. Wildcard “*” without quotes will mean all users
    - **Repository**: The repository on wich to apply the task
+ Parameters
    - **Action**: an action provided by one of the application plugin
    - **Param Name / Value pairs**: additional parameters required by the chosen action.

## Triggering the scheduler
First you must be familiar with the PHP CLI way of activating the framework. If not, please read https://pyd.io/administrator/enriching-your-users-experience/command-line-version-of-the-server/ from the admin guide.

The action to trigger the scheduler is <code>scheduler_runAll</code>. Thus, if you want to for example, trigger using a CRONTAB this action every 5 minutes, you will add the following entry in the CRONTAB:

`*/5 * * * * php /var/www/pydio/cmd.php -r=ajxp_conf -u=admin -p=YOUR_PASSWORD_HERE -a=scheduler_runAll >> /var/www/pydio/data/cache/cmd_outputs/cron_commands.log`

Using the “Cron Expression” button in the Scheduler menu, you can simply copy and paste the correct expression, replacing your password with the correct value.