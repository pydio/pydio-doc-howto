Resources :

- https://wordpress.org/plugins/miniorange-login-with-eve-online-google-facebook/
- https://pydio.com/en/docs/cells/v2/cells-identity-provider

## Create an OAuth2 client for WordPress on Pydio Cells

This is possible on both the **Home** and **Enterprise** Edition.

### Pydio Cells Enterprise Edition

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


### Pydio Cells Home Edition

On Pydio Cells Home edition you must manually add to your **pydio.json**, the oauth2 client, below is a default sample:

```json
{
    "client_id": "cells-wp",
    "client_name": "cellswordpress",
    "client_secret": "secret",
    "grant_types": [
    "authorization_code"
    ],
    "redirect_uris": [
    "https://my-wordpress.com"
    ],
    "response_types": [
    "code"
    ],
    "scope": "email user pydio openid offline"
}
```

You must add it inside the `staticClients` array (located inside your **pydio.json** file).

**pydio.json:**
``` json
"staticClients": [
    // It must be placed here
]
```

## Configure the OAuth2 client on WordPress

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
keep it open as it is required later for the mapping, and proceed to **Save**.

[:image-popup:cells/authentication/sso_with_oauth2/wordpress/wordpress_oauth_3.png]

Browse to **Attribute/Role mapping** and put inside the **Username** field the previously retrieved Attribute, `name` (see screenshot above).

[:image-popup:cells/authentication/sso_with_oauth2/wordpress/wordpress_oauth_2.png]

Hit save and now you can connect to your WordPress with your PydioCells credentials.

[:image-popup:cells/authentication/sso_with_oauth2/wordpress/wordpress_oauth_4.png]
