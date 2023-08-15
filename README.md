# OpenVPN for Docker
***OpenVPN in a Docker container complete with an EasyRSA PKI CA.***
***

> `WARNING:` This is a draft documentation - instructions can be inaccurate
***

## Server Setup

### 1. First create a volume to store OpenVPN permanent configuration

#### a) Using a docker volume container
```bash
export OVPN_DATA=openvpn_data
docker volume create --name ${OVPN_DATA}
```

#### b) Using a local directory on host
```bash
export OVPN_DATA=${PWD}/ovpn_data
mkdir -p ${OVPN_DATA}
```

### 2. Initialize the server configuration
```bash
docker run \
  --rm \
  -v ${OVPN_DATA}:/etc/openvpn \
  infra7/openvpn initopenvpn \
    -u udp://vpn.example.org

docker run \
  --rm \
  -v ${OVPN_DATA}:/etc/openvpn \
  infra7/openvpn initpki
```

### 3. Generate a client certificate
```bash
docker run \
  --rm \
  -v ${OVPN_DATA}:/etc/openvpn \
  infra7/openvpn easyrsa \
    build-client-full ${CLIENTNAME}

docker run \
  --rm \
  -v ${OVPN_DATA}:/etc/openvpn \
  infra7/openvpn getclient ${CLIENTNAME} 
    > ${CLIENTNAME}.ovpn
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
```bash
docker run \
  --rm \
  --name ovpn-temp \
  -v ${OVPN_DATA}:/etc/openvpn \
  scratch \
  sleep infinity

docker cp \
  /path/to/config/${CLIENTNAME}.ovpn \
  ovpn-temp:/etc/openvpn/

docker stop ovpn-temp
```

#### b) Using a local directory on host
```bash
export OVPN_DATA=${PWD}/ovpn_data

mkdir -p ${OVPN_DATA}
cp /path/to/config/${CLIENTNAME}.ovpn ${OVPN_DATA}/
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
  -v /etc/localtime:/etc/localtime:ro \
  infra7/openvpn
```

## Advanced Options
(Nothing here yet)
