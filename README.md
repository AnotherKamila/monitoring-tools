# monitoring-tools
Kamila's monitoring toolkit. Revolves around [Prometheus](https://prometheus.io/).

*Note*: All of this is a WIP and not everything I mention here is already online. I will be publishing a lot of stuff in the upcoming weeks.

## `cdist` types for easy installation

[cdist](http://www.nico.schottelius.org/software/cdist/) is an interesting take on configuration management. I have created several types that aid in installing a Prometheus monitoring server and exporters:

- `__golang_from_vendor`: install any version of golang from https://golang.org
- `__go_get`: easy installation of golang packages straight from online repos
- `__daemontools`, `__daemontools_service`: easily set up things to run and keep running without messy init scripts
- `__prometheus_server`: install and configure Prometheus server
- `__prometheus_alertmanager`: install and configure Prometheus Alertmanager
- `__grafana_dashboard`: install and configure [Grafana](https://grafana.com/)

### Example:

Server:
```sh
PROMPORT=9090
ALERTPORT=9093

__daemontools --from-package daemontools
__golang_from_vendor --version 1.8.1  # required for prometheus and many exporters

##### Prometheus server: scrapes and evaluates data #########################

require="__golang_from_vendor" __prometheus_server \
	--config "$__type/files/prometheus.yml" \
	--retention-days 14 \
	--storage-path /data/prometheus \
	--listen-address "[::]:$PROMPORT" \
	--rule-files "$__type/files/*.rules" \
	--alertmanager-url "http://monitoring1.node.consul:$ALERTPORT,http://monitoring2.node.consul:$ALERTPORT" \
	--add-daemontools-service

__consul_service prometheus --port $PROMPORT --check-http "http://localhost:$PORT/metrics" --check-interval 10s

##### Alertmanager: routes alerts ###########################################

require="__golang_from_vendor" __prometheus_alertmanager \
	--config "$__type/files/alertmanager.yml" \
	--storage-path /data/prometheus \
	--listen-address "[::]:$PROMPORT" \
	--add-daemontools-service

__consul_service alertmanager --port $ALERTPORT --check-http "http://localhost:$ALERTPORT/metrics" --check-interval 10s
```

Node:
```sh
PORT=9100

__daemontools --from-package daemontools
__golang_from_vendor --version 1.8.1  # required for prometheus and many exporters

export GOBIN=/opt/gocode/bin  # where to find go binaries

TEXTFILES=/service/node_exporter/textfiles  # path for the textfiles collector
__directory $TEXTFILES --mode 777

require="__golang_from_vendor" __go_get github.com/prometheus/node_exporter
__daemontools_service node_exporter --run "$GOBIN/node_exporter -web.listen-address :$PORT -collector.textfile.directory=$TEXTFILES"
__consul_service node-exporter --port $PORT --check-http "http://localhost:$PORT/metrics" --check-interval 10s
```

## Exporters for monitoring software that doesn't have built-in instrumentation

- [opennebula exporter](https://github.com/AnotherKamila/opennebula-exporter): Exports useful metrics for [OpenNebula](https://opennebula.org/). For now it is an evil hack, but it will get better with time.
- [OpenNebula integration test](https://github.com/AnotherKamila/opennebula-exporter/tree/master/integration_test): tests VM creation and connectivity, and exports the results in a Prometheus-compatible format.

## Useful alerts collection

TBD
