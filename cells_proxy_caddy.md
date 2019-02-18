//TODO
## Using [Caddy](https://caddyserver.com) as a reverse proxy

The example below shows the configuration of a proxy that serves the demo.pydio.com url and redirects to a Pydio Cells environment running on port 8080 of the localhost :

```conf

demo.pydio.com {
  log stdout

  tls /etc/certs/pydio.crt /etc/certs/pydio.key

  timeouts 0

  # And the rest to pydio
  proxy / localhost:8080 {
    insecure_skip_verify
    transparent
    websocket
  }
}
```

If your Caddy reverse proxy is not on the same machine as your Cells instance you may have to take a look at the following clues.


To properly configure the certificates that you want to use, please refer to the [tls plugin page of the caddy documentation](https://caddyserver.com/docs/tls).