This how-to shows you how to let users authenticate to Cells using their GitHub credentials.

## Create a Github Application

Create a New OAuth Application on **Github**,

[:image-popup:cells/authentication/sso_with_oauth2/github/github_create_app_1.png]

- **Application name:** Name your application
- **Homepage URL:** Your pydio Cells URL (main page)
- **Authorization callback URL:** http(s)://your-pydio**/auth/dex/callback** (the endpoint is the immutable part)

[:image-popup:cells/authentication/sso_with_oauth2/github/github_create_app_2.png]

[:image-popup:cells/authentication/sso_with_oauth2/github/github_create_app_3.png]

## Set the GitHub connector on Pydio Cells

In your Pydio Cells instance go to **Cells Console > Authentication > OAUTH2/OIDC > + Connector**.

[:image-popup:cells/authentication/sso_with_oauth2/github/cells_create_github_oidc_1.png]

[:image-popup:cells/authentication/sso_with_oauth2/github/cells_create_github_oidc_2.png]

Choose **GitHub**.

- **Client ID:** the client ID of your Github application (Fetched from github application, see step 1 )
- **Client Secret:** the client Secret of your Github application (Fetched from github application, see step 1)
- **Callback URL:** the same url defined during the creation of the GitHub application (Fetched from github application, see step 1)

[:image-popup:cells/authentication/sso_with_oauth2/github/cells_create_github_oidc_3.png]