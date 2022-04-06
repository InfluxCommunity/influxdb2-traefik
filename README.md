# influxdb2-traefik

Simple Configuration for how to render InfluxDB v2 behind Traefik Proxy using Docker Compose

## Situation

InfluxDBv2 does not provide any configuration flags / environment variables
that can help render the UI as paths e.g. `/influxdb`. 

An [Open Issue since InfluxDB2.0-alpha](https://github.com/influxdata/influxdb/issues/15721) 
has yet to find a solution.

However, there is a potential way to circumvent this problem but using `Host` in Traefik.

> NOTE: PLEASE DO NOT DEPLOY THIS STACK IN PRODUCTION.

## Usage

Create a docker network called `proxy-network`

```bash
docker network create proxy-network
```

Bring the stack up using:

```bash
docker compose -p influxdb2-traefik up
```

## Results / Caveats / Design

The UI will be available on `http://influxdb.localhost` instead of
`http://localhost/influxdb`

### Metrics
You can configure Traefik to store its metrics via its Static Configuration file
(`traefik.toml`) however you will need to add the information like:

- __Bucket__
- __OrgName__
- __Token__

as _hard-coded_ credentials because [Traefik Static Configuration DO NOT support Go Templating]
(https://doc.traefik.io/traefik/providers/file/#go-templating)

A good possibility is to use placeholders and substitute them via `envsubstr` in a separate 
bash file, before bringing the stack up

### Common Variables Usage

within the `docker-compose.yml` file

```yaml
x-common-env-variables: &common-env
  DOCKER_INFLUXDB_INIT_ORG: ${INFLUXDB_INIT_ORG}
  DOCKER_INFLUXDB_INIT_BUCKET: ${INFLUXDB_INIT_BUCKET}
  DOCKER_INFLUXDB_INIT_ADMIN_TOKEN: ${INFLUXDB_INIT_ADMIT_TOKEN}
```

can be used to configure both, InfluxDB and Telegraf for and _out-of-the-box_ stack. However,
additional required Environment Variables for InfluxDB are introduced via `influxdb.env`.

The values are filled by `.env` which docker compose automatically uses during stack up.

### Security

not the best thing to let Traefik use `/var/run/docker.sock`, however setting it to _read-only_, 
avoiding extra privilege escalations and using a namespace should make this example a bit more secure.

### Configuration Checks

Use the integrated `compose` CLI from docker to verify if everything looks okay or not!

```bash
docker compose config
```

## LICENSE

Published under __MIT License__

## Maintainer

[Shan Desai](https://github.com/shantanoo-desai)
