This tutorial quickly introduce how to set up and run [Prometheus](https://prometheus.io) and [Grafana](https://grafana.com) to gather and display metrics for your Pydio Cells instance.

Please note that it is an Enterprise Distribution only feature.

### Enabling metrics

The backend code is instrumented with gauges and counters. We use an interface for various metrics systems (uber-go/tally), using a no-op implementation by default.

Using the `--enable_metrics` flag upon startup registers a Prometheus compatible collector instead of the no-op implementation, and achieve three things:

- Expose metrics as HTTP on a random port under the `/metrics` endpoint on **each Cells process** (only one per process, not per service)
- Gather info about these exposed endpoints for all processes via the registry and thus list all these endpoints as Prometheus compatible targets.
- Dynamically update a JSON file under `<cells root dir>/services/pydio.grpc.metrics/prom_clients.json`. This file is watched by Prometheus so that the endpoints can be dynamically discovered (see below).

In a distributed mode, you must run Prometheus on the same node where `pydio.gateway.metrics` is running.

### Installation

You can download [Prometheus](https://prometheus.io/download/) and [Grafana](https://grafana.com/grafana/download) binaries for your platform or install them using Docker.
Remember that you must install them on the same node where the `pydio.gateway.metrics` service is running.

### Setup

#### Prometheus

Edit `prometheus.yml` to add a new job in the `scrape_config section`, using the embeded `file_sd_configs` Prometheus discovery mechanism.
This tool allows Prometheus to watch for a specific JSon (or YAML) file and thus know where to load scraping targets.

YAML section should look like (the first job is set by default by Prometheus to monitor itself):

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

You can configure Prometheus to start on the port you wish, default is 9090.

With Cells ED running, you can check that all processes are correctly detected by visiting [http://localhost:9090/targets](http://localhost:9090/targets).

[:image-popup:/devops/Prometheus-Targets.png]

#### Grafana

First Start Grafana. By default, it is accessible at port 3000. You define your own specific port using the `GF_SERVER_HTTP_PORT` environement variable.

Choose an admin/password and perform basic install steps.

Add a Prometheus DataSource in Grafana pointing to the Prometheus port defined in the previous step.

### The Grafana Dashboard

A [simple dashboard has been published on the Grafana website](https://grafana.com/dashboards/9817).  
It can be simply imported with the Graphana UI by following steps:

- In the left menu, select Dashboard > Import
- In the "Grafana.com Dashboard" text field, enter the dashboard ID **9817**

The new dashboard should be available and show something like the image below.

[:image-popup:/devops/Grafana-Dashboard.png]
