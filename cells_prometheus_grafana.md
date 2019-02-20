# Enabling monitoring with Prometheus / Grafana


## Enabling metrics in Cells-Enterprise

Cells code is instrumented using Gauge, Counters, etc... We use an interface for various metrics systems (uber-go/tally), using a Noop
implementation by default. In Cells-Enterprise, the start flag `--enable_metrics` will register a Prometheus compatible collector instead of the Noop implementation, and achieve two things:

- Expose metrics as http on a random port under the /metrics endpoint on *each cells process* (only one per process, not per service)
- Via Cells registry, gather info about these exposed endpoints (for all processes), to list all these endpoints as Prometheus compatible targets.
- Dynamically update a JSON file under $HOME/.config/pydio/cells/services/pydio.grpc.metrics/prom_clients.json to be monitored by prometheus (see below).

In a distributed mode, you will have to run Prometheus on the same node where pydio.gateway.metrics is running.

## Installing Prometheus and Grafana

Download [Prometheus](https://prometheus.io/download/) and [Grafana](https://grafana.com/grafana/download) binaries for your platform or install using Docker.
Install them on the node where the pydio.gateway.metrics service will be running.

### Setup Prometheus

Edit `prometheus.yml` to add a new job in the scrape_config section, using the embeded `file_sd_configs` prometheus discovery mechanism.
This tool allows prometheus to watch for a specific file in json or yaml, and load scraping targets from there.
Yaml section should look like (the first job is set by default in prometheus to monitor itself):

```yaml
# A scrape configuration containing exactly one endpoint to scrape:
scrape_configs:
  # The job name is added as a label `job=<job_name>` to any timeseries scraped from this config.
  - job_name: 'prometheus'

    # metrics_path defaults to '/metrics'
    # scheme defaults to 'http'.

    static_configs:
    - targets: ['localhost:9090']

  - job_name: 'cells'

    file_sd_configs:
      - files:
        - {PATH_TO_CELLS_CONFIG_FOLDER}/services/pydio.gateway.metrics/prom_clients.json
```

You can configure  Prometheus to start on the port you wish, default is 9090.

With cells-enterprise running, you can check that all processes are correctly detected by checking [http://localhost:9090/targets](http://localhost:9090/targets).

<!-- ![Prom Targets](https://github.com/pydio/internal-tracker/raw/master/howtos/resources/Prometheus-Targets.png) -->
[:image-popup:/devops/Prometheus-Targets.png]

### Setup Grafana

Start grafana, will be accessible via port 3000 by default, you can change it by exporting `GF_SERVER_HTTP_PORT` to the value you want.

Set up admin / password and basic install steps for Grafana.

Add a Prometheus DataSource in Grafana pointing to the Prometheus port defined in the previous step.

## Grafana Dashboard

A simple dashboard has been published on the Grafana website. See https://grafana.com/dashboards/9817 
It can be simply imported with the following steps

- In the left menu, select Dashboard > Import
- In the "Grafana.com Dashboard" text field, enter the dashboard ID **9817**

The new dashboard should be available and show something like the image below.

<!-- ![Dashboard](https://github.com/pydio/internal-tracker/raw/master/howtos/resources/Grafana-Dashboard.png) -->
[:image-popup:/devops/Grafana-Dashboard.png]
