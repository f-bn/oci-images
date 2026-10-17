# --- Build stage ---
FROM cgr.dev/chainguard/wolfi-base:latest AS build

ARG VERSION="26.9.19.9"
ARG CHANNEL="stable"

WORKDIR /build

ADD https://packages.clickhouse.com/tgz/${CHANNEL}/clickhouse-common-static-${VERSION}-amd64.tgz \
  /build/clickhouse-common-static-${VERSION}-amd64.tgz
ADD https://packages.clickhouse.com/tgz/${CHANNEL}/clickhouse-common-static-${VERSION}-amd64.tgz.sha512 \
  /build/clickhouse-common-static-${VERSION}-amd64.tgz.sha512
ADD https://packages.clickhouse.com/tgz/${CHANNEL}/clickhouse-server-${VERSION}-amd64.tgz \
  /build/clickhouse-server-${VERSION}-amd64.tgz
ADD https://packages.clickhouse.com/tgz/${CHANNEL}/clickhouse-server-${VERSION}-amd64.tgz.sha512 \
  /build/clickhouse-server-${VERSION}-amd64.tgz.sha512
ADD https://packages.clickhouse.com/tgz/${CHANNEL}/clickhouse-client-${VERSION}-amd64.tgz \
  /build/clickhouse-client-${VERSION}-amd64.tgz
ADD https://packages.clickhouse.com/tgz/${CHANNEL}/clickhouse-client-${VERSION}-amd64.tgz.sha512 \
  /build/clickhouse-client-${VERSION}-amd64.tgz.sha512

RUN set -ex; \
  for package in clickhouse-common-static clickhouse-server clickhouse-client; do \
    archive="${package}-${VERSION}-amd64.tgz" ; \
    sha512sum -c "${archive}.sha512" ; \
    tar -xzf "$archive" --strip-components=1 ; \
  done

# --- Final stage ---
FROM cgr.dev/chainguard/wolfi-base:latest

COPY --from=build --chown=0:0 /build/usr/bin/ /usr/bin/
COPY --from=build --chown=0:0 /build/usr/share/clickhouse /usr/share/clickhouse

RUN set -ex ; \
  addgroup -S -g 999 clickhouse ; \
  adduser -S -D -H -s /bin/false -u 999 -G clickhouse clickhouse

RUN set -ex ; \
  install -d -o clickhouse -g clickhouse -m 0750 \
    /etc/clickhouse-server \
    /etc/clickhouse-server/config.d \
    /etc/clickhouse-server/users.d \
    /etc/clickhouse-keeper \
    /etc/clickhouse-keeper/config.d \
    /etc/clickhouse-client \
    /etc/clickhouse-client/config.d ; \
  install -d -o clickhouse -g clickhouse -m 0755 /var/lib/clickhouse ; \
  install -d -o clickhouse -g clickhouse -m 0755 /var/log/clickhouse-server ; \
  install -d -o root -g root -m 0755 /docker-entrypoint-initdb.d

# Default configurations
COPY --from=build --chown=clickhouse:clickhouse --chmod=0644 \
  /build/etc/clickhouse-server/config.xml \
  /build/etc/clickhouse-server/users.xml \
  /etc/clickhouse-server/
COPY --from=build --chown=clickhouse:clickhouse --chmod=0644 \
  /build/etc/clickhouse-keeper/keeper_config.xml \
  /etc/clickhouse-keeper/
COPY --from=build --chown=clickhouse:clickhouse --chmod=0644 \
  /build/etc/clickhouse-client/config.xml /etc/clickhouse-client/config.xml

# Configuration overrides for Docker environment
COPY --chown=clickhouse:clickhouse --chmod=0644 \
  config.docker.xml /etc/clickhouse-server/config.d/00-docker.xml
COPY --chown=clickhouse:clickhouse --chmod=0644 \
  client.docker.xml /etc/clickhouse-client/config.d/00-docker.xml

ENV CLICKHOUSE_CONFIG=/etc/clickhouse-server/config.xml
ENV CLICKHOUSE_WATCHDOG_ENABLE=0

EXPOSE 8123/tcp 9000/tcp 9004/tcp 9005/tcp 9009/tcp

VOLUME [ "/var/lib/clickhouse" ]

ENTRYPOINT [ "/usr/bin/clickhouse" ]

CMD [ "docker-init" ]