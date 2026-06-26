# Browservice — server-side Chromium rendering for legacy browsers.
#
# Browservice runs a full Chromium instance on the server and streams the
# rendered page as JPEG frames to the client over HTTP. The iMac only needs
# to display images and send click/key events — no JavaScript, no TLS, no
# modern CSS at the client side.
#
# We use the official AppImage from ttalvitie/browservice (GitHub Releases)
# and extract it with --appimage-extract to avoid the FUSE requirement inside
# Docker. The GitHub API is queried at build time so every image build picks
# up the latest release automatically.
#
# Chromium/CEF requires either the SYS_ADMIN capability or --no-sandbox.
# We pass --no-sandbox; see compose.yaml for the cap_add note.

FROM ubuntu:22.04

ARG DEBIAN_FRONTEND=noninteractive

RUN apt-get update \
    && apt-get install -y --no-install-recommends \
       ca-certificates \
       curl \
       jq \
    && rm -rf /var/lib/apt/lists/*

# Download the latest official x86_64 AppImage, extract in-place, discard archive.
RUN set -eux; \
    APPIMAGE_URL=$(curl -fsSL \
        https://api.github.com/repos/ttalvitie/browservice/releases/latest \
        | jq -r '[.assets[] | select(.name | test("x86_64.*\\.AppImage$"; "i"))] | .[0].browser_download_url'); \
    echo "Fetching: $APPIMAGE_URL"; \
    curl -fsSL "$APPIMAGE_URL" -o /opt/browservice.AppImage; \
    chmod +x /opt/browservice.AppImage; \
    cd /opt && ./browservice.AppImage --appimage-extract; \
    mv /opt/squashfs-root /opt/browservice; \
    rm /opt/browservice.AppImage

# Data directory for Browservice session state.
RUN mkdir -p /data \
    && useradd -r -s /sbin/nologin -d /data browservice \
    && chown browservice:browservice /data

VOLUME ["/data"]

ENV BROWSERVICE_PORT=8080

EXPOSE 8080

USER browservice
WORKDIR /data

# --no-sandbox is required for Chromium/CEF inside Docker without SYS_ADMIN.
# --http-listen-addr binds to all interfaces so the host can reach it.
# Adjust port via the BROWSERVICE_PORT env var or override the CMD.
CMD ["/opt/browservice/AppRun", \
     "--http-listen-addr=0.0.0.0:8080", \
     "--no-sandbox"]
