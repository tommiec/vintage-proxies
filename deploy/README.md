# Deploy Stack (Pre-built Images)

This folder contains a `compose.yaml` that uses only pre-built images — no local
`build:` context. Use it when your deployment tool cannot build images at deploy time
(Portainer CE, a remote Docker host, any GitOps runner that expects a registry image).

The root [compose.yaml](../compose.yaml) builds `cryanc/carl` from
source and is meant for local development. Here it is replaced by an image published
to GHCR by GitHub Actions:

```text
ghcr.io/tommiec/cryanc-carl:latest
```

This image must be public for unauthenticated pulls. See [Forking](../README.md#forking)
in the root README if you want to publish your own images.

See the root [Update Model](../README.md#update-model) for which components update
from upstream registries and which ones require a rebuild in this repository.

## Files

- `compose.yaml` — stack definition using pre-built images only.
- `.env.example` — template for the `.env` file; copy and fill in `PROXY_HOSTNAME`.

## DNS migration before deployment

This stack no longer supplies DNS. Move any former NAS-DNS clients to their
network resolver and test them before deploying. Preserve old DNS data until
the migration is verified; do not remove shared networks or other services.

## Setup

1. Copy `.env.example` to `.env` alongside `compose.yaml` and set `PROXY_HOSTNAME`.
2. Deploy with your tool of choice:
   - **Plain Docker Compose**: `docker compose up -d`
   - **Portainer CE**: point the stack at this `compose.yaml` and supply `.env` as the
     environment file.
   - **GitOps runner**: commit `compose.yaml`; keep `.env` out of git and supply it as
     a secret or side-loaded file.
3. Configure the browser's proxy to `<NAS-IP>` and the relevant port (see the root
   [README](../README.md#ports)).

## Updating cryanc

1. Edit `cryanc/Dockerfile` in this repo and push to `main`.
2. GitHub Actions rebuilds and publishes `ghcr.io/tommiec/cryanc-carl:latest`.
3. Pull and restart: `docker compose pull cryanc && docker compose up -d cryanc`.

Do not push local builds to GHCR. Let GitHub Actions publish the cryanc image.
