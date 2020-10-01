In this tutorial, we explain how to use a Caddy **2** webserver as reverse proxy in front of a Pydio Cells installation.  
Caddy 2 is the latest version of the [Caddy](https://caddyserver.com) webserver. It brings many enhancements but _also some breaking changes when migrating from version 1_; typically in the `CaddyFile` configuration file.

In this example, Pydio Cells is running, **on the same machine** as the Caddy reverse proxy, with following configuration:

- Internal (bind) URL: `0.0.0.0:8080`
- External (public) URL: `https://share.example.com` - please note that the external URL is also the proxy URL.
- TLS: self-signed.

We use https with self-signed certificate between Caddy and Cells so that the Cells Sync Desktop tool can communicate with the backend:  
the communication between the server and the Sync client uses the gRPC protocol on HTTP2 and this protocol forbids TLS dropping at the reverse proxy layer.

Yet, public end-point uses the Let's Encrypt certificate that is provided by Caddy by default.

In a simple case, the caddy file of the reverse proxy looks then like this:

```conf
https://share.example.com {

  tls tls@example.com

  reverse_proxy localhost:8080 {
    # Use https with a self signed cert between Caddy and Cells
    transport http {
      tls
      tls_insecure_skip_verify
    }
  }
}

# You might also want to redirect HTTP traffic toward HTTPS  
http://share.example.com {
  redir https://share.example.com
}
```

If you are running the Enterprise Distribution and want [to enable Prometheus](en/docs/kb/deployment/monitoring-cells-prometheus-grafana), you might want to add these 2 directives in the main block (**before** the reverse proxy directive for Cells):

```conf
https://share.example.com {

  tls tls@example.com

  route /prometheus* {
    reverse_proxy localhost:9090
  }

  basicauth /prometheus* {
    sysadminUser JDJhJDE0JGZad2Y0VHF5OWtHb2NaV3BnMC9jak9jRTkzUi9pSnMxWUM0cmxMWVhJSFguaWtCYnYxdEZt
  }

  reverse_proxy localhost:8080 {
    # Use https with a self signed cert between Caddy and Cells
    transport http {
      tls
      tls_insecure_skip_verify
    }
  }
}
```

**Notes:**

- We use caddy2 basic authentication mechanism to protect prometheus end-point, otherwise anybody knowing the URL could access the metrics of your instance.
- 2nd argument of the `basicauth` directive is a hash password that can be simply generated using with the `caddy hash-password` command on the server where you have installed caddy
- Do not forget to set the `web.external-url` flag on Prometheus when using such a configuration, typically, if you are using systemd, you must add this flag to the `ExecStart` directive `--web.external-url=https://share.example.com/prometheus`

--------------------------------------------------------------------------------------------------------
_See Also_

[Running Cells Behind a reverse proxy](en/docs/cells/v2/run-cells-behind-proxy)
[Using Caddy version 1](en/docs/kb/deployment/running-cells-behind-caddy-reverse-proxy)
