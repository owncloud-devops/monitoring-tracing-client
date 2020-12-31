# Monitoring and tracing client

This monitoring and tracing clients are deployed on our internal oCIS testing instances as well as our internal testing infrastructure.

Have a look at:

- Grafana: [grafana.infra.owncloud.works](https://grafana.infra.owncloud.works)
- Prometheus: [prometheus.infra.owncloud.works](https://prometheus.infra.owncloud.works)
- Jaeger Query: [jaeger.infra.owncloud.works](https://jaeger.infra.owncloud.works)

## Overview

For a wider overview on what we try to achieve, have a look [here](https://owncloud.github.io/ocis/deployment/monitoring-tracing/).

The following overview focuses mainly on the monitoring & tracing client.

### Monitoring

This deployment provides Telegraf, which is used for collecting host and docker metrics as well as oCIS metrics. These metrics are available on a metrics endpoint and can be scraped by Prometheus. If you don't use Prometheus as your monitoring platform, Telegraf will most likely provide an output for your platform too. Telegraf is also configured to add tags to the collected metrics so that you can properly query them. It also helps you to identify the origin of metrics, e.g. the oCIS instance and host, in case it is a multi host oCIS deployment.
In order to make this monitoring client reusable, the configuration is divided in logical parts. This also allows you to select a specific configuration that fits your deployment. There are different configurations e.g. because the oCIS metrics endpoint URLs will be different if you run oCIS in one container or every extension in its own container.

### Tracing

This deployment provides Jaeger agent, which acts as relay between oCIS extensions, which are generating traces, and our tracing server. Jaeger agent receives the traces sent by oCIS (see the needed configuration for oCIS in this ([example](https://github.com/owncloud/ocis/blob/master/deployments/examples/ocis_traefik/monitoring_tracing/docker-compose-additions.yml)). The agent adds process tags, so that you can filter and query traces by oCIS instances and hosts. Finally it forwards the traces to the tracing server.

## Available specific Telegraf configurations

`TELEGRAF_SPECIFIC_CONFIG` enables you to select a specific configuration from `./config/telegraf_specific_config` by setting the variable to the file name without the file extension `.conf`.

Following specific configurations are available:

### none (`TELEGRAF_SPECIFIC_CONFIG=none`, default)

This option provides an empty configuration file, which means no specific configuration is used.

### oCIS single container (`TELEGRAF_SPECIFIC_CONFIG=ocis_single_container`)

This option provides an specific configuration for an oCIS run in a single container. It is applicable for following deployment examples:

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
