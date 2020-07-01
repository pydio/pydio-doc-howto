## About Microsoft Identity Platform

Microsoft Identity Platform offers registration and configuration of applications that sign in all Microsoft Identities and get tokens to call Microsoft APIs like Microsoft Graph or other APIs. It consists of a fully compliant OAuth2 and OpenID Connect standard-compliant authentication service. It is an evolution of the Azure Active Directory (Azure AD) developer platform.

### Register cells application with the Microsoft Identity Platform
Please visit [this article](https://docs.microsoft.com/en-us/graph/auth-register-app-v2) to register a new application.

*:warning: The callback url (or redirect uri) is generated in the step below. The format of the url may vary depending on the version of Cells so please refer to your admin console.*

## Add new connector in Cells
When you finished the registration new app in Azure, go to the admin console of Cells to add a new connector: type "Microsoft"

## Add an Microsoft Identity connector in Cells

Navigate to the Admin Console > Authentication > OAUTH2 / OIDC

Use the following configuration example to create a new connector :

### Connector Options
- Connector type : Microsoft
- Id: <your_id_here>
- Name: <your_name_here> (the name will appear to the end user in the Login dialog box)

### Microsoft Options
- Credentials:
  - Client ID: <your_client_id> (created in the precedent step)
  - Client Secret: <your_client_secret> (created in the precedent step)
- Redirect URI: (**generated - use it to register cells with the Microsoft Identity platform**)
- Tenant: <your_tenant_id> (created in the precedent step)

[:image:connectors/connector_azure_01.png]