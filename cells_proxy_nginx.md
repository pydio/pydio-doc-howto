### Specific Cells parameters

To configure your external and bind for Cells you run the following command:

```
./cells configure sites
```

* **Bind URL**: is the interface and port used to bind Cells on the server.

* **External URL**: is the url used to access Cells from outside.

> Note: in the examples below we use ip addresses but you can also use domains (make sure that they are reachable)

| Cells Server    | Reverse proxy Server |
| --------------- | -------------------- |
| 192.50.0.1:8080 | 192.168.1.201        |


Hence on **Cells Server** we use the following values upon installation, we also want to access Cells through **https**.

> (assuming that we are binding Cells on port 8080, internal_url)

| Bind URL        | External URL          |
| --------------- | --------------------- |
| 192.50.0.1:8080 | https://192.168.1.201 |


To clarify the example, we will access **Cells** through `https://192.168.1.201` while Cells is actually running on another server (192.50.0.1).

### Basic NGINX reverse proxy configuration

```nginx
server {
        client_max_body_size 200M;
        server_name example.pydio.com;

        proxy_send_timeout   600;
        proxy_read_timeout   600;
        proxy_request_buffering off;

        location / {
                proxy_buffering off;
                proxy_pass https://192.50.0.1:8080$request_uri;
                #proxy_pass_request_header@s on;
                #proxy_set_header Host $host;
                proxy_set_header X-Real-IP $remote_addr;
        }

        location /ws {
                proxy_buffering off;
                proxy_pass https://192.50.0.1:8080;
                proxy_set_header Upgrade $@http_upgrade;
                proxy_set_header Connection "upgrade";
                proxy_read_timeout 86400;
        }

    error_log /var/log/nginx/cells-proxy-error.log;
    access_log /var/log/nginx/cells-proxy-access.log;

    listen [::]:443 ssl http; 
    listen 443 ssl http;
    ssl_certificate     www.example.com.crt;
    ssl_certificate_key www.example.com.key;
    ssl_protocols       TLSv1 TLSv1.1 TLSv1.2;
    ssl_ciphers         HIGH:!aNULL:!MD5;
}

server {
    if ($host = example.pydio.com) {
        return 301 https://$host$request_uri;
    } 

    listen 80 http;
    listen [::]:80 http;
    server_name example.pydio.com;
    return 404;
}
```

### Cells Sync

**Mandatory section for the Sync Client to work behind a Nginx reverse proxy.**

If your Cells Server is running behind a Nginx reverse proxy you must meet 2 requirements and then add the config below to your main nginx reverse proxy configuration.

TLS and HTTP2 meaning that the reverse proxy and Cells must communicate with SSL (you can use the self signed option during installation).

Once that is done you must set a port for **grpc** in this example it's **33060**,

to set it you have the folllowing env variable, `PYDIO_GRPC_EXTERNAL=33060`

otherwise you can set it when running the binary with the following flag **--grpc_external**, for instance;

`./cells start --grpc_external=33060`



> In all those examples you can subsitute the port 33060 by the port of your choice

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
    grpc_pass grpcs://192.50.0.1:33060;
  }@
  
  error_log /var/log/nginx/proxy-grpc-error.log;
  access_log /var/log/nginx/proxy-grpc-access.log;
}
```

> Below you can have a look at the complete file

#### Finale note

Make sure to substitute the values of the **certificates** and **ip/domains**.



#### The complete configuration

**cells.conf**

```nginx
server {
    client_max_body_size 200M;
    server_name example.pydio.com;

    location / {
            proxy_buffering off;
            proxy_pass https://192.50.0.1:8080$request_uri;
            #proxy_pass_request_header@s on;
            #proxy_set_header Host $host;
            proxy_set_header X-Real-IP $remote_addr;
    }

    location /ws {
            proxy_buffering off;
            proxy_pass https://192.50.0.1:8080;
            proxy_set_header Upgrade $@http_upgrade;
            proxy_set_header Connection "upgrade";
            proxy_read_timeout 86400;
    }

    error_log /var/log/nginx/cells-proxy-error.log;
    access_log /var/log/nginx/cells-proxy-access.log;

    listen [::]:443 ssl http; 
    listen 443 ssl http;
    ssl_certificate     www.example.com.crt;
    ssl_certificate_key www.example.com.key;
    ssl_protocols       TLSv1 TLSv1.1 TLSv1.2;
    ssl_ciphers         HIGH:!aNULL:!MD5;
}

server {
    if ($host = example.pydio.com) {
        return 301 https://$host$request_uri;
    } 

    listen 80 http;
    listen [::]:80 http;
    server_name example.pydio.com;
    return 404;
}

server {
	listen 33060 ssl http2;
	listen [::]:33060 ssl http2;
  ssl_certificate     www.example.com.crt;
  ssl_certificate_key www.example.com.key;
  ssl_protocols       TLSv1 TLSv1.1 TLSv1.2;
  ssl_ciphers         HIGH:!aNULL:!MD5;
  keepalive_timeout 600s;
	
    location / {
		grpc_pass grpcs://192.50.0.1:33060;
	}@
  
  error_log /var/log/nginx/proxy-grpc-error.log;
  access_log /var/log/nginx/proxy-grpc-access.log;
}
```

--------------------------------------------------------------------------------------------------------
_See Also_

[Running Cells Behind a reverse proxy](en/docs/cells/v2/run-cells-behind-proxy)