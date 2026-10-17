# --- Build stage ---
FROM cgr.dev/chainguard/wolfi-base:latest AS base-common

RUN --mount=type=cache,target=/etc/apk/cache \
  apk add \
    bash \
    bison \
    build-base \
    clang-21 \
    cmake \
    execline-dev \
    flex \
    gc-dev \
    icu-dev \
    libedit-dev \
    libuuid \
    lz4-dev \
    meson \
    ninja \
    openssl-4.0-dev \
    perl \
    systemd-dev \
    util-linux-dev \
    zlib-dev \
    zstd-dev

# --- Build stage ---
FROM base-common AS build-server

ARG VERSION="18.6"

ADD https://ftp.postgresql.org/pub/source/v${VERSION}/postgresql-${VERSION}.tar.gz /build/postgresql-${VERSION}.tar.gz
ADD https://ftp.postgresql.org/pub/source/v${VERSION}/postgresql-${VERSION}.tar.gz.sha256 /build/postgresql-${VERSION}.tar.gz.sha256

WORKDIR /build

RUN set -ex ; \
  cat postgresql-${VERSION}.tar.gz.sha256 | sha256sum -c ; \
  tar -xvzf postgresql-${VERSION}.tar.gz --strip-components=1

RUN --mount=type=bind,source=meson-config.ini,target=/build/meson-config.ini \
  set -ex ; \
    meson setup --native-file meson-config.ini builddir ; \
    ninja -C builddir ; \
    meson install -C builddir --destdir "${PWD}/dist"

# --- Build stage ---
FROM base-common AS base-extensions

COPY --from=build-server /build/dist/usr/bin/pg_config      /usr/bin/pg_config
COPY --from=build-server /build/dist/usr/lib/postgresql     /usr/lib/postgresql
COPY --from=build-server /build/dist/usr/include/postgresql /usr/include/postgresql

ENV PKG_CONFIG_PATH="/usr/lib/postgresql/pkgconfig"

# --- Build stage ---
FROM base-extensions AS build-pgcron

ARG PG_CRON_VERSION="1.6.8"

ADD https://github.com/citusdata/pg_cron.git#v${PG_CRON_VERSION} /build

WORKDIR /build

RUN set -ex ; \
  make -j"$(nproc)" ; \
  make DESTDIR="${PWD}/dist" install

# --- Build stage ---
FROM base-extensions AS build-pgrepack

ARG PG_REPACK_VERSION="1.5.3"

ADD https://github.com/reorg/pg_repack.git#ver_${PG_REPACK_VERSION} /build

WORKDIR /build

RUN set -ex ; \
  make -j"$(nproc)" ; \
  make DESTDIR="${PWD}/dist" install

# --- Build stage ---
FROM base-extensions AS build-pgpartman

ARG PG_PARTMAN_VERSION="5.5.0"

ADD https://github.com/pgpartman/pg_partman.git#v${PG_PARTMAN_VERSION} /build

WORKDIR /build

RUN set -ex ; \
  make -j"$(nproc)" ; \
  make DESTDIR="${PWD}/dist" install

# --- Build stage ---
FROM base-extensions AS build-pgvector

ARG PG_VECTOR_VERSION="0.8.7"

ADD https://github.com/pgvector/pgvector.git#v${PG_VECTOR_VERSION} /build

WORKDIR /build

RUN set -ex ; \
  make -j"$(nproc)" ; \
  make DESTDIR="${PWD}/dist" install

# --- Build stage ---
FROM base-extensions AS build-pgstatmonitor

ARG PG_STAT_MONITOR_VERSION="2.4.0"

ADD https://github.com/percona/pg_stat_monitor.git#${PG_STAT_MONITOR_VERSION} /build

ENV USE_PGXS=1

WORKDIR /build

RUN set -ex ; \
  make -j"$(nproc)" ; \
  make DESTDIR="${PWD}/dist" install

# --- Build stage ---
FROM base-extensions AS build-pgtextsearch

ARG PG_TEXTSEARCH_VERSION="1.5.1"

ADD https://github.com/timescale/pg_textsearch.git#v${PG_TEXTSEARCH_VERSION} /build

WORKDIR /build

RUN set -ex ; \
  make -j"$(nproc)" ; \
  make DESTDIR="/build/dist" install

# --- Build stage ---
FROM base-extensions AS build-timescaledb

ARG TIMESCALEDB_VERSION="2.30.2"

ADD https://github.com/timescale/timescaledb.git#${TIMESCALEDB_VERSION} /build

WORKDIR /build

RUN BUILD_DIR="/build/tsbuild" ./bootstrap

