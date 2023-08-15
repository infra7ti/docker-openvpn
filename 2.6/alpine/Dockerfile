# Dockerfile for infra7/docker-openvpn
# Docker image based on: kilemanna/openvpn and martin/openvpn 

FROM alpine:latest AS builder

RUN apk upgrade --update --no-cache \
  && apk add --update --no-cache \
    git \
    go \
  && git clone https://github.com/rakasatria/ovpn-radius /build \
  && cd /build/src \
  && go mod tidy \
  && go build


# -- Main image ---------------------------------------------------------------
FROM alpine:latest

LABEL MAINTAINER "Christian Tosta <tosta at i7ti.net>"

COPY --from=builder --chmod=0755 /build/src/ovpn-radius /usr/local/bin
COPY --chmod=0755 ../../bin /usr/local/bin

RUN apk upgrade --update --no-cache \
  && apk add --update --no-cache \
    bash \
    easy-rsa \
    freeradius-utils \
    iptables \
    openssl \
    openvpn \
    sqlite \
  && for s in /etc/openvpn/*.sh; do \
      mv -v ${s} /usr/local/bin/ovpn-$(echo ${s} | xargs basename); \
    done \
  && ln -s /usr/share/easy-rsa/easyrsa /usr/local/bin/

ENV OPENVPN=/etc/openvpn \
    EASYRSA=/usr/share/easy-rsa \
    EASYRSA_CRL_DAYS=3650 \
    EASYRSA_PKI=$OPENVPN/pki \
    EASYRSA_VARS_FILE=/etc/openvpn/vars

VOLUME ["/etc/openvpn"]
VOLUME ["/etc/openvpn/plugin"]

EXPOSE 1194/udp

WORKDIR /etc/openvpn

CMD ["startopenvpn"]
