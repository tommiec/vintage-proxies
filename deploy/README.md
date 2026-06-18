# Deploy Stack (Pre-built Images)

This folder contains a `compose.yaml` that uses only pre-built images — no local
`build:` context. Use it when your deployment tool cannot build images at deploy time
(Portainer CE, a remote Docker host, any GitOps runner that expects a registry image).

The root [compose.yaml](../compose.yaml) builds `cryanc/carl` from source and is meant
for local development. Here, `cryanc` is replaced by the image published to GHCR by
GitHub Actions:

```text
ghcr.io/tommiec/cryanc-carl:latest
```

## Files

- `compose.yaml` — stack definition using pre-built images only.
- `.env.example` — template for the `.env` file; copy and fill in `PROXY_HOSTNAME`.

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
3. Pull the new image and restart: `docker compose pull cryanc && docker compose up -d cryanc`.

Do not push local `cryanc-carl` builds to GHCR. Let GitHub Actions publish the image.
