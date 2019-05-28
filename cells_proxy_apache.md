In this tutorial, we shortly present a basic setup to use an Apache webserver as reverse proxy in front of a Pydio Cells installation.

### Specific Pydio Cells Configuration

During the installation process, instead of the default settings, enter a configuration similar to this:

```conf
Internal URL: cells.example.com:7070
External Host: http://cells.example.com
```

- Internal Host: address where the application http server is bound to. It MUST contain a server name and a port.
- External host: url the end user will use to connect to the application, (in this case the external is the reverse proxy, because it will be used to access Cells)
- Protocol: if you are using SSL, the external host must be starting with `https://` (you don't need to specify the port)
- Example:
  If you want your application to run on the localhost at port 8080 with SSL 
  and use the url `mycells.mypydio.com`,
  then set CELLS_INTERNAL to `localhost:8080` and CELLS_EXTERNAL to `https://mycells.mypydio.com`.

**If you wish to use the 0.0.0.0 address you must respect this rule, cells_bind has to be exactly like this `cells_internal=0.0.0.0:<port>` and `cells_external=<domain name or IP address>:<port>`, the *port* is mandatory in both otherwise you will have a grey screen stuck in the loading**

### Configure Apache

You must enable the following mods with apache :

- `proxy`
- `proxy_http`
- `proxy_wstunnel`

`sudo a2enmod **modname**`

#### A simple example

> For this example, we are using a very simple setup. The proxy is running on a server with this IP `192.168.0.176`, while Pydio Cells is running on another server with IP `192.168.0.172` and listening at port `8080`.

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
  
  # Collabora Online
  # ProxyPassMatch "/lool/(.*)ws$" wss://192.168.0.172:9980/lool/*1/ws nocanon

  # Onlyoffice
  # ProxyPassMatch "/onlyoffice/(.*)/websocket$" ws://192.168.0.172:8080/onlyoffice/$1/websocket nocanon

  #Finally simple proxy instruction
  ProxyPass "/" "http://192.168.0.172:8080/" nocanon
  ProxyPassReverse "/" "http://192.168.0.172:8080/" nocanon

  #Uncomment if you are going to use SSL
  #SSLEngine on
  #SSLCertificateFile /etc/ssl/localcerts/server.crt
  #SSLCertificateKeyFile /etc/ssl/localcerts/server.key
  #SSLCertificateChainFile /etc/ssl/localcerts/bundled.crt

  ErrorLog ${APACHE_LOG_DIR}/error-ssl.log
  CustomLog ${APACHE_LOG_DIR}/access-ssl.log combined
</VirtualHost>
```

### Important points

Please note:

- The **AllowEncodedSlashes** enabled, may be necessary if not activated globally in Apache. It enables API calls like `/a/meta/bulk/path%2F%to%2Ffolder`.
- When configuring Cells, even on another port, **make sure to bind it directly to the cells.example.com** as well (like Apache). This is necessary for the presigned URL used with S3 API for uploads and downloads (they used signed headers and a mismatch between received Host headers may break the signature). Another option is to still bind Cells using a local IP, then in the Admin Settings, under Configs Backend, use the field “Replace Host Header for S3 Signature” and use the internal IP here.
- `nocanon` is required after the **ProxyPass** to be able to download files that contains special characters such as commas/parthensis.