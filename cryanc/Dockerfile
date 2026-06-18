# Crypto Ancienne (carl) - TLS proxy for legacy browsers.
#
# carl has no official image and does not bind a network port itself: it reads a
# full proxy request from stdin and writes the response to stdout. We compile it
# from source and serve it per connection via socat acting as an inetd-style
# super-server.

# --- build: compile carl ---
FROM alpine:3.20 AS build
ARG CRYANC_REF=main
RUN apk add --no-cache git gcc musl-dev
RUN git clone https://github.com/classilla/cryanc.git /src
WORKDIR /src
RUN git checkout "$CRYANC_REF"
RUN gcc -O3 -o carl carl.c

# --- runtime ---
FROM alpine:3.20
# netcat-openbsd: provides nc with -z flag (BusyBox nc does not support it).
RUN apk add --no-cache socat netcat-openbsd \
    && addgroup -S carl \
    && adduser -S -G carl -H -s /sbin/nologin carl
COPY --from=build /src/carl /usr/local/bin/carl

# CARL_PORT: internal listen port. CARL_OPTS: options for carl(1).
ENV CARL_PORT=8765 \
    CARL_OPTS="-p -t"

EXPOSE 8765

USER carl

HEALTHCHECK --interval=30s --timeout=3s --start-period=5s --retries=3 \
    CMD nc -z 127.0.0.1 "$CARL_PORT" || exit 1

# socat accepts each TCP connection and pipes the socket to carl's stdin/stdout.
CMD ["sh", "-c", "exec socat TCP-LISTEN:${CARL_PORT},reuseaddr,fork EXEC:\"/usr/local/bin/carl ${CARL_OPTS}\""]
