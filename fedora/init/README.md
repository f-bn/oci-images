## General informations

Custom Fedora *system* container image for testing purposes.

This image uses `systemd` as entrypoint to create a lightweight "system" environment (much like LXC or `systemd-nspawn` but with Docker).

## How to run this image ?

In order to launch a Docker container with `systemd` as main process, a set of specific options must be set:

```shell
$ docker run [options] \
  --detach \
  --tty \
  --env=container="docker"\  # optional as set in the image
  --stop-signal=SIGRTMIN+3 \ # optional as set in the image
  --cgroupns=private \
  --security-opt=writable-cgroups=true \
  --tmpfs=/run \
  --tmpfs=/run/lock \
  --tmpfs=/tmp \
  ghcr.io/f-bn/fedora:45-init
```