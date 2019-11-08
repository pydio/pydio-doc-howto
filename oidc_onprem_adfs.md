This how-to shows you how to let users authenticate to Cells Enterprise using the Azure Active Directory Fedaration Service platform deployed on-premises.

## Install and configure ADFS service on windows server 2012

This how-to assumes that you have had already a domain and ADFS server. If you haven't installed ADFS server yet, please visit [this link](https://docs.microsoft.com/en-us/windows-server/identity/ad-fs/deployment/windows-server-2012-ad-fs-deployment-guide) for more information. In our test environment, we have a windows server 2012 running in "win.pyd.io" domain. After installation new AD FS role, the service has SSO URL is: https://win.pyd.io/adfs/ls/

## Add new Relaying Party Trust

This [link](https://docs.microsoft.com/en-us/windows-server/identity/ad-fs/operations/create-a-relying-party-trust) will introduce by steps the how-to register new application to AD FS server. What we do in this section is telling AD FS our server information such as name, callback url, SAML Binding method...

Registered cells application in Relaying Party Trusts

[:image-popup:connectors/connector_onpremiseadfs3.png]

Set endpoints of Cells application in Relaying Party Trusts

[:image-popup:connectors/connector_onpremiseadfs4.png]

## Add new connector in Cells

[:image-popup:connectors/connector_onpremiseadfs1.png]

1. *pydio-saml-onpremise-adfs* is connector identity. It's unique string in the list of connector in Cells.
2. *On-Premise ADFS SAML* the string on login page of Cells to allow user to select connector to authenticate
3. *https://win.pyd.io/adfs/ls/* the url of adfs server that handle the authentication.
4. The path to certificate file which is used by ADFS server to sign saml response
5. The callback url in Cells is always in this format: "https://domain.com/auth/dex/callback
6. The name of attribute in adfs saml response is longer than normal. They are usually:
   1. http://schemas.xmlsoap.org/ws/2005/05/identity/claims/givenname
   2. http://schemas.xmlsoap.org/ws/2005/05/identity/claims/emailaddress
7. 

[:image-popup:connectors/connector_onpremiseadfs2.png]

