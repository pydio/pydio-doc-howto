In this tutorial, we explain how you can configure a reverse proxy for your Cells Docker container and what settings are the most important to change.


## Run your Cells docker container behind a Traefik reverse-proxy using SSL

For the purpose of this deployment, we will use docker-compose as it offers a simple way to visualize the full stack deployment under a single file. The reverse proxy we're going to use is Traefik, a very efficient Go reverse proxy designed to perfectly integrate with Docker and Kubernetes.

This configuration file is valid for a cells service accessible via https://cells.domain.tld. Please edit the `<cell.domain.tld>` value to the actual url you will use to reach the service. As the point of this documentation is to setup a reverse proxy with SSL, we did not mention volumes mountpoints specific to a proper cells configuration.

### docker-compose.yml

```yaml
version: '3'
services:

  reverse:
    image: traefik:v2.0
    ports:
      - "443:443"
    volumes:
      - /var/run/docker.sock:/var/run/docker.sock:ro
      - ./letsencrypt:/letsencrypt
    command:
      - "--providers.docker=true"
      - "--entrypoints.websecure.address=:443"
      - "--certificatesresolvers.mytlschallenge.acme.tlschallenge=true"
      - "--certificatesresolvers.mytlschallenge.acme.email=<user@domain.tld>"
      - "--certificatesresolvers.mytlschallenge.acme.storage=/letsencrypt/acme.json"

  cells:
    image: pydio/cells:1.6.1
    ports:
      - "8000:80"
    environment:
      - CELLS_BIND=cells.domain.tld:80
      - CELLS_EXTERNAL=https://cells.domain.tld
      - CELLS_NO_SSL=1
    labels:
      - "traefik.http.routers.cells.rule=Host(`<cells.domain.tld>`)"
      - "traefik.http.routers.cells.entrypoints=websecure"
      - "traefik.http.routers.cells.tls.certresolver=mytlschallenge"
    #extra_hosts:
    #  - "<cells.domain.tld>:<ip_to_access_domain>"
```

### What's in this file

**reverse**

- ports : our reverse proxy will listen incoming requests on port 443
- volumes:
  - in order to listen to dockers event, traefik containers needs to access our machine docker socket
  - we also setup a permanent volume so the SSL certificates are not regenerated every time we relaunch our container
- command:
  - provider: we link traefik routing system to docker
  - entrypoints: we name `websecure` our link to port 443
  - certificateresolvers:
  - we name mytlschallenge the certificate issuing process, set it up to `tlschallenge` - meaning that Let's encrypt will
  - check the domain is ours by connecting to our server on port 443 -, provide a valid email to complete the certificate issuing, and define the path in the container where the certificate will be stored / found
- in case the machine is not accessible from outside your network, you might use `dnschallenge` instead of `tlschallenge`
  
**cells**

- ports: our cells instance will be configured to listen on port 80, and the container will expose port 8000 to avoid conflicts with the ports our reverse proxy listens on
- environment:
- CELLS_BIND : address where the application http server is bound to - our cell container will accept requests addressed to cells.domain.tld, in plain http
- CELLS_EXTERNAL : url the end user will use to connect to the application - in our case, by typing https://cells.domain.tld in a web browser
- CELLS_NO_SSL : as Traefik manages the SSL certificate, it is the point of termination of the SSL connection, and communication between cells and traefik are done in plain HTTP, so we deactivate SSL on cells side
- labels:
  - we name `cells` the configuration we're going to use, and specify it will be used for requests searching to reach the host `cells.domain.tld`
  - we specify that the requests that will be served are the one reaching the entrypoint `websecure` - the one we defined in traefik, listening on port 443
  - we define that the SSL communication for this host is using the resolver `mytlschallenge` we configured in traefik.
  - extra_hosts: this option is used exclusively if there is no DNS entry accessible by the cells container matching the domain name, as cells must be able to access itself via the CELLS_BIND address. extra_hosts directive simply add a new entry to the /etc/hosts file of the container.