WORKDIR /build/tsbuild
RUN set -ex ; \
  make -j"$(nproc)" ; \
  make DESTDIR="/build/dist" install

# --- Build stage ---
FROM base-extensions AS build-hypopg

ARG HYPOPG_VERSION="1.4.3"

ADD https://github.com/HypoPG/hypopg.git#${HYPOPG_VERSION} /build

WORKDIR /build

RUN set -ex ; \
  make -j"$(nproc)" ; \
  make DESTDIR="/build/dist" install

# --- Build stage ---
FROM base-extensions AS build-pgbackrest

ARG PGBACKREST_VERSION="2.59.2"

ADD https://github.com/pgbackrest/pgbackrest/releases/download/release/${PGBACKREST_VERSION}/pgbackrest-${PGBACKREST_VERSION}.tar.gz \
  /build/pgbackrest-${PGBACKREST_VERSION}.tar.gz
ADD https://github.com/pgbackrest/pgbackrest/releases/download/release/${PGBACKREST_VERSION}/pgbackrest-${PGBACKREST_VERSION}.tar.gz.sha256sum \
  /build/pgbackrest-${PGBACKREST_VERSION}.tar.gz.sha256sum

WORKDIR /build

RUN set -ex ; \
  cat pgbackrest-${PGBACKREST_VERSION}.tar.gz.sha256sum | sha256sum -c ; \
  tar -xvzf pgbackrest-${PGBACKREST_VERSION}.tar.gz \
    --strip-components=1 --exclude="pgbackrest-${PGBACKREST_VERSION}/dist"

RUN --mount=type=cache,target=/etc/apk/cache \
  apk add \
    bzip2-dev \
    libssh2-dev \
    libxml2-dev \
    yaml-dev

RUN set -ex ; \
  meson setup "${PWD}/dist" "${PWD}" ; \
  ninja -C "${PWD}/dist"

# --- Build stage ---
FROM base-extensions AS build-pgcopydb

ARG PGCOPYDB_VERSION="0.18"

ADD https://github.com/dimitri/pgcopydb.git#v${PGCOPYDB_VERSION} /build

WORKDIR /build

RUN make DESTDIR="${PWD}/dist" -j"$(nproc)" install

# --- Final stage ---
FROM cgr.dev/chainguard/wolfi-base:latest

ARG VERSION="18.6"

# Compatibility environment variables
ENV PG_MAJOR="${VERSION%%.*}"
ENV PGDATA="/var/lib/postgresql/${PG_MAJOR}/data"

# Dependencies
RUN --mount=type=cache,target=/etc/apk/cache \
  apk add \
    gc \
    glibc-locale-en \
    gosu \
    icu \
    libedit \
    liblz4-1 \
    libbz2-1 \
    libssh2 \
    libsystemd \
    libuuid \
    libxml2-16 \
    libzstd1 \
    posix-libc-utils \
    postgresql-${PG_MAJOR}-oci-entrypoint

# Core server
COPY --from=build-server --exclude=usr/include /build/dist/ /

# Extensions
COPY --from=build-pgcron    /build/dist/ /
COPY --from=build-pgrepack  /build/dist/ /
COPY --from=build-pgpartman /build/dist/ /
COPY --from=build-pgvector  /build/dist/ /
COPY --from=build-pgstatmonitor /build/dist/ /
COPY --from=build-pgtextsearch /build/dist/ /
COPY --from=build-timescaledb /build/dist/ /
COPY --from=build-hypopg /build/dist/ /

# Additional tools
COPY --from=build-pgbackrest /build/dist/src/pgbackrest /usr/bin/pgbackrest
COPY --from=build-pgcopydb /build/dist/usr/bin/pgcopydb /usr/bin/pgcopydb

RUN set -ex ; \
  addgroup -S -g 999 postgres ; \
  adduser -S -D -s /bin/sh -h /var/lib/postgresql -u 999 -G postgres postgres

RUN set -ex ; \
  install -d -o postgres -g postgres -m 0755 /etc/postgresql/config.d ; \
  install -d -o postgres -g postgres -m 0700 ${PGDATA} /var/lib/postgresql/backups

COPY --chown=postgres:postgres \
  postgresql.default.conf /etc/postgresql/postgresql.conf

ENV LANG="en_US.utf8"
ENV POSTGRES_INITDB_ARGS="--no-instructions --data-checksums"

EXPOSE 5432/tcp

STOPSIGNAL SIGINT

VOLUME [ "/var/lib/postgresql" ]

ENTRYPOINT [ "docker-entrypoint.sh" ]

CMD [ "postgres", "-c", "config_file=/etc/postgresql/postgresql.conf" ]
