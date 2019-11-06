
This simple how-to is based on the following article to help you run Cells Home with traefik reverse proxy in docker environment.

https://www.digitalocean.com/community/tutorials/how-to-use-traefik-as-a-reverse-proxy-for-docker-containers-on-ubuntu-18-04

## Prerequisites

- OS: Ubuntu 18
- Installed packages: docker, apache2-utils, docker-compose
  
## Apache utils for credential generation

Let's generate a hashed password to protect the dashboard behind a user/password.

Install the following package if you are missing it.

`sudo apt install apache2-utils `

To generate the password with htpasswd. Substitute `secure_password` with the password you'd like to use for the Traefik `admin`(you can also change the username) user:

` htpasswd -nb admin secure_password `

The output from the command will look like this:

Output: `admin:$apr1$ruca84Hq$mbjdMZBAG.KWn7vfN/SNK/ `

You'll use this output in the Traefik configuration file(traefik.toml) to set up a Basic Authentication for the Traefik health check and monitoring dashboard. Keep the entire output line inside a text file/note, you will need it later.

## Create new docker network

Create `web` internal network in docker.

` docker network create web `

This command will create a docker bridged network.

## Variable Environment

Before launching docker-compose, you will need to export some env variables.

```bash
export TRAEFIK_PORT=80

export TRAEFIK_DASHBOARD_PORT=81

export TRAEFIK_DOMAIN_NAME=your.domain

export TRAEFIK_NET_NAME=web
```

