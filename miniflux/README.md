## General informations

Custom Miniflux v2 image built from sources.

Built with Go 1.27.

## Miniflux configuration

This image doesn't come with a default configuration as Miniflux can be configured via environment variables or by bind-mouting a configuration file.

```shell
# Example using configuration file
$ cat miniflux.conf
RUN_MIGRATIONS=1
CREATE_ADMIN=1
ADMIN_USERNAME=<user>
ADMIN_PASSWORD=<pasword>
DATABASE_URL="postgres://miniflux:<password>@localhost/miniflux?sslmode=disable"

$ docker run [options] \
  -v /path/to/miniflux.conf:/etc/miniflux/miniflux.conf:ro \
  ghcr.io/f-bn/miniflux:2.3.3 -config-file /etc/miniflux/miniflux.conf
```

More informations about Miniflux configuration [here](https://miniflux.app/docs/configuration.html).