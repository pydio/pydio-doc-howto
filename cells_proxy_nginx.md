This _how to_ will give you all the hints with a simple configuration allowing you to run Pydio Cells behind your Nginx reverse proxy.

### Specific Cells parameters

During your Pydio Cells installation (actually at the beginning) you will be prompted for 2 important parameters:

- Internal Url: is the url used to bind your cells to the server.
- External Url: is the url used to access your cells from outside.

So in our case for the reverse proxy, we must pay extra attention to **External URL** as it has to match the address of our reverse proxy that is going to be used to access Cells.

For instance we have **Cells (192.168.0.12)** and the **Reverse proxy (192.168.1.201)** running on 2 different machines(servers, etc...).
| Cells Server | Reverse Proxy Server |
| ------------ | -------------------- |
| 192.168.0.12 | 192.168.1.201        |

Therefore on you **Cells Server** we use the following values upon installation (assuming that we want to run Cells on port `8080`),
we also want to access the Cells instance through port `80` of the reverse proxy.

| internal url      | external url         |
| ----------------- | -------------------- |
| 192.168.0.12:8080 | http://192.168.1.201 |

Now that we are all set let's take a look a the reverse proxy configuration.

### Nginx reverse proxy configuration

This is a basic configuration that you can use and modify to meet your needs.

```conf
server {
        listen 80;
        listen [::]:80;
        client_max_body_size 200M;
        server_name reverse.proxy;

        location / {
                proxy_buffering off;
                proxy_pass http://192.168.0.12:8080$request_uri;
                #proxy_pass_request_headers on;
                #proxy_set_header Host $host;
                proxy_set_header X-Real-IP $remote_addr;
        }

        location /ws {
                proxy_buffering off;
                proxy_pass http://192.168.0.12:8080;
                proxy_set_header Upgrade $http_upgrade;
                proxy_set_header Connection "upgrade";
                proxy_read_timeout 86400;
        }

        error_log /var/log/nginx/cells-proxy-error.log;
        access_log /var/log/nginx/cells-proxy-access.log;


}
```

> As you can notice the proxy_pass is the **internal_url**.

Then once you restarted your Nginx, you can go to `http://192.168.1.201` and you will have access to Pydio Cells.