ARG VERSION="45"
FROM quay.io/fedora/fedora:${VERSION} AS build

ENV ROOTFS="/build"

RUN --mount=type=bind,source=packages,target=/packages \
  set -ex ; \
    mkdir -p ${ROOTFS} ; \
    dnf install -y \
      --installroot=${ROOTFS} \
      --use-host-config \
      --no-docs \
      --setopt=install_weak_deps=False \
      $(paste /packages) ; \
    dnf --installroot=${ROOTFS} clean all

COPY --chown=0:0 \
  systemd/10-disable-sandboxing.conf ${ROOTFS}/etc/systemd/system/service.d/10-disable-sandboxing.conf

COPY --chown=0:0 --chmod=0644 \
  dnf.conf ${ROOTFS}/etc/dnf/dnf.conf
 
RUN truncate -s0 ${ROOTFS}/etc/machine-id

RUN rm -rf ${ROOTFS}/var/cache/* ${ROOTFS}/var/log/dnf* ${ROOTFS}/etc/ssh/ssh_host_*

# --- Final stage ---
FROM scratch

COPY --from=build /build/ /

ENV container="docker"

STOPSIGNAL SIGRTMIN+3

ENTRYPOINT [ "/usr/lib/systemd/systemd" ]