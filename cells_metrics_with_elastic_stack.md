In this how to, we are going to see how you can retrieve logs, system informations and more by setting up the ELK stack along Cells.

### Elastic stack

"ELK" is the acronym for three open source projects: Elasticsearch, Logstash, and Kibana. Elasticsearch is a search and analytics engine. **Logstash** is a server‑side data processing pipeline that ingests data from multiple sources simultaneously, transforms it, and then sends it to a "stash" like **Elasticsearch**. **Kibana** lets users visualize data with charts and graphs in Elasticsearch.

Now with a new component the stack is most commonly referred as _Elastic Stack_.

There is a new component, **Beats** which is somewhat a lightweight _logstash_ that has multiple variants for instance, Filebeat which is focused on files, logs, Metricbeat is focused on sending system and service statistics.

### Install Kibana and Elasticsearch on the server that is going to process the data

Actually you could install them all in different machines for resources management but for our example Kibana and Elasticsearch are going to run on 1 server.

* Firstly, let's make sure that you have **Java 8** by running `java -version`, otherwise to install java 8, use the following commands (for this example openjdk is used but you can use Oracle's Java).

```bash
sudo apt install openjdk-8-jdk-headless
sudo apt install openjdk-8-jre-headless
```

then add the elastic repository,

>debian users might need this: `sudo apt-get install apt-transport-https`

```bash
wget -qO - https://artifacts.elastic.co/GPG-KEY-elasticsearch | sudo apt-key add -

echo "deb https://artifacts.elastic.co/packages/6.x/apt stable main" | sudo tee -a /etc/apt/sources.list.d/elastic-6.x.list

sudo apt update
```

* Now you are ready to install Kibana and Elasticsearch, let's proceed;

```bash
sudo apt install elasticsearch
sudo apt install kibana
```

Once both installations are complete some settings have to be changed in the configuration.

> You might require root rights to edit the config files.

For Kibana you must edit `/etc/kibana/kibana.yml`:

* Change:

 ```yaml
 server.host: "address"
 elasticsearch.hosts: ["http(s)://address(or domain name):port"].
```

Then for Elasticsearch edit `/etc/elasticsearch/elasticsearch.yml` and edit:

```yaml
`network.host: <address>`
```

to the address where your elastic is running.

after all of the modifications lets start the services,

```bash
sudo systemctl start elasticsearch
sudo systemctl start kibana
```

(if you want them to start automatically after a manual restart or if your server fails.)

```bash
sudo systemctl enable elasticsearch
sudo systemctl enable kibana
```

### Logstash and/or beats to fetch metrics

For this part you can use logstash as a standalone or use a lightweight version called **Beats**.
**Logstash** has builtin all type of metrics whereas **Beats** that depends on the type that you are going to use,of course you can use many beats at the same time.
For instance: 

* Filebeats: which focuses solely on fetching from a log file (like a `tail -f <file>`)

* Metricbeats: can retrieve metrics (such as CPU, RAM, ....) from services or even application such as one in go (you will have to add some code to let the beat retrieve metrics from your application).

* And many more.

### Basic configuration for cells

First set a filebeat on the machine running cells,

for debian/ubuntu machines use the following:

```bash
curl -L -O https://artifacts.elastic.co/downloads/beats/filebeat/filebeat-6.6.1-amd64.deb
sudo dpkg -i filebeat-6.6.1-amd64.deb
```

centos

```bash
curl -L -O https://artifacts.elastic.co/downloads/beats/filebeat/filebeat-6.6.1-x86_64.rpm
sudo rpm -vi filebeat-6.6.1-x86_64.rpm
```

once installed edit the `/etc/filebeat/filebeat.yml`, and add the following,

```yaml
#-------------------------- Elasticsearch output ------------------------------
    hosts: ["ip:port"]

    protocol: "http/s"
    username: "elastic"
    password: "changeme"

#============================== Kibana =====================================
    host: "ip:port"
```

now for the logs,

```yaml
#=========================== Filebeat inputs =============================
    enabled: true
    paths:
     - /home/cells/.config/pydio/cells/logs/cells.log

     json.keys_under_root: true
     json.overwrite_keys: false
     json.add_error_key: true
```

now lets test the config and start the beat,

```bash
sudo filebeat test config
sudo filebeat test output

sudo filebeat setup
sudo systemctl start filebeat
```
