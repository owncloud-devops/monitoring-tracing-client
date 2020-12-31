# Monitoring and tracing client

This monitoring and tracing clients are deployed on our internal oCIS testing instances, as well as our internal testing infrastructure.

Have a look at:

- Grafana: [grafana.infra.owncloud.works](https://grafana.infra.owncloud.works)
- Prometheus: [prometheus.infra.owncloud.works](https://prometheus.infra.owncloud.works)
- Jaeger Query: [jaeger.infra.owncloud.works](https://jaeger.infra.owncloud.works)

## Overview

For a wider overview on what we try to achieve, see [here](https://owncloud.github.io/ocis/deployment/monitoring-tracing/).

The following overview will mainly focus on the monitoring & tracing client.

### Monitoring

This deployment provides Telegraf, wich is used for collecting host and docker metrics, as well as oCIS metrics. These metrics are made available on a metrics endpoint and can be scraped by Prometheus. If you don't use Prometheus as your monitoring plattform like we propose, Telegraf surely provides an output for your plattform too. Telegraf is also configured to add Tags to the collected metrics so that you can properly query them and identify, which oCIS instance and wich host, if it's a multi host oCIS deployment, the metrics originate from.
In order to make this monitoring client reusable, the config is split up in logical parts and allows you to select the oCIS deployment specific configuration that fits your needs. The configuration differs from deployment to deployment because the oCIS metrics endpoint urls are different eg. if you run oCIS in one container or every extension in its own container.

### Tracing

This deployment provides Jager agent, which acts as kind of relay between the traces generating oCIS extensions and our tracing server. Jaeger agent receives the traces send by oCIS (see the needed configuration for oCIS ([example](https://github.com/owncloud/ocis/blob/master/deployments/examples/ocis_traefik/monitoring_tracing/docker-compose-additions.yml)). The agent adds some process tags, that you later can filter and query traces properly by oCIS instances and hosts and then forwards the traces to the tracing server.

## Available oCIS specific Telegraf configurations

`TELEGRAF_SPECIFIC_CONFIG` enables you to select a specific configuration from `./config/telegraf_specific_config` by setting it to the filename without the file extension `.conf`.

Following specific configurations are available:

### none (`TELEGRAF_SPECIFIC_CONFIG=none`, default)

This provides an empty configuration file. This means no specific configuration is used.

### oCIS single container (`TELEGRAF_SPECIFIC_CONFIG=ocis_single_container`)

This provides an specific configuration if you run oCIS in a single container. This needs to be used for following deployment examples:

- [oCIS with Traefik](https://owncloud.github.io/ocis/deployment/ocis_traefik/)
- [oCIS with Keycloak](https://owncloud.github.io/ocis/deployment/ocis_keycloak/)
- [ownCloud 10 with ownCloud Web](https://owncloud.github.io/ocis/deployment/owncloud10_with_oc_web/)

## Deploy with oCIS

1. deploy an oCIS server following our [deploy examples](https://owncloud.github.io/ocis/deployment/#deployments-scenarios-and-examples).
1. uncomment following line (usually at the end) in the `.env` file of your example deployment: `COMPOSE_FILE=docker-compose.yml:monitoring_tracing/docker-compose-additions.yml` and run again `docker-compose up -d`. Docker-compose should ask you to manually create a network. Please do so (command needed to run is displayed in the message). After that you again need to run `docker-compose up -d`. Your oCIS deployment is now ready for being used with the monitoring and tracing client.
1. clone this repository
1. create `.env` file by running `cp .env.dist .env` inside the root of this repository
1. configure as needed inside the `.env` file
1. make sure your selected `TELEMETRY_SERVE_DOMAIN` domain exists and resolves to the ip of your current server (or use a wildcard dns entry)
1. run `docker-compose up -d`
1. configure Prometheus on the monitoring server to scrape your current deployment by adding the domain you configured in `TELEMETRY_SERVE_DOMAIN` to the Prometheus scrape list ([see monitoring server FAQ](https://github.com/owncloud-devops/monitoring-tracing-server#how-can-i-set-up-my-ocis-deployment--server-being-scraped)).
