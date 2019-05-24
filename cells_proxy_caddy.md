In this how to, we are going to take a look at the Caddy webserver and how it can be configured as a reverse proxy.

[:image-popup:/devops/caddy-black.png]

Caddy is a lightweight webserver coded in go, it is also the server embedded with pydio cells that exposes the services.
For more details and to download, visit the [Caddy website](https://caddyserver.com).

The example below shows the configuration of a proxy that serves the `demo.pydio.com` url and redirects to a Pydio Cells.

For instance you have a Pydio Cells running on `internal url = 0.0.0.0:8080 and external_url = https://demo.pydio.com` (you can notice that the external_url is also the proxy url).

```conf

https://demo.pydio.com {
  log stdout
  timeouts 0
  
  # if you want to use tls with self signed, etc... refer to the documentation for more details
  # tls /etc/certs/pydio.crt /etc/certs/pydio.key

  # And the rest to pydio
  proxy / localhost:8080 {
    insecure_skip_verify
    transparent
    websocket
  }
}
```

_This first example is a Caddy reverse-proxy running on the same machine as Cells._

For this case using the Caddyfile above, you must use one of the following values for the **internal URL**:

* `http://0.0.0.0:8080`
* `http://demo.pydio.com`

Detail about some directives:

> The `websocket` to forward ws connections and `transparent` for the headers.

If your Caddy reverse proxy is not on the same machine as your Cells instance you may have to take a look at the following clues.
To properly configure the certificates that you want to use, please refer to the [tls plugin page of the caddy documentation](https://caddyserver.com/docs/tls).