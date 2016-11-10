A dropbox account can be used as a storage backend for Pydio. This tutorial goes through the necessary steps to create the link.

## Creating Pydio application in the dropbox account
In order to let Pydio communicate with your dropbox account, you will first have to declare Pydio as an authorized application on the dropbox side. This is done by creating an “application” that will access to the “Datastore API”.

Go to https://www.dropbox.com/developers/apps and use the **Create app** button. Select “**Dropbox API app**” and select the appropriate options depending on your needs. See the screenshot below.

[:image-popup:workspaces/accessing_an_existing_dropbox/screenshot-2013-07-19-at-14-55-37.png]

Once it’s created, you will be able to get the **App Key** and **App Secret** of the application.

[:image-popup:workspaces/accessing_an_existing_dropbox/screenshot-2013-07-19-at-14-57-11.png]

## Create a workspace using the Dropbox Driver [Pydio 7 and above]

_Note_: If you are using Pydio 6, see below. 

### Dropbox Apps and OAuth2

The Dropbox plugin uses the Dropbox API to exchange information with the Dropbox system, and Dropbox relies on the OAuth 2 to allow the pydio system to make calls to the API on your behalf.

Dropbox have written a guide to explain the principle, and it contains links and explanations onto how to setup a new App in your Dropbox account :

[https://www.dropbox.com/developers/reference/oauth-guide](https://www.dropbox.com/developers/reference/oauth-guide)

[https://www.dropbox.com/developers/apps](https://www.dropbox.com/developers/reference/oauth-guide)

### Workspace Parameters

To create the workspace, make sure access.dropbox is enabled and create a New Workspace using this driver. The following parameters are expected: 

 - **Client ID** : corresponds to the app key of your Dropbox account app (the old options in the main section have been kept for legacy)
 - **Client Secret** : corresponds to the app secret of your Dropbox account app
 - **Scope** : is not used in the Dropbox implementation of OAuth – it needs to stay empty
 - **Authorize URL** : is the one defined in the developer guide : https://www.dropbox.com/1/oauth2/authorize
 - **Token URL** : is the one defined in the developer guide : https://www.dropbox.com/1/oauth2/token
 - **Redirect URL** : this is the tricky one, see below.
 
### Redirect URL
 
It is the url the Dropbox redirects to after the user authorizes the application. It needs to be a public url that points to your Pydio system and that trigger the 'dropbox_oauth_authorize' action. You must thus make sure that the beginning of the URL correspond exactly to where your pydio is publicly available. 

For example, if your pydio server is accessible on https://www.mypydio.com/pydio, then your redirect url will be
https://www.mypydio.com/pydio/index.php?get_action=dropbox_oauth_authorize

You will also need to add this Redirect URL to your Dropbox app. It needs to match perfectly the one you’ve set up on your pydio system

From there, you should be good to go.

## [PYDIO 6 AND BELOW ONLY] Pear OAuth implementation.

Now that you have all the necessary information, you can create a workspace using the Dropbox driver. You will require one specific package from the PEAR library. Make sure to have PEAR installed on your system, and run the following:

    pear install channel://pear.php.net/HTTP_OAuth-0.2.3

Create the workspace, by using the information gathered above:

Your email / password for dropbox account, and the application Key and Secret that you just generated.

Now you have to actually authorize the application, which is done through an OAuth negotiation. Switch to the Dropbox workspace, and you will see a red error message asking you to click on a link. Click on the link before it disappears!

[:image-popup:workspaces/accessing_an_existing_dropbox/screenshot-2013-07-19-at-15-06-11.png]

This will store an authorization token, and you won’t have to redo this, unless you delete the content of data/plugins/access.dropbox.