You can also put them inside  `.env` [Docker documentation](https://docs.docker.com/compose/env-file/)

## traefik.toml

Because traefik config file is in text format(TOML), it doesn't recognize variables environments. Please manually replace the strings (including the curly braces **{}**) by actual the values before running docker.

> Note: the encrypted credential is the string generated in the first step. In this instance this is the encrypted version of "P@ssw0rd".

```toml
defaultEntryPoints = ["http"]

[entryPoints]
  [entryPoints.dashboard]
    address = ":{TRAEFIK_PORT}"
    [entryPoints.dashboard.auth]
      [entryPoints.dashboard.auth.basic]
        users = ["admin:$apr1$x/i/X0fT$xEWW3okaBdNoZz/UFn7Dz/"]
  [entryPoints.http]
    address = ":{TRAEFIK_DASHBOARD_PORT}"    

[api]
entrypoint="dashboard"

[docker]
domain = "{TRAEFIK_DOMAIN_NAME}"
watch = true
network = "{TRAEFIK_NET_NAME}"

```

**Warning, when deployed in production you should not let the dashboard exposed to the outside the easiest way is to disable it by deleting the dashboard lines.**

## Docker compose

With the docker-compose file below, all the data of Cells Home will be stored in a "cells" folder inside the current folder.
Here is a look at the global tree:

```
├── cells
│   ├── cells-vault-key
│   ├── configs-versions.db
│   ├── data
│   ├── logs
│   ├── pydio.json
│   ├── services
│   └── static
├── docker-compose.yml
└── traefik.toml
└── mysql
```

Content of docker-compose.yml:

```yaml
version: "3"

networks:
  web:
    external: true
  internal:
    external: false

services:
  traefik:
    image: traefik:1.6-alpine
    container_name: traefik
    hostname: traefik
    domainname: ${DOMAIN_NAME}    
    restart: always
    ports:
      - "${TRAEFIK_PORT}:${TRAEFIK_PORT}"
      - "${TRAEFIK_DASHBOARD_PORT}:${TRAEFIK_DASHBOARD_PORT}"
    labels:
      - traefik.enable=true
      - traefik.backend=traefik
      - traefik.frontend.rule=Host:traefik.${DOMAIN_NAME}
      - traefik.port=${TRAEFIK_DASHBOARD_PORT}
      - traefik.docker.network=web
    volumes:
      - /var/run/docker.sock:/var/run/docker.sock:ro
      - $PWD/traefik.toml:/traefik.toml:ro      
    networks:
      - internal
      - web
  # Cells image with two named volumes for the static and for the data
  cells:
    image: pydio/cells:latest
    hostname: cells
    domainname: ${DOMAIN_NAME}
    restart: always
    volumes: ["./cells:/root/.config/pydio/cells"]
    environment:
      - CELLS_BIND=0.0.0.0:${TRAEFIK_PORT}
      - CELLS_EXTERNAL=cells.${DOMAIN_NAME}:${TRAEFIK_PORT}
      - CELLS_NO_SSL=1
    labels:
      - traefik.enable=true
      - traefik.backend=cells
      - traefik.frontend.rule=Host:cells.${DOMAIN_NAME}
      - traefik.docker.network=web
      - traefik.port=${TRAEFIK_PORT}
    networks: 
      - internal
      - web
    depends_on:
      - mysql
  # MySQL image with a default database cells and a dedicated user pydio
  mysql:
    image: mysql:5.7
    restart: always
    volumes: ["./mysql:/var/lib/mysql"]
    environment:
      MYSQL_ROOT_PASSWORD: P@ssw0rd
      MYSQL_DATABASE: cells
      MYSQL_USER: pydio
      MYSQL_PASSWORD: P@ssw0rd
    networks: 
      - internal
    labels:
      - traefik.enable=false
    command: [mysqld, --character-set-server=utf8mb4, --collation-server=utf8mb4_unicode_ci]
```

## DNS problems
To make use of the multiple routes that traefik generates you will most likely need to add dns entries with a valid domain.

Otherwise for quick testing, you can add the following lines inside your `/etc/hosts`
```
127.0.0.1 cells.your.domain
127.0.0.1 traefik.your.domain
```
_(127.0.0.1) can be changed for the ip of the server running traefik._

## Config Cells

When you finished to customize your **docker-compose.yaml**, you can start the services with  `docker-composer up -d`, (the -d will keep it in background) and then you can access the Cells web installer:

* With this url: _**http://cells.**`your-domain`_

Default values for the database configuration are:
- Server name: mysql
- Port: 3306
- Username: pydio
- Password: P@ssw0rd
- Database Name: cells

Mysql data will be stored inside "mysql" folder in current path (as a docker volume).

## Running Traefik With SSL


Here is an example URL to help you understand the values below `http://xyz.example.com`.

**traefik.toml**
```TOML
logLevel = "ERROR"
defaultEntryPoints = ["http", "https"]
insecureSkipVerify = true

[entryPoints]
  [entryPoints.dashboard]
    address = ":81"
    [entryPoints.dashboard.auth]
      [entryPoints.dashboard.auth.basic]
        users = ["admin:$apr1$x/i/X0fT$xEWW3okaBdNoZz/UFn7Dz/"]
  [entryPoints.http]
    address = ":80"   
    [entryPoints.http.redirect]
      entryPoint = "https"
  [entryPoints.https]
    address = ":443" 
    [entryPoints.https.tls]
      [[entryPoints.https.tls.certificates]]
      certFile = "/certs/cells.crt"
      keyFile = "/certs/cells.key" 
    
[retry]

[api]
entrypoint="dashboard"

[docker]
domain = "example.com"
watch = true
network = "web"

# [acme]
# email = "mail@gmail.com"
# storage = "acme.json"
# entryPoint = "https"
# onHostRule = true
# [acme.httpChallenge]
# entryPoint = "http"
```

In the following configuration the important parameters are:

### SSL Configuration

| Paramater name     | value            | what is it                                            |
| ------------------ | ---------------- | ----------------------------------------------------- |
| certFile           | /certs/cells.crt | path to the certificate file                          |
| keyFile            | /certs/cells.crt | path to the certificate key                           |
| insecureSkipVerify | true             | it will skip the certificate verification for traefik |

_Note: the insecureSkipVerify should be used when you have valid certificates_

In this example we use the custom certificates but if you wish to use the integrated let's encrypt you will have to modify the configuration.

#### Troubleshooting

To make use of SSL you must have it enabled on both ends, meaning Traefik and Cells.

### Access points

This is the list of the access points from the outside.

- port 80(will be redirected to 443 automatically) or 443 will give you access to the services

**Warning, when deployed in production you should not let the dashboard exposed to the outside the easiest way is to disable it**

**docker-compose.yaml**
```yaml
version: "3"

networks:
  web:
    external: true
  internal:
    external: false

services:
  traefik:
    image: traefik
    container_name: traefik
    restart: always
    ports:
      - "80:80"
      - "443:443"
    labels:
      - traefik.enable=true
      - traefik.backend=traefik
      - traefik.frontend.rule=Host:traefik.example.com
      - traefik.port=81
      - traefik.docker.network=web
    volumes:
      - /var/run/docker.sock:/var/run/docker.sock:ro
      - $PWD/traefik.toml:/traefik.toml:ro
      # - $PWD/acme.json:/acme.json:ro
      - $PWD/certs/:/certs/
    networks:
      - internal
      - web

  # Cells image with two named volumes for the static and for the data
  cells:
    image: pydio/cells:latest
    hostname: xyz
    domainname: example.com
    restart: always
    volumes: 
      # - $PWD/cells:/root/.config/pydio/cells
      - $PWD/certs/:/root/ssl/certs/
    environment:
      - CELLS_BIND=xyz.pydio.com:443
      - CELLS_EXTERNAL=https://xyz.example.com
      - CELLS_SSL_KEY_FILE=/root/ssl/certs/cells.key
      - CELLS_SSL_CERT_FILE=/root/ssl/certs/cells.crt
    labels:
      - traefik.enable=true
      - traefik.backend=xyz
      - traefik.frontend.rule=Host:xyz.pydio.com
      - traefik.docker.network=web
      - traefik.port=443
      - traefik.protocol=https
    networks: 
      - internal
      - web
    depends_on:
      - mysql

  # MySQL image with a default database cells and a dedicated user pydio
  mysql:
    image: mysql:latest
    restart: always
    environment:
      MYSQL_ROOT_PASSWORD: root
      MYSQL_DATABASE: cells
      MYSQL_USER: pydio
      MYSQL_PASSWORD: P@ssw0rd
    networks: 
      - internal
    labels:
      - traefik.enable=false
    command: [mysqld, --character-set-server=utf8mb4, --collation-server=utf8mb4_unicode_ci]
    logging:
      driver: none
```
_the acme.json volume is disabled but if you are going to use lets encrypt with Traefik make sure to uncomment it_

For the SSL configuration on Cells side, take a look at the [following page](https://github.com/pydio/cells/tree/master/tools/docker).

To access the traefik dashboard (if you do not have the domain name) you can add an entry inside your /etc/hosts such as:

```conf
x.x.x.x traefik.example.com
```

To access cells you can directly go to `http(s)://xyz.example.com`

## Encountered issues
In the case of Traefik (SSL ON) and Cells (SSL OFF) there is a bit of an issue because Traefik does HTTPS requests on cells.

* Looking for a backend redirect, seems that when you try to login it requests Cells with the following url `https://xxxx/auth/dex/token` but cells expects `http://xxxx/auth/dex/token`.

--------------------------------------------------------------------------------------------------------
_See Also_

[Running Cells Behind a reverse proxy](en/docs/cells/v2/run-cells-behind-proxy)