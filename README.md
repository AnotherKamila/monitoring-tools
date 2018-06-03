# monitoring-tools
Kamila's monitoring toolkit. Revolves around [Prometheus](https://prometheus.io/).

## Quickly set up a monitoring server

_Note: I made this for [DataCenterLight](http://datacenterlight.ch). Check it out!_

[cdist](http://www.nico.schottelius.org/software/cdist/) is an interesting take on configuration management developed at [ungleich](http://ungleich.ch/). It is simple, versatile, and has no dependencies on the target system. In fact, it is a bit too simple compared to other, far more sophisticated configuration management solutions -- and that's precisely why I like it.

I use it for configuring all my monitoring servers (among other things). [Below](#example) you can find a cdist manifest snippet that comes from our real configuration at [DataCenterLight](http://datacenterlight.ch).

I have created several cdist types useful for installing a Prometheus monitoring server, as well as many exporters:

### High-level types for Prometheus and friends

- [`__prometheus_server`](https://github.com/ungleich/cdist/blob/master/cdist/conf/type/__prometheus_server/man.rst): Installs and configures Prometheus server. Yay!
- [`__prometheus_alertmanager`](https://github.com/ungleich/cdist/blob/master/cdist/conf/type/__prometheus_alertmanager/man.rst): Installs and configures Prometheus Alertmanager. Yay too!
- [`__grafana_dashboard`](https://github.com/ungleich/cdist/blob/master/cdist/conf/type/__grafana_dashboard/man.rst): Installs and configures [Grafana](https://grafana.com/). Yay three!

### Extras (use as needed)

#### Daemontools

[Daemontools](https://cr.yp.to/daemontools.html) are awesome. In line with the Unix philosophy, they do one thing and do it well. In this case they make sure that services run. Without the messy parts!

- [`__daemontools`](https://github.com/ungleich/cdist/blob/master/cdist/conf/type/__daemontools/man.rst): Installs daemontools or your favorite derivative, plus an init script in case your package doesn't come with one.
- [`__daemontools_service`](https://github.com/ungleich/cdist/blob/master/cdist/conf/type/__daemontools_service/man.rst): Creates a service directory. I gladly use this for everything that doesn't come with a good init script, such as some exporters. It's so simple!

#### Installing golang and go packages

Most distros provide a reasonably new version of Prometheus and the basic exporters (worst case you'll have to use the `--install-from-backports` with my `__prometheus_server`). This is the recommended way. However, if you need something more, these types let you install a recent version of golang and go packages. Note that I am currently not using these in production. I am listing them here anyway for completeness.

- [`__golang_from_vendor`](https://github.com/ungleich/cdist/blob/master/cdist/conf/type/__golang_from_vendor/man.rst): Installs golang from https://golang.org. This makes it easy to install any version of golang, including those that your old and obsolete distro doesn't have.
- [`__go_get`](https://github.com/ungleich/cdist/blob/master/cdist/conf/type/__go_get/man.rst): Easy installation of golang packages straight from online repos. Useful for installing a newer version of Prometheus or various exporters.

### Example

Put this in you cdist manifest, edit to your needs, and rejoice!

Monitoring server:
```sh
__prometheus_server \
    --install-from-backports \
    --config "$__manifest/files/prometheus.yml" \
    --retention-days 14 \
    --storage-path /data/prometheus \
    --rule-files "$__manifest/files/*.rules"
    
__prometheus_alertmanager \
    --install-from-backports \
    --config "$__manifest/files/alertmanager.yml" \
    --storage-path /data/alertmanager
    
__grafana_dashboard
```

### A cdist lightning-quick-start

```sh
git clone https://github.com/ungleich/cdist.git
cd cdist
$EDITOR cdist/conf/manifest/init
bin/cdist config -vv <hostname>
```

Read more at [cdist quickstart](http://www.nico.schottelius.org/software/cdist/man/latest/cdist-quickstart.html).

**Questions, ideas, and pull requests are welcome!**

## Exporters for monitoring software that doesn't have built-in instrumentation

- [ttn2prom](https://github.com/AnotherKamila/ttn2prom): Trivial Python webserver that accepts webhooks (i.e. HTTP requests), processes the data in them, and exports the stuff in the Prometheus format. Very easy to adapt. This version exports data from some LoRaWan temperature sensors through webhooks from [The Things Network](https://www.thethingsnetwork.org/).
- [opennebula exporter](https://github.com/AnotherKamila/opennebula-exporter): Exports useful metrics for [OpenNebula](https://opennebula.org/). For now it is an evil hack, but it will get better with time.
- [OpenNebula integration test](https://github.com/AnotherKamila/opennebula-exporter/tree/master/integration_test): tests VM creation and connectivity, and exports the results in a Prometheus-compatible format.

## Useful alerts collection

TBD
