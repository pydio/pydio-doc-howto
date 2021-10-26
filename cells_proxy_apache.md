In this tutorial, we present a basic setup to use an Apache webserver as reverse proxy in front of a Pydio Cells installation.

**_Cells Sync will not be able to work with apache2, apache is currently not completely supporting gRPC._**

>> You _can_ use Apache in front of the webUI, but if you want to use **Cells Sync** you must use another reverse proxy, at least for the sync (**more details below**).

### Specific Pydio Cells Configuration

During the installation process, instead of the default settings, enter a configuration similar to this:

```conf
Bind Address: cells.example.com:7070
External URL: https://cells.example.com
```

To configure the external URL on Cells, run the following command:

```
./cells configure sites
```

- Bind Address: interface and port on which Cells server is bound. It MUST contain a server name (or IP) and a port.
- External URL: the public URL that you communicate to your end users, in our case, this should point towards your reverse proxy. Note that the external URL must contain the protocol (http or https) depending on wether you support TLS at the Apache layer or not.

Assuming that you want to run Apache and Cells on the same machine, and that port 8080 is available for Cells to bind to, you can set e.g.:

- bind address to `localhost:8080` 
- external address to `https://cells.example.com`.

### Configure Apache

You must enable the following mods with Apache :

- `proxy`
- `proxy_http`
- `proxy_wstunnel`

`sudo a2enmod **modname**`

#### A simple example

> For this example, we are using a simple setup. The proxy is running on a server with IP `192.168.0.176` (or domain cells.example.com), while Pydio Cells is running on another server with IP `192.168.0.172` and listening at port `8080`.

Edit Apache mod_ssl configuration file to have this:

```apache
Listen 8080
<VirtualHost *:8080>

  ServerName cells.example.com
  ServerAdmin cells.example.com
  AllowEncodedSlashes On
  RewriteEngine On
 
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

  # Finally simple proxy instruction
  ProxyPass "/" "http://192.168.0.172:8080/" nocanon
  ProxyPassReverse "/" "http://192.168.0.172:8080/" nocanon

  # Uncomment the following if you have valid TLS certificates and want to use SSL
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

- The `AllowEncodedSlashes on` directive is necessary if it is not globally activated in Apache. It enables API calls like `/a/meta/bulk/path%2F%to%2Ffolder`.
- When configuring Cells, even on another port, **make sure to bind it directly to the cells.example.com** as well (like Apache). This is necessary for the presigned URL used with S3 API for uploads and downloads (they used signed headers and a mismatch between received Host headers may break the signature). Another option is to still bind Cells using a local IP, then in the Admin Settings, under Configs Backend, use the field “Replace Host Header for S3 Signature” and use the internal IP here.
- `nocanon` is required after the **ProxyPass** to be able to download files which names contain special characters such as commas or quotes.
- **websocket**: make sure to also proxy the websocket depending on your protocol (ws, or wss).


### Cells Sync

Unfortunately Apache does not seem to completely support gRPC. Therefore you have to use another software to reverse proxy the gRPC part (tied to the Cells Sync desktop application).

- You are running Cells with SSL (can be self-signed).
- You have set the port with the following env variable `CELLS_GRPC_EXTERNAL=33060`.
  
> Note: the port defined for the gRPC can use any value.

#### Caddy webserver

You can run caddy with the following configuration:

```config
cells.example.com:33060 {
	tls /var/certs/cells/cert.pem /var/certs/cells/key.pem
	log stdout
	errors stdout

	proxy / https://192.168.0.172:33060 {
		websocket
		insecure_skip_verify
	}
}
```

#### Nginx

You can run nginx with the following configuration, which will allow you to connect with gRPC and use the Sync desktop application.

```nginx
server {
	listen 33060 ssl http2;
	listen [::]:33060 ssl http2;
  ssl_certificate     cells.example.com.crt;
  ssl_certificate_key cells.example.com.key;
  ssl_protocols       TLSv1 TLSv1.1 TLSv1.2;
  ssl_ciphers         HIGH:!aNULL:!MD5;
	keepalive_timeout 600s;
  
    location / {
		grpc_pass grpcs://192.168.0.172:33060;
	}
  
  error_log /var/log/nginx/proxy-grpc-error.log;
  access_log /var/log/nginx/proxy-grpc-access.log;
}
```

--------------------------------------------------------------------------------------------------------
_See Also_

[Running Cells Behind a reverse proxy](en/docs/cells/v2/run-cells-behind-proxy)
