# A Cryptographic Engine for the Cloud: Designing a Secure and Agile IPsec VNF with StrongSwan and a Custom Data Plane

**Created by:** Manthan S & Likhith U

![Docker](https://img.shields.io/badge/Docker-Enabled-blue?logo=docker)
![StrongSwan](https://img.shields.io/badge/IPsec-StrongSwan-success?logo=shield)
![License](https://img.shields.io/badge/License-Academic-lightgrey)
![Status](https://img.shields.io/badge/Status-Deployment%20Ready-brightgreen)

---

## Overview

This containerized project establishes a **secure Site-to-Site IPsec VPN** between two peers using **StrongSwan**.  
The design abstracts underlying technology for flexibility and future compatibility.

This configuration works well even when the public IP of the virtual machine is hidden (not visible via `ip` command).  
Primary testing and validation were performed on **AWS**.

> ⚙️ **Note:** Running the container requires `--privileged` mode to load necessary kernel modules.  
> If not all modules are available, mount `/lib/modules` into the container. Additional changes may be required for non-standard architectures.

---

## Table of Contents

- [Running the Container](#running-the-container)
- [Configuration](#configuration)
  - [Manual Configuration](#manual-configuration)
  - [Enabling or Disabling Additional Modules](#enabling-or-disabling-additional-modules)
- [VTI Interface](#vti-interface)
- [Development](#development)
  - [Freezing Versions](#freezing-versions)
- [Authors](#authors)
- [License](#license)

---

## Running the Container

Execute the following command:

```bash
docker run --privileged \
    -v /lib/modules:/lib/modules \
    --env-file environment.sh \
    openvnf/vnf-ipsec
Configuration
Configuration is managed via environment variables, or a separate file referenced by $ENVFILE.

Example:

bash
Copy code
# Source configuration file before startup
ENVFILE=environment.sh
Core Environment Variables
Variable	Description
SET_ROUTE_DEFAULT_TABLE	Add route for tunnel in default routing table (useful with Calico or host networking)
IPSEC_LOCALIP	Local IP address (%any if dynamic)
IPSEC_LOCALID	Local identifier (must match IPSEC_REMOTEID on peer)
IPSEC_LOCALNET	Local subnet to share over VPN
IPSEC_REMOTEIP	Public IP of remote peer
IPSEC_REMOTENET	Remote subnet to share
IPSEC_REMOTEID	Remote identifier for connection
IPSEC_PSK	Pre-shared key (use a long random string)
IPSEC_KEYEXCHANGE	Key exchange protocol (ike / ikev1 / ikev2)
IPSEC_ESPCIPHER	ESP encryption/authentication algorithms
IPSEC_IKECIPHER	IKE/ISAKMP algorithms
IPSEC_LIFETIME	Connection lifetime (e.g., 1h, 3h, 24h)
IPSEC_IKELIFETIME	IKE SA lifetime before rekey (default: 3h)
IPSEC_FORCEUDP	Force UDP encapsulation even without NAT (yes / no)
IPSEC_IKEREAUTH	Reauthenticate on IKE_SA rekey (yes / no)
IPSEC_INTERFACES	Interfaces to bind Charon to (comma-separated)
DEBUG	Enable debug output (yes)

🔒 To use keys and certificates instead of PSK, repository code must be extended.

Manual Configuration
You can bypass environment variables and mount a custom StrongSwan configuration.

Enable manual mode:

bash
Copy code
IPSEC_USE_MANUAL_CONFIG=TRUE
Provide the following files:

File	Destination
ipsec.conf	/etc/ipsec.config.d/ipsec.<connection-name>.conf
ipsec.secrets	/etc/ipsec.secrets.de/ipsec.<connection-name>.secrets

Optional overrides:

bash
Copy code
/etc/ipsec.conf
/etc/ipsec.secrets
By default, Charon generates configurations dynamically.
To provide a custom Charon configuration:

bash
Copy code
IPSEC_MANUAL_CHARON=True
Enabling or Disabling Additional Modules
StrongSwan’s Charon daemon supports many plugins.
Enable or disable modules by mounting configuration files under /etc/strongswan.d/charon/.

Example (farp.conf default):

bash
Copy code
farp {
    load = no
}
To customize, mount your configuration at:

bash
Copy code
/etc/strongswan.d/charon/farp.conf
VTI Interface
The container supports Virtual Tunnel Interfaces (VTI) for routing IPsec traffic.

Related Environment Variables
nginx
Copy code
IPSEC_LOCALIP
IPSEC_VTI_KEY
IPSEC_VTI_STATICROUTES
IPSEC_VTI_IPADDR_LOCAL
IPSEC_VTI_IPADDR_PEER
Usage
Run the container with init to initialize a VTI tunnel.

Set a tunnel key:

bash
Copy code
IPSEC_VTI_KEY=<integer>
Add static routes (optional):

bash
Copy code
IPSEC_VTI_STATICROUTES=<comma-separated routes>
Assign point-to-point addresses:

bash
Copy code
IPSEC_VTI_IPADDR_LOCAL
IPSEC_VTI_IPADDR_PEER
Development
Freezing Versions
To upgrade or lock package versions:

bash
Copy code
apk update && apk upgrade
freeze_apk_versions
Copy the generated /root/MANIFEST to the repository and commit.

License
This project is licensed for academic and research purposes only.
© 2025 Manthan S & Likhith U. All rights reserved.











