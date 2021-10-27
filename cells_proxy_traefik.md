In this tutorial, we explain how to configure a reverse proxy for your Cells Docker container and what settings are the most important to change.

## Run your Cells docker container behind a Traefik reverse-proxy using SSL

Traefik is a very efficient Go reverse proxy designed to perfectly integrate with Docker and Kubernetes.  

For the purpose of this deployment, we use `docker-compose`: it offers a simple way to visualize the full stack deployment under a single file.  

### Quick start on localhost

```yaml
version: "3.7"

services:
  reverse:
    image: traefik:2.5
    ports: ["80:80"]
    volumes:
      - /var/run/docker.sock:/var/run/docker.sock:ro
    command:
      - --providers.docker
      - --api
      - --entrypoints.web.address=:80
    labels:
      - traefik.http.routers.reverse.service=api@internal
      - traefik.http.routers.reverse.rule=PathPrefix(`/api`)||PathPrefix(`/dashboard`)
      - traefik.http.routers.reverse.entrypoints=web

  cells:
    image: pydio/cells:latest
    expose: [8080]
    environment:
      - CELLS_BIND=0.0.0.0:8080
      - CELLS_EXTERNAL=http://localhost
      - CELLS_NO_TLS=1
    labels:
      - traefik.http.routers.cells.rule=Host(`localhost`)
      - traefik.http.routers.cells.entrypoints=web

  mysql:
    image: mysql:8
    restart: unless-stopped
    environment: 
      - MYSQL_ROOT_PASSWORD=cells
      - MYSQL_DATABASE=cells
      - MYSQL_USER=pydio
      - MYSQL_PASSWORD=cells
    cap_add: 
      - SYS_NICE 
    command: 
      - mysqld
      - --character-set-server=utf8mb4
      - --collation-server=utf8mb4_unicode_ci
      - --default_authentication_plugin=mysql_native_password
```

You can then access the Cells installer on http://localhost and the Traefik dashboard at http://localhost/dashboard/ (the trailing slash is important or you get a 404 - Page not found exception).

### For live environments

This file sets up a production ready Cells ecosystem with opiniated configuration.

