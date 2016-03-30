A dropbox account can be used as a storage backend for Pydio. This tutorial goes through the necessary steps to create the link.

## Creating Pydio application in the dropbox account
In order to let Pydio communicate with your dropbox account, you will first have to declare Pydio as an authorized application on the dropbox side. This is done by creating an “application” that will access to the “Datastore API”.

Go to https://www.dropbox.com/developers/apps and use the **Create app** button. Select “**Dropbox API app**” and select the appropriate options depending on your needs. See the screenshot below.

[:image-popup:workspaces/accessing_an_existing_dropbox/screenshot-2013-07-19-at-14-55-37.png]

Once it’s created, you will be able to get the **App Key** and **App Secret** of the application.

[:image-popup:workspaces/accessing_an_existing_dropbox/screenshot-2013-07-19-at-14-57-11.png]

## Create a workspace using Dropbox Driver
Now that you have all the necessary information, you can create a workspace using the Dropbox driver. You will require one specific package from the PEAR library. Make sure to have PEAR installed on your system, and run the following:

    pear install channel://pear.php.net/HTTP_OAuth-0.2.3

Create the workspace, by using the information gathered above:

Your email / password for dropbox account, and the application Key and Secret that you just generated.

Now you have to actually authorize the application, which is done through an OAuth negotiation. Switch to the Dropbox workspace, and you will see a red error message asking you to click on a link. Click on the link before it disappears!

[:image-popup:workspaces/accessing_an_existing_dropbox/screenshot-2013-07-19-at-15-06-11.png]

This will store an authorization token, and you won’t have to redo this, unless you delete the content of data/plugins/access.dropbox.