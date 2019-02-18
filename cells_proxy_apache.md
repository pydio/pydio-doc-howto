//TODO
## Using Apache as a reverse proxy

### Specific Pydio Cells Configuration

During the installation process, instead of the offered settings you should enter configuration similar to this:

```sh
Binding Host (Internal, Other): cells.example.com:7070
External Host: cells.example.com
```

* Binding Host : address where the application http server is bound to. It MUST contain a server name and a port.
* External host : url the end user will use to connect to the application.
* Example:
  If you want your application to run on the localhost at port 8080 and use the url mycells.mypydio.com, then set CELLS_BIND to localhost:8080 and CELLS_EXTERNAL to mycells.mypydio.com

**If you wish to use the 0.0.0.0 address you must respect this rule, cells_bind has to be exactly like this `cells_bind=0.0.0.0:<port>` and `cells_external=<domain name,address>:<port>`, the *port* is mandatory in both otherwise you will have a grey screen stuck in the loading**

### Configure Apache

You must enable the following mods with apache :
- `proxy`
- `proxy_http`
- `proxy_wstunnel`

Edit Apache mod_ssl configuration file to have this:

```conf
Listen 8080
<VirtualHost *:8080>
ServerName demo.fr
ServerAdmin demo.fr
  AllowEncodedSlashes On
  RewriteEngine On
  #SSLProxyEngine On
  #SSLProxyVerify None
  #SSLProxyCheckPeerCN Off
  #SSLProxyCheckPeerName Off

  #Proxy WebSocket
  #RewriteCond %{HTTP:Upgrade} =websocket [NC]
  #RewriteRule /(.*) wss://127.0.0.1:8080/$1 [P,L]
  ProxyPassMatch "/ws/(.*)" ws://192.168.0.172:8080/ws/$1 nocanon
  # for ssl
  # ProxyPassMatch "/ws/(.*)" wss://ip.or.domain.server/ws/$1 nocanon
  
  # onlyoffice
  # ProxyPassMatch "/onlyoffice/(.*)/websocket$" ws://192.168.0.172:8080/onlyoffice/$1/websocket nocanon

  #Finally simple proxy instruction
  ProxyPass "/" "http://192.168.0.172:8080/"
  ProxyPassReverse "/" "http://192.168.0.172:8080/"

  #Uncoment if you are going to use SSL
  #SSLEngine on
  #SSLCertificateFile /etc/ssl/localcerts/server.crt
  #SSLCertificateKeyFile /etc/ssl/localcerts/server.key
  #SSLCertificateChainFile /etc/ssl/localcerts/bundled.crt

ErrorLog ${APACHE_LOG_DIR}/error-ssl.log
CustomLog ${APACHE_LOG_DIR}/access-ssl.log combined
</VirtualHost>
```

> For this example my proxy is running on `192.168.0.176`, while my cells is running on another server `192.168.0.172` under the port `8080` .

Please note:

- The **AllowEncodedSlashes** On that may be necessary if not activated globally in apache (to call APIs like /a/meta/bulk/path%2F%to%2Ffolder

- When I configure Cells, even on another port, I actually **make sure to bind it directly to the cells.example.com** as well (like Apache). This is necessary for the presigned URL used with S3 API for uploads and downloads (they used signed headers and a mismatch between received Host headers may break the signature). Another option is to still bind Cells using a local IP, then in the Admin Settings, under Configs Backend, use the field “Replace Host Header for S3 Signature” and use the internal IP here.