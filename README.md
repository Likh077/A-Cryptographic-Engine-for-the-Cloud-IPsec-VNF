Site-to-Site VPN Gateway

Created by: Manthan S and Likhith U

Overview

This container image establishes a secure IPsec-based site-to-site VPN connection between two peers.
The implementation currently utilizes StrongSwan, though the design goal is to keep the underlying technology abstracted for flexibility and future compatibility.

This configuration is suitable for environments where the public IP address of the virtual machine is not directly visible (for instance, hidden from the ip command).
Testing and validation were primarily conducted on AWS.

⚙️ Note: Running the container requires --privileged mode since it loads necessary kernel modules.
If not all modules are available on your host, mount /lib/modules into the container.
Code changes may be required for non-standard architectures.

🧭 Table of Contents

Running the Container

Configuration

Manual Configuration

Enabling or Disabling Additional Modules

VTI Interface

Development

Freezing Versions

🚀 Running the Container

Execute the following command to launch the container:

docker run --privileged \
    -v /lib/modules:/lib/modules \
    --env-file environment.sh \
    openvnf/vnf-ipsec

⚙️ Configuration

Configuration parameters are controlled through environment variables.
You may also define them in a separate file and reference it with $ENVFILE.

Example:

# Source the configuration file before startup
ENVFILE=

Core Environment Variables
# Add route for tunnel in default routing table (useful with Calico or host networking)
SET_ROUTE_DEFAULT_TABLE=FALSE

# Local IP address for this node (use '%any' if dynamic)
IPSEC_LOCALIP=

# Local identifier (must match IPSEC_REMOTEID on peer)
IPSEC_LOCALID=

# Local network to be shared over VPN
IPSEC_LOCALNET=

# Public IP of the remote peer
IPSEC_REMOTEIP=

# Remote network to be shared
IPSEC_REMOTENET=

# Remote identifier for the connection
IPSEC_REMOTEID=

# Pre-shared key (use a long random string)
IPSEC_PSK=

# Key exchange method: ike | ikev1 | ikev2
IPSEC_KEYEXCHANGE=

# ESP encryption/authentication algorithms
# Example: aes192gcm16-aes128gcm16-ecp256,aes192-sha256-modp3072
IPSEC_ESPCIPHER=

# IKE/ISAKMP SA algorithms
# Example: aes192gcm16-aes128gcm16-prfsha256-ecp256-ecp521,aes192-sha256-modp3072
IPSEC_IKECIPHER=

# Connection lifetime (e.g. 1h, 3h, 24h)
IPSEC_LIFETIME=

# IKE SA lifetime before rekeying (default: 3h)
IPSEC_IKELIFETIME=

# Force UDP encapsulation for ESP (yes | no)
IPSEC_FORCEUDP=

# Reauthenticate on IKE_SA rekey (yes | no)
IPSEC_IKEREAUTH=

# Network interfaces to bind Charon to (comma-separated)
IPSEC_INTERFACES=eth0

# Enable debugging output
# DEBUG=yes


🔒 If you prefer using keys and certificates instead of pre-shared keys, the repository code must be extended to support it.

🧩 Manual Configuration

You can bypass environment variables entirely and mount your custom StrongSwan configuration instead.

To enable manual configuration:

IPSEC_USE_MANUAL_CONFIG=TRUE


Provide the following files:

ipsec.conf
 → /etc/ipsec.config.d/ipsec.<connection-name>.conf

ipsec.secrets
 → /etc/ipsec.secrets.de/ipsec.<connection-name>.secrets

You may also override:

/etc/ipsec.conf

/etc/ipsec.secrets

By default, Charon’s configuration is generated dynamically.
If you wish to provide your own, set:

IPSEC_MANUAL_CHARON=True

⚡ Enabling or Disabling Additional Modules

StrongSwan’s charon daemon supports numerous modules and plugins.
You can manually enable or disable specific ones by editing their configuration files.

Example default for the farp module:

farp {
    load = no
}


To modify it, create a custom file and mount it at:

/etc/strongswan.d/charon/farp.conf

🌉 VTI Interface

Related environment variables:

IPSEC_LOCALIP
IPSEC_VTI_KEY
IPSEC_VTI_STATICROUTES
IPSEC_VTI_IPADDR_LOCAL
IPSEC_VTI_IPADDR_PEER


When the container is run with the argument init, it acts as an initialization container that sets up a VTI (Virtual Tunnel Interface) to route IPsec traffic.

To enable VTI:

IPSEC_VTI_KEY=<integer>


This creates a tunnel using IPSEC_LOCALIP as the local endpoint and associates it with the provided key.
Ensure the same key is used when running the default container mode.

Add static routes via:

IPSEC_VTI_STATICROUTES=<comma-separated routes>


Assign point-to-point addresses using:

IPSEC_VTI_IPADDR_LOCAL
IPSEC_VTI_IPADDR_PEER

🧠 Development
Freezing Versions

To upgrade or lock package versions:

Run:

apk update && apk upgrade


inside an existing image container.

Execute:

freeze_apk_versions


to regenerate the /root/MANIFEST file.

Copy it to the repository and commit the change.

🏁 Summary

The Site-to-Site VPN Gateway provides a modular, StrongSwan-based IPsec solution for securely linking two remote networks.
It supports flexible configuration, manual overrides, and integration with orchestration tools such as Docker and Kubernetes, making it ideal for enterprise, research, and cloud-based deployments.

Authors:
👨‍💻 Manthan S
👨‍💻 Likhith U