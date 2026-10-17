# --- Build stage ---
FROM cgr.dev/chainguard/wolfi-base:latest AS build

ARG TARGETOS
ARG TARGETARCH
ARG VERSION=13.2.4
ARG REVISION=38252365904

WORKDIR /build

ADD https://dl.grafana.com/grafana/release/${VERSION}/grafana_${VERSION}_${REVISION}_${TARGETOS}_${TARGETARCH}.tar.gz \
  /build/grafana.tar.gz
ADD https://dl.grafana.com/grafana/release/${VERSION}/grafana_${VERSION}_${REVISION}_${TARGETOS}_${TARGETARCH}.tar.gz.sha256 \
  /build/grafana.tar.gz.sha256

RUN set -ex ; \
  echo "$(awk '{print $1;}' grafana.tar.gz.sha256) grafana.tar.gz" | sha256sum -c ; \
  tar -xvzf /build/grafana.tar.gz --strip-components=1

# --- Final stage ---
FROM cgr.dev/chainguard/wolfi-base:latest

COPY --from=build --chown=0:0 --chmod=0755 \
  /build/bin /usr/share/grafana/bin
COPY --from=build --chown=0:0 --chmod=0755 \
  /build/conf /usr/share/grafana/conf
COPY --from=build --chown=0:0 --chmod=0755 \
  /build/docs /usr/share/grafana/docs
COPY --from=build --chown=0:0 --chmod=0755 \
  /build/plugins-bundled /usr/share/grafana/plugins-bundled
COPY --from=build --chown=0:0 --chmod=0755 \
  /build/public /usr/share/grafana/public

COPY --chown=0:0 --chmod=0644 \
  grafana.default.ini /etc/grafana/grafana.ini

RUN set -ex ; \
  addgroup -S -g 999 grafana ; \
  adduser -S -D -H -s /bin/false -u 999 -G grafana grafana

RUN install -d -o grafana -g grafana -m 750 /etc/grafana /var/lib/grafana

USER grafana

EXPOSE 3000/tcp

VOLUME [ "/var/lib/grafana" ]

WORKDIR /usr/share/grafana

ENTRYPOINT [ "/usr/share/grafana/bin/grafana" ]

CMD [ "server", "--config=/etc/grafana/grafana.ini" ]