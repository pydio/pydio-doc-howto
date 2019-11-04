### Register cells application in Azure
Please visit [this article](https://docs.microsoft.com/en-us/graph/auth-register-app-v2) to register new application on azure.
- The redirect URI (or reply URL) for cells application is always in format: https://server.cells.domain/auth/dex/callback
- You should create a new "client secret" in "Certificates and secrets" of new registed application
  [:image-popup:connectors/connector_azure_08.png]

When finished, takes a note on following information:
- Directory (tenent) ID
- Client Secret
- The OpenID Connect metadata document ([document link](https://login.microsoftonline.com/tenentId/v2.0)). 
  
  In Cells, the metadata url is automatically added ".well-known/openid-confuguration" at the end. It looks like: https://login.micosoftonline.com/[tenant_id]/v2.0
   [:image-popup:connectors/connector_azure_07.png]  

## Add new connector in Cells
When you finished the registration new app in Azure, go to the admin console of Cells to add a new connector: type "Microsoft"
  [:image-popup:connectors/connector_azure_01.png]

  [:image-popup:connectors/connector_azure_01.png]