It exposes various services via HTTPS (provided by Let's Encrypt) under $PUBLIC_FQDN:

- The Pydio Cells server at the root
- The Traefik dashboard  under `/dashboard/`
- Promotheus metrics under `/prometheus`

Both Traefik and Prometheus endpoints are protected by basic authentication with login/password: admin/admin

> Do not forget to prepare / reset acme.json file when changing the Let's Encrypt configiguration,typically at first start or when switching from staging to prod CA server: `touch acme.json; chmod 600 acme.json`

> Note that we do not do the TLS termination of the requests for Cells at the Traefik level in order to enable gRPC communication between the End-User and the Cells application (necessary for the Sync client typically). If you do not need that, refer to the example above to perform TLS termination at the reverse proxy layer and thus have a simpler and easier to maintain configuration.

```yaml
version: "3.7"

volumes:
    cells_working_dir: {}
    cells_data: {}
    mysql_data: {}
    prometheus_data: {}

services:
  # Traefik as reverse proxy with dashboard enabled
  reverse:
    image: traefik:2.5
    restart: unless-stopped
    command:
      # More logs when debugging
      #- --log.level=DEBUG
      # Tell traefik to watch docker events for hot reload
      - --providers.docker
      - --providers.docker.exposedbydefault=false
      # Enable the dashboard on https
      - --api
      # Listen default HTTP ports
      - --entrypoints.web.address=:80
      - --entrypoints.websecure.address=:443
      # Trust all certificates that are exposed by the downstream services, this is 
      # necessary to accept the default self-signed cert exposed by the Cells service.
      - --serverstransport.insecureskipverify=true
      # Automatic generation of certificate with Let's Encrypt
      - --certificatesresolvers.leresolver.acme.email=${TLS_MAIL_ADDRESS}
      - --certificatesresolvers.leresolver.acme.storage=/acme.json
      - --certificatesresolvers.leresolver.acme.tlschallenge=true
      # Insure to use staging CA server while testing to avoid being black listed => generated cert is un-trusted by browsers. Comment out once everything is correctly configured.
      - --certificatesresolvers.leresolver.acme.caserver=https://acme-staging-v02.api.letsencrypt.org/directory
    ports:
      - 80:80
      - 443:443
    volumes:
      - /var/run/docker.sock:/var/run/docker.sock:ro
      # Persists certificate locally, otherwise we will recreate new ones at each restarts and quickly hit limits.
      # Remember to flush the file if you want to switch from staging CA server to prod
      - ./acme.json:/acme.json
    labels:
      # Redirect HTTP traffic to HTTPS
      - traefik.http.routers.redirs.rule=hostregexp(`{host:.+}`)
      - traefik.http.routers.redirs.entrypoints=web
      - traefik.http.routers.redirs.middlewares=redirect-to-https
      - traefik.http.middlewares.redirect-to-https.redirectscheme.scheme=https
      # Expose the traefik dashboard on the reserved sub path, TLS is provided by the Let's Encrypt cert provider.
      - traefik.enable=true
      - traefik.http.routers.reverse.service=api@internal
      - traefik.http.routers.reverse.rule=PathPrefix(`/api`)||PathPrefix(`/dashboard`)
      - traefik.http.routers.reverse.entrypoints=websecure
      - traefik.http.routers.reverse.tls.certresolver=leresolver
      # Protect dashboard with simple auth => log with admin / admin for this example
      - traefik.http.routers.reverse.middlewares=admin
      # Password is generated with `htpasswd -nb admin admin` beware to escape all '$' replacing them by '$$'
      - "traefik.http.middlewares.admin.basicauth.users=admin:$$apr1$$KnKvATsN$$L8K.P.maCu4zR/rVzD8h0/"

  # DB backend
  mysql:
    image: mysql:8
    restart: unless-stopped
    environment: 
      - MYSQL_ROOT_PASSWORD=${MYSQL_ROOT_PASSWORD}
      - MYSQL_DATABASE=cells
      - MYSQL_USER=${MYSQL_CELLS_USER_LOGIN}
      - MYSQL_PASSWORD=${MYSQL_CELLS_USER_PASSWORD}
    cap_add: 
      - SYS_NICE 
    command: 
      - mysqld
      - --character-set-server=utf8mb4
      - --collation-server=utf8mb4_unicode_ci
      - --default_authentication_plugin=mysql_native_password


# Pydio Cells app
  cells:
    image: ${CELLS_DOCKER_IMAGE}
    restart: unless-stopped
    # Not compulsory but it eases some of the maintenance operations. 
    hostname: cells
    expose:
      - 443
    volumes:
      - cells_working_dir:/var/cells
      - cells_data:/data
      - ./metrics:/var/cells/services/pydio.gateway.metrics
    environment:
      - CELLS_WORKING_DIR=/var/cells
      - CELLS_DATA=/data
      - CELLS_BIND=${PUBLIC_FQDN}:443
      - CELLS_EXTERNAL=https://${PUBLIC_FQDN}
      - CELLS_ENABLE_METRICS=true
    labels:
      - traefik.enable=true
      - traefik.http.services.cells.loadbalancer.server.scheme=https
      - traefik.http.routers.cells.rule=Host(`${PUBLIC_FQDN}`)
      - traefik.http.routers.cells.entrypoints=websecure
      - traefik.http.routers.cells.tls=true
      - traefik.http.routers.cells.tls.certresolver=leresolver
    depends_on:
      - mysql

  # Prometheus to expose metrics
  prometheus:
    image: prom/prometheus
    expose:
      - 9090
    volumes:
      - prometheus_data:/prometheus
      - ./prom.yml:/etc/prometheus/prometheus.yml
      - ./metrics:/etc/prometheus/watch:ro
    command:
      - --config.file=/etc/prometheus/prometheus.yml
      - --storage.tsdb.path=/prometheus
      - --storage.tsdb.retention.time=30d
      - --web.external-url=https://${PUBLIC_FQDN}/prometheus
      - --web.listen-address=:9090
    labels:
      # Expose the metrics on the reserved sub path, TLS is provided by the Let's Encrypt cert provider.
      - traefik.enable=true
      - traefik.http.routers.prometheus.rule=Host(`${PUBLIC_FQDN}`)&&PathPrefix(`/prometheus`)
      - traefik.http.routers.prometheus.entrypoints=websecure
      - traefik.http.services.prometheus.loadbalancer.server.port=9090
      - traefik.http.routers.prometheus.tls.certresolver=leresolver
      # Protect metrics entry point with simple auth
      - traefik.http.routers.prometheus.middlewares=admin
```

In order to use the above file, you must add a `.env` and a `prom.yml` file in the same folder with following content:

```properties
CELLS_DOCKER_IMAGE=pydio/cells-enterprise:latest
PUBLIC_FQDN=share.example.com
TLS_MAIL_ADDRESS=tls@example.com
MYSQL_ROOT_PASSWORD=cells
MYSQL_CELLS_USER_LOGIN=pydio
MYSQL_CELLS_USER_PASSWORD=cells
```

```yaml
# Simple prometheus configuration that watches itself and the entry points declared by cells:
scrape_configs:
  # The job name is added as a label `job=<job_name>` to any timeseries scraped from this config.
  - job_name: 'prometheus'
    metrics_path: '/prometheus/metrics'
    # scheme defaults to 'http'.
    static_configs:
    - targets: ['prometheus:9090']
  - job_name: 'cells'
    file_sd_configs:
      - files:
        - /etc/prometheus/watch/prom_clients.json
```


### A few more hints about the docker-compose file

For `reverse`:

- `ports`: our reverse proxy listens incoming requests on port 443
- `volumes`:
  - in order to listen to docker events, traefik containers needs to access our machine docker socket
  - we also setup a permanent volume so the SSL certificates are not regenerated every time we relaunch our container
- `command`:
  - `provider`: we link traefik routing system to docker
  - `entrypoints`: we name `websecure` our link to port 443
  - `certificateresolvers`:
    - we name `leresolver` the certificate issuing process, set it up to `tlschallenge` - meaning that Let's encrypt checks the domain is ours by connecting to our server on port 443 -, provide a valid email to complete the certificate issuing, and define the path in the container where the certificate is stored / found
    - in case the machine is not accessible from outside your network, you might use `dnschallenge` instead of `tlschallenge`
  
For `cells`:

- `expose`: our cells instance is configured to listen on port 443, but this port is **not** exposed outside of docker.
- `environment`:
  - `CELLS_BIND`: address where the application http server is bound to - our cell container accepts requests addressed to `$PUBLIC_FQDN:443`, with a self signed certificate that is managed by Cells
  - `CELLS_EXTERNAL`: url the end user uses to connect to the application - in our case, by typing `https://$PUBLIC_FQDN` in a web browser
- `labels`:
  - we name `cells` the configuration we are going to use, and specify it is to be used for requests searching to reach the host.
  - we specify that the requests that are served are the ones reaching the entrypoint `websecure` - that we have defined in traefik, listening on port 443
  - we define that the SSL communication for this host is using the resolver `leresolver` we configured in traefik
  - Note: the `extra_hosts` option can be used if there is no DNS entry accessible by the Cells container matching the FQDN: and if you are using an old version of Cells (prior to 2.1). Cells must be able to access itself via the `CELLS_BIND` address. `extra_hosts` directive simply add a new entry to the `/etc/hosts` file of the container.
