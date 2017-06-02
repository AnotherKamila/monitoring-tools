# monitoring-tools
Kamila's monitoring toolkit. Revolves around [Prometheus](https://prometheus.io/).

## Quickly set up a monitoring server

_Note: I made this for [DataCenterLight](http://datacenterlight.ch). Check it out!_

[cdist](http://www.nico.schottelius.org/software/cdist/) is an interesting take on configuration management developed at [ungleich](http://ungleich.ch/). It is simple, versatile, and has no dependencies on the target system. In fact, it is a bit too simple compared to other, far more sophisticated configuration management solutions -- and that's precisely why I like it.

I use it for configuring all my monitoring servers (among other things). [Below](#example) you can find a cdist manifest snippet that comes from our real configuration at [DataCenterLight](http://datacenterlight.ch).

I have created several cdist types useful for installing a Prometheus monitoring server, as well as many exporters:

### Installing golang and go packages

- [`__golang_from_vendor`](https://github.com/ungleich/cdist/blob/master/cdist/conf/type/__golang_from_vendor/man.rst): Installs golang from https://golang.org. This makes it easy to install any version of golang, including those that your old and obsolete distro doesn't have.
- [`__go_get`](https://github.com/ungleich/cdist/blob/master/cdist/conf/type/__go_get/man.rst): Easy installation of golang packages straight from online repos. Useful for installing Prometheus itself or the various exporters.

### Daemontools

[Daemontools](https://cr.yp.to/daemontools.html) are awesome. In line with the Unix philosophy, they do one thing and do it well. In this case they make sure that services run. Without the messy parts!

- [`__daemontools`](https://github.com/ungleich/cdist/blob/master/cdist/conf/type/__daemontools/man.rst): Installs daemontools or your favorite derivative, plus an init script in case your package doesn't come with one.
- [`__daemontools_service`](https://github.com/ungleich/cdist/blob/master/cdist/conf/type/__daemontools_service/man.rst): Creates a service directory. I gladly use this for everything that doesn't come with a good init script, such as Prometheus and exporters. It's so simple!

### High-level types for Prometheus and friends

_Note: these are not merged in upstream yet. I will update with links when they are!_

- `__prometheus_server`: Installs and configures Prometheus server. Yay!
- `__prometheus_alertmanager`: Install and configures Prometheus Alertmanager. Yay too!
- `__grafana_dashboard`: Install and configures [Grafana](https://grafana.com/). Yay three!

### Example

Put this in you cdist manifest, edit to your needs, and rejoice!

Monitoring server:
```sh
PROMPORT=9090
ALERTPORT=9093

__daemontools
__golang_from_vendor --version 1.8.1  # required for prometheus and many exporters

require="__daemontools __golang_from_vendor" __prometheus_server \
	--config "$__type/files/prometheus.yml" \
	--retention-days 14 \
	--storage-path /data/prometheus \
	--listen-address "[::]:$PROMPORT" \
	--rule-files "$__type/files/*.rules" \
	--alertmanager-url "http://monitoring1.node.consul:$ALERTPORT,http://monitoring2.node.consul:$ALERTPORT"

require="__daemontools __golang_from_vendor" __prometheus_alertmanager \
	--config "$__type/files/alertmanager.yml" \
	--storage-path /data/prometheus \
	--listen-address "[::]:$ALERTPORT"
```

Node exporter:
```sh
__daemontools --from-package daemontools
__golang_from_vendor --version 1.8.1  # required for prometheus and many exporters

require="__golang_from_vendor" __go_get github.com/prometheus/node_exporter
__daemontools_service node_exporter --run "/opt/gocode/bin/node_exporter"
```

_Note: these types are currently available in cdist's `master` branch, but not yet in a release._

### A cdist lightning-quick-start

```sh
git clone https://github.com/ungleich/cdist.git
cd cdist
$EDITOR cdist/conf/manifest/init
bin/cdist config -vv <hostname>
```

Read more at [cdist quickstart](http://www.nico.schottelius.org/software/cdist/man/latest/cdist-quickstart.html).

**And don't forget: Questions, ideas, and pull requests are welcome!**

## Exporters for monitoring software that doesn't have built-in instrumentation

- [opennebula exporter](https://github.com/AnotherKamila/opennebula-exporter): Exports useful metrics for [OpenNebula](https://opennebula.org/). For now it is an evil hack, but it will get better with time.
- [OpenNebula integration test](https://github.com/AnotherKamila/opennebula-exporter/tree/master/integration_test): tests VM creation and connectivity, and exports the results in a Prometheus-compatible format.

## Useful alerts collection

TBD
