In this tutorial, we explain how to use a Caddy webserver as reverse proxy in front of a Pydio Cells installation.

[:image-popup:/devops/caddy-black.png]

Caddy is a lightweight webserver coded in Go language. It is embedded in Pydio Cells to serve as a gateway and expose the various services.
For more details and to download, please visit the [Caddy website](https://caddyserver.com).

In this example, we have a Pydio Cells running with following URLs, **on the same machine** as the Caddy reverse proxy:

- Internal (bind) URL: `share.example.com:8080`
- External (public) URL `https://share.example.com` - you can notice that the external URL is also the proxy URL.

The caddy file of the reverse proxy loooks like:

```conf
https://share.example.com {

  log stdout
  timeouts 0
  
  # if you want to use tls with self signed, etc... refer to the documentation for more details
  # tls /etc/certs/example.com.crt /etc/certs/example.com.key

  # And the rest to pydio
  proxy / localhost:8080 {
    transparent
    websocket
  }

}
```

Detail about some directives:

> The `websocket` to forward ws connections and `transparent` for the headers.

If your Caddy reverse proxy is not on the same machine as your Cells instance you may have to take a look at the following clues.
To properly configure the certificates that you want to use, please refer to the [tls plugin page of the caddy documentation](https://caddyserver.com/docs/tls).