# --- Build stage ---
FROM docker.io/golang:1.27.3 AS build

ARG TARGETOS
ARG TARGETARCH
ARG VERSION="2.3.4"

ADD --keep-git-dir https://github.com/miniflux/v2.git#${VERSION} /build

WORKDIR /build

RUN make ${TARGETOS}-${TARGETARCH}

# --- Final stage ---
FROM cgr.dev/chainguard/wolfi-base:latest

ARG TARGETOS
ARG TARGETARCH

COPY --from=build --chown=0:0 --chmod=0755 \
  /build/miniflux-${TARGETOS}-${TARGETARCH} /usr/bin/miniflux

RUN set -ex ; \
  addgroup -S -g 999 miniflux ; \
  adduser -S -D -H -s /bin/false -u 999 -G miniflux miniflux

RUN install -o miniflux -g miniflux -m 0750 -d /etc/miniflux

USER miniflux

EXPOSE 8080/tcp

ENTRYPOINT [ "/usr/bin/miniflux" ]