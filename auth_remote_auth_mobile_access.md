For those of you using the auth.remote plugin in slave mode with an external CMS like Drupal, WordPress or Joomla!, the RESTful access to Pydio necessary to create a communication with the mobile (iOS / Androïd) clients is tricky, but should work. It’s all about setting the right configuration for the various MASTER_XXX options of the auth.remote driver (via the GUI).

### Technical background : read it!
When accessing Pydio through the web in an “auth.remote” configuration, the login action is never performed via Pydio, but it is done via your CMS login form and your CMS triggers the necessary actions to log your users in Pydio as well.

However,  the mobile clients are not supposed to know how to connect to e.g Drupal, and can only try to authenticate against Pydio.  In that case, once the authentication is submitted to Pydio, a remote call mechanism  will let Pydio submit  the authentication credentials to the CMS :  it means that Pydio acts “like a user” and will grab the CMS login form, fill it, and submit it. At this point, if the login is successfull, the CMS will log Pydio back!

WordPress implementation is quite straightforward, but Joomla & Drupal necessits 2 calls, one to get the login form secure tokens, and a second for to submit the form with these values. This is performed by loading the HTML page that contains the form, parsing it, find the login form identified by it’s HTML ID, and grab the hidden inputs key/values to submit the POST query.

For this reason, you’ll have to make sure that the **_MASTER_URL / LOGIN_URL_** points to a page that indeed displays the login form, that this form’s HTML id is the right one as defined by **_Auth Form Id_**. For Joomla, the default id is **login-form**, but some installs seems to use **form-login** instead. For Drupal, the default id is user-login-form. For wordpress it is not necessary.

[:image-popup:authentication/remote_auth_mobile_access/screenshot-2013-05-13-at-11-37-34.png]

### Troubleshooting
If it’s still not working, and you see an error concerning  “loadHTML() function failing” in your log file, the page HTML is probably creating errors at parsing time. In that case, create a simple login page with the least HTML possible, but with just the login form of your CMS.

When testing the configuration, you may have to try several times, and soon you will be locked out of the server by the brute force login defence mechanism. In that case, searche for the **failedAJXP.log** file inside the PHP temporary folder and delete it, and this should work! Otherwise simply turn the server in debug mode (see AJXP_SERVER_DEBUG on bootstrap_context.php), this will avoid the brute force login mechanism.

### My CMS is not in the list!
Don’t panic, you will “simply” have to implement your own function to perform the authentication submission. The idea is to use an HttpClient to POST the right values to the CMS. Add this function in the auth.remote/cms_auth_functions.php file, and check the existing implementations to see how it’s done. In some case, only a POST is necessary, in other there is a two-way communication, in that case you can use a DOMDocument to parse the HTML response of the CMS.