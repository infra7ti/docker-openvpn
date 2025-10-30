# Dockerfile for infra7/docker-openvpn

ARG ALPINE_VERSION=3.22
FROM alpine:${ALPINE_VERSION}

ARG OPENVPN_VERSION=2.6
LABEL \
  org.opencontainers.image.title="OpenVPN" \
  org.opencontainers.image.description="OpenVPN is a full-featured TLS VPN solution" \
  org.opencontainers.image.authors="https://github.com/infra7ti/docker-openvpn/issues" \
  org.opencontainers.image.url="https://github.com/infra7ti/docker-openvpn" \
  org.opencontainers.image.source="https://github.com/infra7ti/docker-openvpn.git" \
  org.opencontainers.image.vendor="Infra7 TI Services" \
  org.opencontainers.image.licenses="MIT" \
  org.opencontainers.image.version="${OPENVPN_VERSION}"

ARG OPENVPN_VERSION=2.6
ADD ${OPENVPN_VERSION}/root/ /

RUN apk upgrade --update --no-cache \
  && apk add --update --no-cache \
    bash \
    easy-rsa \
    envsubst \
    iptables \
    openssl \
    openvpn \
    sqlite \
  && chmod +x /sbin/entrypoint \
    /sbin/ovpn-* \
  && ln -s /usr/share/easy-rsa/easyrsa \
    /sbin/

EXPOSE 1194/udp

VOLUME ["/etc/openvpn"]
WORKDIR /etc/openvpn

ENTRYPOINT ["/sbin/entrypoint"]
CMD ["true"]
