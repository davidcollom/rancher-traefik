# `rancher-traefik`

A [Traefik](https://traefik.io/) container image configured for [Rancher](https://www.rancher.com/).

Rancher can drive Traefik through its metadata or API configuration backend, so this image bundles a
Traefik configuration that enables the Rancher provider, and adds an entrypoint script that turns
environment variables (or Docker/Kubernetes secrets) into Traefik command line arguments.

## Contents

| Path | Description |
| ---- | ----------- |
| `Dockerfile` | Multi-stage build: copies the `traefik` binary and config into a minimal Alpine image. |
| `traefik.toml` | Traefik static configuration, including the Rancher provider. |
| `run.sh` | Entrypoint that loads secrets and converts `TRAEFIK_*` env vars into CLI flags. |
| `Makefile` | Build helpers. |

## Building

```bash
make build
```

Or directly:

```bash
docker build -t rancher-traefik:latest .
```

## Configuration

### Environment variables

Any environment variable prefixed with `TRAEFIK_` is converted into a Traefik command line flag. The
prefix is stripped, the name is lowercased and underscores become dots:

| Environment variable | Traefik flag |
| -------------------- | ------------ |
| `TRAEFIK_ACCESSLOG=true` | `--accesslog=true` |
| `TRAEFIK_LOG_LEVEL=DEBUG` | `--log.level=DEBUG` |
| `TRAEFIK_ENTRYPOINTS_WEB_ADDRESS=:8080` | `--entrypoints.web.address=:8080` |

### Secrets

Files placed in `/run/secrets/` are loaded into the environment before Traefik starts. The file name
becomes the variable name, uppercased with `/` replaced by `_`:

```
/run/secrets/TRAEFIK_API_DASHBOARD=true
  -> TRAEFIK_API_DASHBOARD=true
```

This makes it straightforward to mount Kubernetes or Docker secrets for sensitive values.

## Rancher provider

`traefik.toml` enables the Rancher configuration backend:

```toml
[rancher]
domain = "localhost"
exposedByDefault = false
enableServiceHealthFilter = true

[rancher.metadata]
intervalPoll = true

[rancher.api]
endpoint = "http://rancher/v1"
```

Set the `traefik.domain` label on a Rancher service to override the default domain, and note that
services are not exposed by default — each one must be opted in.

## Running

```bash
docker run -d \
  -p 80:80 -p 443:443 \
  -v /run/secrets:/run/secrets:ro \
  rancher-traefik:latest
```

## Debugging

Set `DEBUG` in the environment to print the resolved Traefik command before it is executed:

```bash
DEBUG=1 docker run --rm rancher-traefik:latest
```

## License

See the repository for licensing details.
