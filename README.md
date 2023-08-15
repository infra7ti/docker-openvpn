# OpenVPN for Docker
***OpenVPN in a Docker container complete with an EasyRSA PKI CA.***
***

`WARNING:` This is a draft documentation - instructions are not verified yet
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
```
```bash
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
```
```bash
docker run \
  --rm \
  -v ${OVPN_DATA}:/etc/openvpn \
  infra7/openvpn getclient ${CLIENTNAME} 
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
```
```bash
docker run \
  --rm \
  --name ovpn-temp \
  -v ${OVPN_DATA}:/etc/openvpn \
  scratch \
  sleep infinity
```
Copy configuration file(s) to /etc/openvpn inside the container
```bash
docker cp \
  /path/to/config/${CLIENTNAME}.conf \
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
cp /path/to/files/${CLIENTNAME}.conf ${OVPN_DATA}/client/
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

#### - Revoke a client certificate
If you need to remove access for a client then you can revoke the client certificate by running
```bash
docker run -v ${OVPN_DATA}:/etc/openvpn --rm -it infra7/openvpn revokeclient ${CLIENTNAME}
```

#### - List all generated certificate names
```bash
 docker run -v ${OVPN_DATA}:/etc/openvpn --rm infra7/openvpn listcerts
```

#### - Renew the CRL
```bash
docker run -v ${OVPN_DATA}:/etc/openvpn --rm -it infra7/openvpn revokeclient ${CLIENTNAME}
```

#### - Adjust container time according to host time
To set cointainer date/time to be the same as in the host map /etc/localtime to container (using "docker -v"):
```bash
docker run \
  (...) \
  -v /etc/localtime:/etc/localtime:ro \
  infra7/openvpn
```

#### - Enable Radius Authentication
Add additional settings to your server configuration file:
```
auth-user-pass-verify "/usr/local/bin/ovpn-radius auth " via-file # authenticate to radius
client-connect "/usr/local/bin/ovpn-radius acct " # sent acounting request start and update to radius
client-disconnect "/usr/local/bin/ovpn-radius stop " # sent acounting request stop to radius
```

Adjust configuration in `/etc/openvpn/plugin/config.json` with your own enviroment and data:
```
{
    "LogFile": "/proc/self/fd/1",
    "ServerInfo":
    {
      "Identifier": "OpenVPN",
      "IpAddress": "192.168.255.1",
      "PortType": "5",
      "ServiceType": "5"
    },
    "Radius":
    {
      "Authentication":
      {
        "Server": "10.10.10.124:1812",
        "Secret": "s3cr3t"
      },
      "Accounting":
      {
        "Server": "10.10.10.124:1813",
        "Secret": "s3cr3t"
      }
    }
}
```

> `Note:` Read more at https://github.com/rakasatria/ovpn-radius


#### - Enable Debug Output
To enable (bash) debug output set an environment variable with the name DEBUG and value of 1 (using "docker -e")
```bash
docker run -e DEBUG=1 \
  (...) \
  infra7/openvpn
```

## Default settings and features
* OpenVPN 2.6.0+
* Easy-RSA v3.0.1+
* tun mode because it works on the widest range of devices. tap mode, for instance, does not work on Android, except if the device is rooted.
* The UDP server uses192.168.255.0/24 for clients.
* TLS 1.2 minimum
* TLS auth key for HMAC security
* Diffie-Hellman parameters for perfect forward secrecy
* Verification of the server certificate subject
* Extended Key usage check of both client and server certificates
* 2048 bits key size
* Client certificate revocation functionality
* SHA256 signature hash
* AES-256-CBC cipher
* TLS cipher limited to TLS-ECDHE-RSA-WITH-AES-128-GCM-SHA256, TLS-ECDHE-ECDSA-WITH-AES-128-GCM-SHA256, TLS-DHE-RSA-WITH-AES-256-GCM-SHA384 or TLS-DHE-RSA-WITH-AES-256-CBC-SHA256
* Compression enabled and set to adaptive
* Floating client ip's enabled
* Tweaks for Windows clients
* net30 topology because it works on the widest range of OS's. p2p, for instance, does not work on Windows.
* Google DNS (8.8.4.4 and 8.8.8.8)
* The configuration is located in /etc/openvpn
* Certificates are generated in /etc/openvpn/pki

## Building Docker image locally
For now only OpenVPN 2.6.x on Alpine are supported
```bash
git clone https://github.com/infra7ti/docker-openvpn.git && cd docker-openvpn
VERSION=2.6 TARGET_OS=alpine docker-compose build --no-cache
```

***
Based on: kylemanna/docker-openvpn and martin/openvpn
