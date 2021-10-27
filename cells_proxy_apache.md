In this tutorial, we present a basic setup to use an Apache webserver as reverse proxy in front of a Pydio Cells installation.

### Specific Pydio Cells Configuration

During the installation process, instead of the default settings, enter a configuration similar to this:

```conf
+---+------------------------+-------------+---------------------------------------------+
| # |        BIND(S)         |     TLS     |                EXTERNAL URL                 |
+---+------------------------+-------------+---------------------------------------------+
| 0 | https://localhost:8080 | Self-signed | https://cells.example.com                   |
+---+------------------------+-------------+---------------------------------------------+
```

To configure the external URL on Cells, run the following command:

```
./cells configure sites
```

- Bind Address: interface and port on which Cells server is bound. It MUST contain a server name (or IP) and a port.
- External URL: the public URL that you communicate to your end users, in our case, this should point towards the reverse proxy. Note that the external URL must contain the protocol (http or https) depending on wether you support TLS at the Apache layer or not.

### Configure Apache

You must enable the following mods with Apache :

- `proxy`
- `proxy_http`
- `proxy_wstunnel` [official documentation](https://httpd.apache.org/docs/2.4/mod/mod_proxy_wstunnel.html)

To enable a module with apache use `sudo a2enmod <mod_name>`.

#### A simple example

> For this example, we are using a simple setup. The proxy is running on a server under the hostname `cells.example.com` while Pydio Cells is bound on `localhost` with port `8080` with TLS enabled.

Create or Edit your apache virtual host configuration with :

```apache
<VirtualHost *:80>
	ServerName cells.example.com

	RewriteEngine On
	RewriteCond %{HTTPS} off
	RewriteRule (.*) https://%{HTTP_HOST}%{REQUEST_URI}
  
  RewriteCond %{SERVER_NAME} =cells.example.com
  RewriteRule ^ https://%{SERVER_NAME}%{REQUEST_URI} [END,NE,R=permanent]
</VirtualHost>

<VirtualHost *:443>
	ServerName cells.example.com
	AllowEncodedSlashes On
	RewriteEngine On
	SSLProxyEngine On

	## The order of the directives matters.
  # If Cells is not running with https, consider using ws instead of wss
	ProxyPassMatch "/ws/(.*)" wss://localhost:8080/ws/$1 nocanon

  ## This rewrite condition is required if using Cells-Sync
	# RewriteCond %{HTTP:Content-Type} =application/grpc [NC]
	# RewriteRule /(.*) h2://localhost:8080/$1 [P,L]
	
  ProxyPass "/" "https://localhost:8080/"	
	ProxyPassReverse "/" "https://localhost:8080/"
		
	ErrorLog ${APACHE_LOG_DIR}/error.log
	CustomLog ${APACHE_LOG_DIR}/access.log combined

	SSLCertificateFile /etc/letsencrypt/live/cells.example.com/fullchain.pem
	SSLCertificateKeyFile /etc/letsencrypt/live/cells.example.com/privkey.pem
  Include /etc/letsencrypt/options-ssl-apache.conf
</VirtualHost>
```

_**This was tested with Apache/2.4.51 (Debian)**_

### Important points

Please note:

- The `AllowEncodedSlashes On` directive is necessary if it is not globally activated in Apache. It enables API calls like `/a/meta/bulk/path%2F%to%2Ffolder`.
- When configuring Cells, even on another port, **make sure to bind it directly to the cells.example.com** as well (like Apache). This is necessary for the presigned URL used with S3 API for uploads and downloads (they used signed headers and a mismatch between received Host headers may break the signature). Another option is to still bind Cells using a local IP, then in the Admin Settings, under Configs Backend, use the field “Replace Host Header for S3 Signature” and use the internal IP here.
- `nocanon` is required after the **ProxyPass** to be able to download files which names contain special characters such as commas or quotes.
- **websocket**: make sure to also proxy the websocket depending on your protocol (`ws`, or `wss`) and enable the module.
- `RewriteEngine` is required to enable rewriting requests. 


### Cells Sync

For **Cells-Sync** to work it is required to proxy the grpc requests using the http2 module, for that 2 prerequisites need to be met:

- Enable apache module `http2` & `proxy_http2`.
- `http2` does not work with the module `mpm_prefork` therefore it is required to use `mpm_event` instead (see [apache official documentation](https://httpd.apache.org/docs/2.4/howto/http2.html)).


Once you have met the prerequisites, enable the http2 proxy with the following directive (h2 for TLS or h2c for clear text), for more information please refer to Apache's official documentation.

```
## The order of the directive is important please see the full example above.
# RewriteCond %{HTTP:Content-Type} =application/grpc [NC]
# RewriteRule /(.*) h2://localhost:8080/$1 [P,L]
```

_All requests done from Cells-Sync have the header `Content-Type` set to `application/grpc`_.

--------------------------------------------------------------------------------------------------------
_See Also_

[Running Cells Behind a reverse proxy](en/docs/cells/v3/run-cells-behind-proxy)
