# Enable SSO on WordPress with Cells as the identity provider

Resources :

- https://wordpress.org/plugins/miniorange-login-with-eve-online-google-facebook/
- https://pydio.com/en/docs/cells/v2/cells-identity-provider

### Create a OAUTH2 client for WordPress on Pydio Cells

On the left-bar menu, go to **Authentication** > **OAUTH2/OIDC** (top left) > **+ OAUTH2 Client** to create a new client.

You must provide the following information:

|                    |                                      |                                    |
| ------------------ | ------------------------------------ | ---------------------------------- |
| **Name**           | `cellswordpress`                     | put any label of your choice       |
| **Client ID**      | `cells-wp`                           | this setting is a sort of username |
| **Client Secret**  | a secret (like a password)           | this setting is a sort of username |
| **Scope**          | `openid email profile pydio offline` |                                    |
| **Callback URL**   | `https://my-wordpress.com`           |                                    |
| **Grant Types**    | `Authorization code`                 |                                    |
| **Response Types** | `Code`                               |                                    |



### Configure OAuth2 client on WordPress

Download and install the following plugin on WordPress:

- https://wordpress.org/plugins/miniorange-login-with-eve-online-google-facebook/ 
- Make sure to enable the plugin (Plugins menu)


Then proceed on the Plugins settings (left bar menu), click on **miniOrange OAuth**,

you are now invited to configure your OAuth Provider,

|                           |                                                   |
| ------------------------- | ------------------------------------------------- |
| **Client ID**             | you must put the previously created Client ID     |
| **Client Secret**         | you must put the previously created Client Secret |
| **Authorize Endpoint**    | https://your-cells**/oidc/oauth2/auth**           |
| **Access Token Endpoint** | https://your-cells**/oidc/oauth2/token**          |

[:image-popup:cells/authentication/sso_with_oauth2/wordpress/wordpress_oauth_1.png]

Then hit the **Test Configuration button**, once you have the test-configuration (presented in an Array),
keep it open as it is required later for the mapping.

[:image-popup:cells/authentication/sso_with_oauth2/wordpress/wordpress_oauth_3.png]

Browse to **Attribute/Role mapping** and put inside the **Username** field the previously retrieved Attribute, `name` (see screenshot above).

[:image-popup:cells/authentication/sso_with_oauth2/wordpress/wordpress_oauth_3.png]