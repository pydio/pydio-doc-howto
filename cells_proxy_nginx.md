### Specific Cells parameters

To configure your external and bind for Cells you run the following command:

```
./cells configure sites
```

* **Bind Address**: is the interface and port used to bind Cells on the server.

* **External URL**: is the url used to access Cells from outside (in this case, the reverse proxy).

### Basic NGINX reverse proxy configuration

```nginx
server {
    server_name my-cells-server.com;
    client_max_body_size 200M;

    location / {
        # include proxy_params;
        proxy_pass http://localhost:8080;
    }


location /ws/ {
    proxy_pass http://localhost:8080;
    proxy_http_version 1.1;
    proxy_set_header Upgrade $http_upgrade;
    proxy_set_header Connection "Upgrade";
}

    error_log /var/log/nginx/cells-proxy-error.log;
    access_log /var/log/nginx/cells-proxy-access.log;

    listen [::]:443 ssl ipv6only=on; # managed by Certbot
    listen 443 ssl; # managed by Certbot
    ssl_certificate /etc/letsencrypt/live/my-cells-server.com/fullchain.pem; # managed by Certbot
    ssl_certificate_key /etc/letsencrypt/live/my-cells-server.com/privkey.pem; # managed by Certbot
    include /etc/letsencrypt/options-ssl-nginx.conf; # managed by Certbot
    ssl_dhparam /etc/letsencrypt/ssl-dhparams.pem; # managed by Certbot

}


server {
    if ($host = my-cells-server.com) {
        return 301 https://$host$request_uri;
        } # managed by Certbot


        listen 80;
        listen [::]:80;
        server_name my-cells-server.com;
        return 404; # managed by Certbot
    }
```

> This config was updated with nginx version: nginx/1.14.2

### Cells Sync

**Mandatory section for the Sync Client to work behind a Nginx reverse proxy.**

If your Cells Server is running behind a Nginx reverse proxy you must meet 2 requirements and then add the config below to your main nginx reverse proxy configuration.

TLS and HTTP2 meaning that the reverse proxy and Cells must communicate with SSL (you can use the self signed option during installation).

Once that is done you must set a port for **grpc** in this example it's **33060**,

to set it you have the following env variable, `CELLS_GRPC_EXTERNAL=33060`

otherwise you can set it when running the binary with the following flag **--grpc_external**, for instance;

`./cells start --grpc_external=33060`


> In all those examples you can substitute the port 33060 by the port of your choice

Also make sure to put the **address/domain** on which your **Cells Server** is running (refer to the arrays above) line **grpc_pass**.

```nginx
server {
  listen 33060 ssl http2;
  listen [::]:33060 ssl http2;
  ssl_certificate     www.example.com.crt;
  ssl_certificate_key www.example.com.key;
  ssl_protocols       TLSv1 TLSv1.1 TLSv1.2;
  ssl_ciphers         HIGH:!aNULL:!MD5;
  keepalive_timeout 600s;
  
  location / {
    grpc_pass grpcs://localhost:33060;
  }
  
  error_log /var/log/nginx/proxy-grpc-error.log;
  access_log /var/log/nginx/proxy-grpc-access.log;
}
```

> Below you can have a look at the complete file

#### Finale note

Make sure to substitute the values of the **certificates** and **ip/domains**.

--------------------------------------------------------------------------------------------------------
_See Also_

[Running Cells Behind a reverse proxy](../../cells/v2/configure-cells-reverse-proxy)