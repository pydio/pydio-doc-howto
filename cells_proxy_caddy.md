In this tutorial, we explain how to use a Caddy webserver as reverse proxy in front of a Pydio Cells installation.

[:image-popup:/devops/caddy-black.png]

Caddy is a lightweight webserver coded in Go language. It is embedded in Pydio Cells to serve as a gateway and expose the various services.
For more details and to download, please visit the [Caddy website](https://caddyserver.com).

In this example, we have a Pydio Cells running with following URLs, **on the same machine** as the Caddy reverse proxy:

- Internal (bind) URL: `share.example.com:8080`
- External (public) URL `https://share.example.com` - you can notice that the external URL is also the proxy URL.

One of the great feature of Caddy is the automatic generation of Let's Encrypt certificates by default. For the record the [official Caddy documentation states](https://caddyserver.com/docs/tls):

> Since HTTPS is enabled automatically, this directive [the `tls` directive] should only be used to deliberately override default settings. Use with care, if at all.

So if you want to use Let's Encrypt certificates, just leave the `tls` directive out.  
Assuming your Caddy server has control over both standard reserved HTTP ports (namely `80` and `443`) and write permissions on the Caddy working directory (usually on linux like OSs: `/home/<user that starts caddy>/.caddy`), you do not have to do nothing. The certificates will be generated and renewed _automagically_.

If you want to start the Caddy server in non- interractive mode, you might directly precise a contact email address, for instance: `tls tls@example.com`.

If you have your own certificate, you can use following directive: `tls /etc/certs/example.com.crt /etc/certs/example.com.key`

In a simple case, the caddy file of the reverse proxy looks then like this:

```conf
https://share.example.com {

  log stdout
  timeouts 0
  tls /home/pydio/cert/share.example.crt /home/pydio/cert/share.example.key
  
  proxy / 192.168.0.13:8080 {
    #transparent
    websocket
  }
}
```

For the record:

- **websocket**: enables realtime notifications. As Pydio Cells is highly asynchronous, this enables updating the UI once server jobs have been done (for instance, thumbnail generation after an upload, chats...)
- **transparent**: transparently forwards HTTP headers to the Caddy that is embedded in Cells. This directive is compulsory for the `webdav` feature of Cells to run properly. It has yet a drawback: Cells must then use same FQDN for both Public and Bind URL. In more complex setups, and if you know what your are doing, you might add a prior proxy directive that only forward transparently the `/dav` sub tree.


## Sync Client

By default the Sync Client should work out of the box if your Cells is also running behind a Caddy reverse proxy.

In the case that you wish to set a fixed port for the gRPC you must meet the following requirements:

- Cells with SSL using Self-signed
- GRPC port is set with the env variable `PYDIO_GRPC_EXTERNAL=33060`

```conf
https://share.example.com:33060 {
	tls /path/to/cert.pem /path/to/key.pem
	log stdout
	errors stdout

	proxy / https://192.168.0.13:33060 {
		websocket
		insecure_skip_verify
	}
}
```

--------------------------------------------------------------------------------------------------------
_See Also_

[Running Cells Behind a reverse proxy](en/docs/cells/v2/run-cells-behind-proxy)