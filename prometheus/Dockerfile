# --- Build stage ---
FROM docker.io/node:24.22.1 AS build-webui

ARG VERSION="3.15.1"

ADD --keep-git-dir \
  https://github.com/prometheus/prometheus.git#v${VERSION} /build

WORKDIR /build

RUN set -ex ; \
  corepack enable ; \
  make assets

# --- Build stage ---
FROM docker.io/golang:1.27.3 AS build-binary

COPY --from=build-webui /build /build

WORKDIR /build

ENV SKIP_UI_BUILD=true

RUN --mount=type=bind,source=promu.yml,target=/build/.promu.yml \
  make build

# --- Final stage ---
FROM cgr.dev/chainguard/wolfi-base:latest

COPY --from=build-binary --chown=0:0 --chmod=0755 \
  /build/prometheus /build/promtool /usr/bin/
COPY --from=build-binary --chown=0:0 --chmod=0644 \
  /build/documentation/examples/prometheus.yml /etc/prometheus/prometheus.yml

RUN set -ex ; \
  addgroup -S -g 999 prometheus ; \
  adduser -S -D -H -s /bin/false -u 999 -G prometheus prometheus

RUN install -d -o prometheus -g prometheus -m 0750 /etc/prometheus /var/lib/prometheus

USER prometheus

EXPOSE 9090/tcp

VOLUME [ "/var/lib/prometheus" ]

WORKDIR /etc/prometheus

ENTRYPOINT [ "/usr/bin/prometheus" ]

CMD [ "--config.file=/etc/prometheus/prometheus.yml", \
      "--storage.tsdb.path=/var/lib/prometheus" ]