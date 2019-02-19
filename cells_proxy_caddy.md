## Using Caddy as a reverse proxy

[:image-popup:/devops/caddy-black.png]

Caddy is a lightweight webserver coded in go, it's also the server embed with pydio cells that exposes the services.
Caddy is easy to configure and to use visit the caddy site to download and read the documentation for a more advanced configuration.
[Caddy website](https://caddyserver.com)

The example below shows the configuration of a proxy that serves the demo.pydio.com url and redirects to a Pydio Cells environment running on port `8080` of the following address `192.168.0.34` :

For instance you have a pydio cells running on `internal url = 192.168.0.34:8080 and external_url = 192.168.0.162` (you can notice that the external_url is the proxy address) and your caddy proxy is running on a server with the following address
`192.168.0.162`.

```conf

192.168.0.162 {
  log stdout

  # if you want to use tls with self signed, etc... refer to the documentation for more details
  # tls /etc/certs/pydio.crt /etc/certs/pydio.key

  timeouts 0

  # And the rest to pydio
  proxy / 192.168.0.34:8080 {
    insecure_skip_verify
    transparent
    websocket
  }
}
```

> The `websocket` to forward ws connections and `transparent` for the headers.

If your Caddy reverse proxy is not on the same machine as your Cells instance you may have to take a look at the following clues.
To properly configure the certificates that you want to use, please refer to the [tls plugin page of the caddy documentation](https://caddyserver.com/docs/tls).