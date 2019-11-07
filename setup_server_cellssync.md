CellsSync is the desktop app providing offline synchronization of files and folders. Written in 100% GO, as the server is, it shares most of its library with Cells. This how-to provides tips and tricks for configuring Cells in a proper way for starting using CellsSync.


## OIDC Authentication

CellsSync provides authentication to the server by using an OpenIDConnect workflow. This means that when a user wants to login in the desktop app, it is redirected to the Cells web UX inside an external browser, and redirected back to the application one the process is complete. 

As the CellsSync frontend must call a specific OIDC discovery endpoint before initiating the authentication workflow, currently the **Self-Signed TLS configuration on the server is not working**, 


If you do want to enable self-signed, e.g. for development or staging case, you have to manually install the rootCA used for generating the server certificate in your desktop local trust store. Note that if you upgraded from Cells V1 with a self-signed mode, you have to re-run the  `cells config proxy tls`  step to regenerate a working configuration (this will create a rootCA that you can install in your trust store).

## Proxy / Firewalls

For best performances and real-time events, CellsSync communicates with the server using a gRPC connection. gRPC is an HTTP/2 protocol, which implies that **HTTP/2 must be enabled on the bind address facing the outside world**.

If you are behind a proxy or inside a private network, you may have to check your proxy settings: 

- **[SSL Enabled]**  Cert/Key couple, Let's Encrypt, Self-signed config (see note below)
  
  Cells will serve HTTP/1.1 and HTTP/2 on the same port, the one you define for external url (e.g; 443, or 8080 or anything you choose). You don't have to open any other port in your firewall.
  
  - **[No Proxy]** Your Cells is directly facing the outside world with a proper SSL configuration, everything should be working out of the box
  - **[Proxy]** Just make sure your proxy is HTTP/2 enabled. If Cells using self-signed configuration (see note below), you can either install the generated rootCA.pem on the proxy machine, or configure the proxy to SkipVerify (for example *insecure_skip_verify* on Caddy).
  
- **[No SSL]** Cells will serve HTTP/1.1 and HTTP/2 on two different ports. By default, gRPC will pick a randomly available port and advertise it in the /a/config/discovery API. The CellsSync client will automagically query this API to connect. 
  
  - **[No firewall, No Proxy]** If you are on a local machine with all ports open, this should work out of the box.
  - **[Firewall and/or Proxy]** You will have to make proper configuration to open and forward the HTTP/2 on this port. To avoid using a random port at each restart, you can fix this port by using the **PYDIO_GRPC_EXTERNAL** environnement variable at startup. Your proxy will probably not be able to serve HTTPS but HTTP only. 

The various cases are summarized in the figure below.

![api_and_grpc_gateways](https://raw.githubusercontent.com/pydio/cells-dist/master/resources/v2.0.0-rc2/api_and_grpc_gateways.png)

## Testing

The way to test if your setup is correct is to simply create a synchronization task in the app. If you can successfully browse workspaces on the server, it means that gRPC is working well..