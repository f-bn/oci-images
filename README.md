<div align="center">
  <img src="https://raw.githubusercontent.com/opencontainers/artwork/master/oci/icon/color/oci-icon-color.png" alt="OCI Logo" width="150"/>

  **Personal OCI container images**

  ---
</div>

## 📋 Overview

This repository contains custom-built OCI container images for various popular open-source projects I use for my personal projects, tests and home production.

- 🔨 **Built from source** - Built from source whenever possible  
- ⚡ **Optimized builds** - Built with custom build options to remove uneeded features and reduce attack surface  
- 🛡️ **Secure base** - Uses [Wolfi](https://github.com/wolfi-dev) as minimal base image

## 📦 Available Images

| Image | Description | Links |
|-------|-------------|-------|
| **[caddy](./caddy/)** | Fast and extensible multi-platform HTTP/1-2-3 web server with automatic HTTPS | [Website](https://caddyserver.com/) |
| **[clickhouse](./clickhouse/)** | ClickHouse® is a real-time analytics database management system | [Website](https://clickhouse.com/) |
| **[fedora](./fedora/)** | Custom Fedora containers images | [Website](https://fedoraproject.org/) |
| **[grafana](./grafana/)** | The open and composable observability and data visualization platform | [Website](https://grafana.com/) |
| **[miniflux](./miniflux/)** | Minimalist and opinionated feed reader | [Website](https://miniflux.app/) |
| **[navidrome](./navidrome/)** | Modern Music Server and Streamer compatible with Subsonic/Airsonic | [Website](https://www.navidrome.org/) |
| **[pocket-id](./pocket-id/)** | A simple and easy-to-use OIDC provider that allows users to authenticate with their passkeys to your services | [GitHub](https://github.com/pocket-id/pocket-id) |
| **[postgresql](./postgresql/)** | Custom PostgreSQL image built from source with additional extensions and tools | [Website](https://www.postgresql.org/) |
| **[prometheus](./prometheus/)** | The Prometheus monitoring system and time series database | [Website](https://prometheus.io/) |
| **[telegraf](./telegraf/)** | Agent for collecting, processing, aggregating, and writing metrics, logs, and other arbitrary data | [GitHub](https://github.com/influxdata/telegraf) |

## 🚀 Quick Start

All images are publicly available through GitHub Container Registry:

```bash
# Pull an image
docker pull ghcr.io/f-bn/<image>:<tag>

# Example: Pull specific version
docker pull ghcr.io/f-bn/prometheus:3.7.3
```

## 🛠️ Building Locally

Each image directory contains its own `Dockerfile` and usage instructions. To build an image locally:

```bash
docker build -t <image-name>:<tag> [--build-arg VERSION=x.x.x] <image-name>
```

## License

See [LICENSE](./LICENSE) file for details.
