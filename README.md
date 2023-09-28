# OpenVPN for Docker
***OpenVPN in a Docker container complete with an EasyRSA PKI CA.***
***

## Server Setup

### 1. First create a volume to store OpenVPN permanent configuration

#### a) Using a docker volume container
```bash
export OPENVPN_DATA=openvpn_data
docker volume create --name ${OPENVPN_DATA}
```

#### b) Using a local directory on host
```bash
export OPENVPN_DATA=${PWD}/data
mkdir -p ${OPENVPN_DATA}
```

### 2. Initialize the server configuration
```bash
export COUNTRY="WW"
export PROVINCE="The Internet"
export CITY="N/A"
export COMPANY="Acme Inc."
export EMAIL="mail@example.com"
export OU="OpenVPN"
export SERVER="myserver"

docker run \
  --rm \
  -e EASYRSA_REQ_SN="${COUNTRY}/${PROVINCE}/${CITY}/${COMPANY}/${EMAIL}/${OU}" \
  -v ${OVPN_DATA}:/etc/openvpn \
  infra7/openvpn \
    ovpn-create-server ${SERVER}
```

### 3. Generate clients configurations
```bash
export CLIENT1="client1"
export CLIENT1="client2"
docker run \
  --rm \
  -v ${OVPN_DATA}:/etc/openvpn \
  infra7/openvpn ovpn-create-client ${SERVER} ${CLIENT1} ${CLIENT2}
```
```bash
docker run \
  --rm \
  -v ${OVPN_DATA}:/etc/openvpn \
  infra7/openvpn cat /etc/openvpn/${SERVER}/configs/${CLIENT1}.ovpn
    > ${CLIENTNAME}.conf
```

### 4. Start OpenVPN server process
```bash
docker run \
  -d \
  --name openvpn \
  --cap-add=NET_ADMIN \
  --restart=always \
  -p 1194:1194/udp \
  -v ${OVPN_DATA}:/etc/openvpn \
  infra7/openvpn
```

## Client Setup

### 1. Create a volume to store OpenVPN configuration and populate it

#### a) Using a docker volume container

Run a temporary container with named volume mapped to /etc/openvpn
```bash
export OVPN_DATA=openvpn_data

docker run \
  --rm \
  --name ovpn-temp \
  --detach \
  -v ${OVPN_DATA}:/etc/openvpn \
  scratch \
  sleep infinity
```
Copy configuration file(s) to /etc/openvpn inside the container
```bash
docker cp \
  /path/to/config/${CLIENT1}.ovpn \
  ovpn-temp:/etc/openvpn/client/
```
Then stop the temporary container
```bash
docker stop ovpn-temp
```

#### b) Using a local directory on host

Create a directory on host to store configuration data
```bash
export OVPN_DATA=/path/to/project/ovpn_data
mkdir -p ${OVPN_DATA}
```
Then copy configuration file(s) to it
```bash
cp /path/to/files/${CLIENT1}.ovpn ${OVPN_DATA}/client/
```

### 2. Start OpenVPN server process
```bash
docker run \
  -d \
  --name openvpn \
  --cap-add=NET_ADMIN \
  --restart=always \
  -p 1194:1194/udp \
  -v ${OVPN_DATA}:/etc/openvpn \
  infra7/openvpn
```

## Advanced Options

### - Adjust container time according to host time
To set cointainer date/time to be the same as in the host map /etc/localtime to container (using "docker -v"):

```bash
docker run \
  (...) \
  -v /etc/localtime:/etc/localtime:ro \
  infra7/openvpn
```

### - Enable Debug Output
To enable (bash) debug output set the environment variable OPENVPN_DEBUG and value of _true_ (using "docker -e")

```bash
docker run -e OPENVPN_DEBUG=true \
  (...) \
  infra7/openvpn
```

## Default settings and features
* OpenVPN 2.6.0+
* Easy-RSA v3.0.1+
* tun mode because it works on the widest range of devices. tap mode, for instance, does not work on Android, except if the device is rooted
* The UDP server uses 10.8.0.0/24 and subnet topology for clients
* TLS 1.2 minimum
* TLS auth key for HMAC security
* Diffie-Hellman parameters for perfect forward secrecy
* Extended Key usage check of both client and server certificates
* 2048 bits key size
* Client certificate revocation functionality
* SHA256 signature hash
* AES-256-GCM cipher
* Data ciphers limited to AES-256-GCM:AES-128-GCM:AES-256-CBC:AES-128-CBC
* Support for multiple servers and clients

## Proposed settings and features
* Verification of the server certificate subject
* TLS cipher limited to TLS-ECDHE-RSA-WITH-AES-128-GCM-SHA256, TLS-ECDHE-ECDSA-WITH-AES-128-GCM-SHA256, TLS-DHE-RSA-WITH-AES-256-GCM-SHA384 or TLS-DHE-RSA-WITH-AES-256-CBC-SHA256
* Compression enabled and set to adaptive
* Floating client ip's enabled
* Tweaks for Windows clients

## Building Docker image locally
For now only OpenVPN 2.6.x on Alpine are supported

```bash
export VERSION=2.6
export TARGET_OS=alpine

git clone https://github.com/infra7ti/docker-openvpn.git && cd docker-openvpn
docker buildx build -t infra7/openvpn:latest -f ${VERSION}/${TARGET_OS}/Dockerfile --no-cache
```

***
