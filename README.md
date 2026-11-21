# Docker Compose

[![CI](https://github.com/bstraehle/docker/actions/workflows/ci.yml/badge.svg)](https://github.com/bstraehle/docker/actions/workflows/ci.yml)

Run the demo apps locally with [Docker Compose](https://docs.docker.com/compose/). Two stacks share the same REST API:

| Stack | Services | URLs |
| --- | --- | --- |
| [`mvc-app_rest-api`](mvc-app_rest-api/compose.yaml) | [MVC app](https://github.com/bstraehle/dotnet-mvc-app) + [REST API](https://github.com/bstraehle/dotnet-rest-api) | http://localhost:8000, http://localhost:8001/swagger |
| [`spa_rest-api`](spa_rest-api/compose.yaml) | [Angular SPA](https://github.com/bstraehle/angular-spa) + [REST API](https://github.com/bstraehle/dotnet-rest-api) | http://localhost:8080, http://localhost:8001/swagger |

Both stacks publish the API on port 8001, so run one at a time. Ports are published on `localhost` only, except the SPA stack's API port (see below).

## Prerequisites

- [Docker Desktop](https://docs.docker.com/desktop/) or Docker Engine with the [Compose plugin](https://docs.docker.com/compose/install/) (`docker compose`, v2)

## Run

MVC app + REST API

```shell
git clone https://github.com/bstraehle/docker.git
cd docker/mvc-app_rest-api

docker compose up -d
docker compose ps
```

Open http://localhost:8000 and choose **Demo** to list data from the API. The MVC app calls the API over the Compose network (`http://rest-api:8080`).

Angular SPA + REST API

```shell
cd docker/spa_rest-api

docker compose up -d
```

Open http://localhost:8080. The SPA calls the API from your browser at `http://host.docker.internal:8001`, which resolves on Docker Desktop. On Docker Engine for Linux, add `127.0.0.1 host.docker.internal` to `/etc/hosts`.

> Note: Docker Desktop maps `host.docker.internal` to your machine's LAN IP, so this stack publishes the API's port 8001 on all interfaces. The demo API has no authentication, so only run the SPA stack on a trusted network.

Logs and cleanup

```shell
docker compose logs -f
docker compose down
```

Upgrading from the old `docker-compose.yml` files: the services were renamed, so containers that are still running from the old version keep ports 8000, 8001, and 8080 busy. Start with `docker compose up -d --remove-orphans` to replace them.

## Configuration

| Variable | Default | Purpose |
| --- | --- | --- |
| `MVC_APP_IMAGE` | `bstraehle/mvc-app:latest` | MVC app image |
| `REST_API_IMAGE` | `bstraehle/rest-api:latest` | REST API image |
| `SPA_IMAGE` | `bstraehle/spa:latest` | Angular SPA image |

Set them in the shell or in a `.env` file next to `compose.yaml`, e.g. to test a locally built image:

```shell
REST_API_IMAGE=restapi-image docker compose up -d
```

To get newer `:latest` images, run `docker compose pull` before `up`.

## Security Defaults

All containers run with a read-only root filesystem (writable `tmpfs` where needed), all Linux capabilities dropped, and `no-new-privileges`. The .NET containers run as the non-root `app` user (UID 1654). The nginx container keeps only the capabilities it needs to bind port 80 and drop to its worker user.

## CI

[`.github/workflows/ci.yml`](.github/workflows/ci.yml) validates both Compose files, starts each stack, and checks its endpoints on every push and pull request, and weekly to catch breaking changes in the `:latest` images.

## Resources

- [Compose file reference](https://docs.docker.com/reference/compose-file/)
- [Docker security best practices](https://docs.docker.com/build/building/best-practices/)
- Next steps: [Kubernetes](https://github.com/bstraehle/kubernetes), [Istio](https://github.com/bstraehle/istio), [Terraform](https://github.com/bstraehle/terraform)
