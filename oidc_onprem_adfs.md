## About On-Premise ADFS

Azure Active Directory Federation Service platform deployed on-premise provides Single Sign-On on multiple systems and applications from a Windows Server to Windows Users using the SAML2.0 protocol.

## Install and configure ADFS service on windows server 2012

Install following [this guide](https://docs.microsoft.com/en-us/windows-server/identity/ad-fs/deployment/windows-server-2012-ad-fs-deployment-guide)

## Register Cells as a Relying Party Trust in ADFS

*:warning: The callback url (or redirect uri) is generated in the next step. The format of the url may vary depending on the version of Cells so please refer to your admin console.*

Create a Relying Party Trust following [this guide](https://docs.microsoft.com/en-us/windows-server/identity/ad-fs/operations/create-a-relying-party-trust).

On the **Configure URL** page, select the **Enable support for the SAML 2.0 WebSSO protocol** checkbox and enter the callback url generated during the creation of the connector as the Relying party SAML 2.0 SSO service URL.

It can also be configured afterwards in the Endpoints tab of the Relying Party Trust.

[:image:connectors/connector_onpremiseadfs03.png]

On the **Configure Identifiers** page, add your Cells URL as a Relying Party Trust Identifier. It needs to exactly match with the **Entity issuer** URL you specified in the connector configuration.

It can also be configured afterwards in the Identifiers tab of the Relying Party Trust.

[:image:connectors/connector_onpremiseadfs04.png]

## Add an ADFS connector in Cells

Navigate to the Admin Console &gt; Authentication &gt; OAUTH2 / OIDC

Use the following configuration example to create a new connector :

### Connector Options
- Connector type : ```SAML```
- Id: ```<your_id_here>```
- Name: ```<your_name_here>``` (the name will appear to the end user in the Login dialog box)

### SAML Options
- SSO URL user for POST Value: ```https://<your_adfs_url_here>/adfs/ls```
- CA to use when validating the signature of the SAML response : ```<your_certificate_path_on_the_cells_server>```
- Callback URL : (**generated - use it to register cells as a relying party trust in adfs**)
- Name of the attributes to map in the ID Token Claims: (may vary for your usecase)
  - Username: ```http://schemas.xmlsoap.org/ws/2005/05/identity/claims/givenname```
  - Email: ```http://schemas.xmlsoap.org/ws/2005/05/identity/claims/emailaddress```
- Manually specify issuers value:
  - Entity issuer: ```https://<your_cells_url_here>```

[:image:connectors/connector_onpremiseadfs1.png]
[:image:connectors/connector_onpremiseadfs2.png]