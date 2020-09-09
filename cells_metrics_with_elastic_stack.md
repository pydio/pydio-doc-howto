References:

- https://www.elastic.co/guide/en/elasticsearch/reference/current/install-elasticsearch.html#install-elasticsearch
- https://www.elastic.co/guide/en/beats/filebeat/current/keystore.html

## With Docker

The docker method is the easiest, you do not need to install java or/and other dependencies.

A quick simple how-to to run elastic and kibana docker containers.
In this how-to it is assumed  that elasticsearch and kibana containers are both running on a the same server with public address or domain (to be accessed externally).

### Elasticsearch

See: https://www.elastic.co/guide/en/elasticsearch/reference/7.5/docker.html

To run a simple elasticsearch node:

`docker run -it -d --name elastic -p 9200:9200 -p 9300:9300 -e "discovery.type=single-node" docker.elastic.co/elasticsearch/elasticsearch:7.5.0`

To verify that the node is running you can use the following command `curl localhost:9200` or remotely `curl <ip-or-domain>:9200`.

The response will look like this :

```json
➜  ~ curl localhost:9200
{
  "name" : "be7ccc3cf76b",
  "cluster_name" : "docker-cluster",
  "cluster_uuid" : "sRBLMmdkRjWdWVyfC0qVoQ",
  "version" : {
    "number" : "7.5.0",
    "build_flavor" : "default",
    "build_type" : "docker",
    "build_hash" : "e9ccaed468e2fac2275a3761849cbee64b39519f",
    "build_date" : "2019-11-26T01:06:52.518245Z",
    "build_snapshot" : false,
    "lucene_version" : "8.3.0",
    "minimum_wire_compatibility_version" : "6.8.0",
    "minimum_index_compatibility_version" : "6.0.0-beta1"
  },
  "tagline" : "You Know, for Search"
}
```

*For production you should read the article on elastic to secure your installation with a password, [see](https://www.elastic.co/guide/en/elasticsearch/reference/current/configuring-security.html).*

### Kibana

See: https://www.elastic.co/guide/en/kibana/7.5/docker.html

To run kibana, use the following command (also make sure to provide either the name or container ID of your Elasticsearch container).

`docker run --link YOUR_ELASTICSEARCH_CONTAINER_NAME_OR_ID:elasticsearch -p 5601:5601 kibana:7.5.0`

For instance to run kibana in this case:

- `docker run --link elastic:elasticsearch -p 5601:5601 kibana:7.5.0` 
- `elastic` being the container name defined on the first step.

> For production Kibana should also be password protected [see](https://www.elastic.co/guide/en/kibana/7.5/using-kibana-with-security.html) .

### Filebeat

See: https://www.elastic.co/guide/en/beats/filebeat/current/filebeat-input-log.html#filebeat-input-log

Now lets add a filebeat which send the logs to elasticsearch and then displayed on Kibana.

[:image:cells/elastic-stack/kibana-add-logging.png]

1. If you do not have the **"Add Data to Kibana"** menu displayed click on the icon (top-left)

2. Then to add a log source **"Add log data"**

3. A list of sources will be displayed

4. Select **"Logstash logs"**

5. Make sure to select the right operating system (see screenshot below)

> For the examples below we are assuming that you are running on a debian based system.

[:image:cells/elastic-stack/filebeat-os-selection.png]

1. Follow the indicated steps
   1. **Download and install Filebeat**:  copy paste the commands which will install filebeat on your system
   2. **Edit the configuration**: edit with your prefered editor `/etc/filebeat/filebeat.yml` (depending on your case it might require admin access).
      Add Elasticsearch and Kibana hosts.

**Kibana:**

```yaml
#============================== Kibana =====================================

# Starting with Beats version 6.0.0, the dashboards are loaded via the Kibana API.
# This requires a Kibana endpoint configuration.
setup.kibana:

  # Kibana Host
  # Scheme and port can be left out and will be set to the default (http and 5601)
  # In case you specify and additional path, the scheme is required: http://localhost:5601/path
  # IPv6 addresses should always be defined as: https://[2001:db8::1]:5601
  host: "my-elastic-stack.net:5601"

  # Kibana Space ID
  # ID of the Kibana Space into which the dashboards should be loaded. By default,
  # the Default Space will be used.
  #space.id:
```

**Elasticsearch:**

```yaml
#-------------------------- Elasticsearch output ------------------------------
output.elasticsearch:
  # Array of hosts to connect to.
  hosts: ["my-elastic-stack.net:9200"]

  # Optional protocol and basic auth credentials.
  #protocol: "https"
  #username: "elastic"
  #password: "changeme"
```

*(if you are using TLS uncomment **protocol**, and if you are password protected uncomment **username** & **password**)*

**Targeted log file:** (the default comments were removed)

In our case we want to fetch the **cells.logs** (json format) and send them to elasticsearch.

Basic sample configuration that fetches the logs from **cells.log** file and send them to elastic.

Also make sure to enable the **Environment=PYDIO_LOGS_LEVEL=production**.
See [this article](https://pydio.com/en/docs/cells/v2/systems-logs) on how to enable the logs level.

```yaml
#=========================== Filebeat inputs =============================

filebeat.inputs:

- type: log
  enabled: true
  # Tags will help you filter the logs
  tags: ["cells", "my-cells.com"]
  # Paths that should be crawled and fetched. Glob based paths.
  paths:
    - /home/pydio/.config/pydio/cells/logs/cells.log
  json.keys_under_root: true
  json.overwrite_keys: false
  json.add_error_key: true
  json.ignore_decoding_error: true
```

### Displaying Cells logs on Kibana

On Kibana you can create your own dashboards and display only the relevant information that you wish to.

For this example we are only going to see the easy way to retrieve and save a dashboard.

On the left menu bar click on the compass icon (Discover).

You will be presented with an interface with a lots of fields, you must focus on the following (see screenshot below).

[:image:cells/elastic-stack/kibana-logs.png]


1. **Selected fields**: those are the information that you want to display.
2. **Available fields**: those are all the data fields that the filebeat retrieves (also adding additional fields such as timestamps)

In the following example we decided to display the log level, the logger (which service is sending the message) and the msg (message).

You can also filter by date, time (the top right bar), then you can save this preset to use it later for, **visualization** or to create a custom **dashboard**.
