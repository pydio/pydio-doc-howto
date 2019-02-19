## Run your Cells docker CContainer behind an Apache reverse Proxy using SSL

The process is pretty much the same as the previous example on apache,
when you run you docker container either with the command `docker run` or in a docker-compose file have to specify `CELLS_BIND` and `CELLS_EXTERNAL`.

Here's what CELLS_BIND and CELLS_EXTERNAL mean to give a you a general understanding.

```
CELLS_BIND : address where the application http server is bound to. It MUST contain a server name and a port.
CELLS_EXTERNAL : url the end user will use to connect to the application.
Example:
If you want your application to run on the localhost at port 8080 and use the url mycells.mypydio.com, then set CELLS_BIND to localhost:8080 and CELLS_EXTERNAL to mycells.mypydio.com
```
**If you wish to use the 0.0.0.0 address you must respect this rule, cells_bind has to be exactly like this `cells_bind=0.0.0.0:<port>` and `cells_external=<domain name,address>:<port>`, the *port* is mandatory in both otherwise you will have a grey screen stuck in the loading**

To illustrate the concept above an example is provided.

For instance you have your cells container running on a server that has an address such as : `192.168.1.12`
you decide to run your container on port `7070` and therefore to run your container behind the proxy you will have  to set:
`CELLS_BIND = 192.168.1.12:7070` and `CELLS_EXTERNAL = 192.168.1.12` 
> (can be done in the docker-compose or as environment variables in the docker run command).

If you want to use SSL do not forget to also put `CELLS_NO_SSL = 0` that is SSL on cells side but even if you want to use SSL for your Apache Proxy you will have to enable it (and set the certificates path for the proxy to use).

Then create configuration file for apache proxy (if used as it is , it will work when you have ssl enabled on both the proxy and cells) with the following:

```conf
<IfModule mod_ssl.c>
<VirtualHost *:443>

  ServerName domain.pydio.com
  # May be necessary for API direct accesses
  AllowEncodedSlashes On
  RewriteEngine On
   # Make sure to proxy SSL
  SSLProxyEngine On
  # Disable SSLProxyCheck : maybe necessary if Cells is configured with self_signed
  SSLProxyCheckPeerCN Off
  SSLProxyCheckPeerName Off
  SSLProxyVerify none

  # The Certificates path you can change the path to another
    SSLCertificateFile /home/user/cert/apache.crt
    SSLCertificateKeyFile /home/user/cert/apache.key

  # Onlyoffice
  # ProxyPassMatch "/onlyoffice/(.*)/websocket$" ws://192.168.0.172:8080/onlyoffice/$1/websocket nocanon  

  # Proxy WebSocket
  RewriteCond %{HTTP:Upgrade} =websocket [NC]
  # RewriteRule /(.*)           wss://192.168.0.12:7070/$1 [P,L]
  # Finally simple proxy instruction
  ProxyPass "/" "https://192.168.1.12:7070/"
  ProxyPassReverse "/" "https://192.168.1.12:7070/"

</VirtualHost>
</IfModule>
```

> you can apply the same set of rules for nginx, caddy etc... .