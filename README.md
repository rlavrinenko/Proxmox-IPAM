# Proxmox-IPAM
PVE IPAM — IP Address Management Module for Proxmox VE

PVE IPAM is a local IP address management module for Proxmox VE that integrates directly into the native Proxmox Web interface on port 8006.

The module is designed to provide centralized visibility and management of IP address space used by Proxmox hosts, virtual machines, containers, local bridge networks, and manually defined networks.

No separate Web server, Docker container, VM, or additional management port is required.

Management interface:

https://PROXMOX-IP:8006

Node
└── IPAM


MAIN FEATURES

- Native integration into the Proxmox VE Web interface
- Automatic detection of IPv4 networks on vmbr interfaces
- Detection of directly connected routes
- Manual network creation
- Support for routed networks that do not have an IP assigned directly to the Proxmox bridge
- Support for CIDR networks such as:
  192.168.99.0/24
  192.168.50.16/28
  10.10.0.0/16

- Automatic network size calculation
- Used / Reserved / Free IP statistics
- Search for available IP addresses
- Manual IP reservations
- Duplicate IP detection
- IP conflict detection
- MAC address tracking
- Device / VM / LXC ownership information
- Active network scanning
- Passive ARP / neighbor discovery
- Search and filtering
- Support for VM and LXC inventory
- QEMU Guest Agent IP discovery where available


IPAM INTERFACE

The module contains four main sections:

IPAM
├── Networks
├── Addresses
├── Reservations
└── Conflicts


NETWORKS

The Networks section displays all detected and manually configured networks.

Example:

Network           Bridge   PVE IP          Source
192.168.99.0/24   vmbr0    192.168.99.1    Interface address
192.168.50.16/28  vmbr1    -               Manual
10.10.10.0/24     vmbr2    10.10.10.1      Connected route


Available operations:

- Add Network
- Edit Network
- Remove Network
- Find Free IP
- Active Scan
- Refresh


MANUAL NETWORKS

Networks can be added manually even if Proxmox itself does not have
an IP address inside that subnet.

Example:

Bridge:
vmbr1

Network:
192.168.50.16/28

PVE IP:
optional

Gateway:
optional

Name:
Additional /28

Note:
Routed public or private network


The module automatically calculates the usable address range.

For example:

192.168.50.16/28

Network:
192.168.50.16

Usable hosts:
192.168.50.17 - 192.168.50.30

Broadcast:
192.168.50.31


IMPORTANT:

Adding a network to IPAM does NOT modify the Proxmox network configuration.

The module does not automatically:

- create vmbr interfaces;
- assign addresses to Proxmox;
- modify /etc/network/interfaces;
- create Linux routes;
- modify gateways.

Manual networks are registered only for IP address management and inventory.


ADDRESSES

The Addresses section provides a unified IP inventory.

Example:

IP              Network             Bridge   Status      Owner
192.168.99.1    192.168.99.0/24     vmbr0    Used        PVE / vmbr0
192.168.99.5    192.168.99.0/24     vmbr0    Used        VM 100
192.168.99.10   192.168.99.0/24     vmbr0    Used        VM 104
192.168.99.20   192.168.99.0/24     vmbr0    Reserved    Printer
192.168.99.25   192.168.99.0/24     vmbr0    Conflict    Multiple devices


Address information can be collected from:

- Proxmox bridge interfaces
- Linux routing table
- VM configuration
- LXC configuration
- QEMU Guest Agent
- ARP table
- Linux neighbor table
- active scan results
- manual reservations


RESERVATIONS

Administrators can reserve an IP address for a specific system or device.

Example:

IP:
192.168.99.50

Network:
192.168.99.0/24

Bridge:
vmbr0

Name:
Office Printer

MAC:
AA:BB:CC:DD:EE:FF

Note:
Accounting department


Supported operations:

- Add
- Edit
- Remove


FIND FREE IP

The module can automatically find unused addresses inside a selected subnet.

Example:

Network:
192.168.99.0/24

Available addresses:

192.168.99.6
192.168.99.7
192.168.99.8
192.168.99.11


A free address can be reserved directly from the IPAM interface.


CONFLICT DETECTION

The Conflicts section detects duplicate IP usage.

Example:

192.168.99.20

Device 1:
VM 100
MAC AA:BB:CC:11:22:33

Device 2:
VM 105
MAC AA:BB:CC:44:55:66

Status:
CONFLICT


A conflict can also be detected when an IP is reserved for one MAC address
but another MAC address is discovered using the same IP.


ACTIVE SCAN

IPAM can actively scan a selected network to discover currently reachable hosts.

The scan refreshes:

- active hosts;
- ARP entries;
- MAC addresses;
- used IP addresses.


To prevent accidental large-scale scans, the active scanner is limited
to reasonably sized networks.

For example:

192.168.99.0/24
can be scanned completely.


Very large networks such as:

10.0.0.0/8

are not automatically scanned in full.


DATA STORAGE

IPAM configuration and manual reservations are stored locally on the Proxmox node.

Configuration:

/etc/pve-ipam/config.json


Permissions:

root:root
600


ARCHITECTURE

PVE IPAM uses the native Proxmox architecture:

Browser
   │
   │ HTTPS :8006
   ▼
pveproxy
   │
   ▼
Proxmox API
   │
   ▼
pvedaemon
   │
   ▼
PVE IPAM
   │
   ├── vmbr interfaces
   ├── Linux routes
   ├── VM configuration
   ├── LXC configuration
   ├── QEMU Guest Agent
   ├── ARP / neighbors
   └── manual reservations


No additional Web service is required.


INSTALLATION

PVE IPAM must be installed through:

- external SSH;
- physical server console;
- IPMI;
- iDRAC;
- iLO;
- another independent remote console.


DO NOT install the module from:

Proxmox Web UI
→ Node
→ Shell


The installer may restart Proxmox services such as:

pvedaemon
pveproxy


The Proxmox Web Shell may therefore disconnect during installation.


SUMMARY

PVE IPAM provides centralized IP address management directly inside Proxmox VE.

Main capabilities:

- automatic network discovery;
- manual network registration;
- support for routed networks;
- CIDR subnet management;
- VM/LXC IP inventory;
- free IP discovery;
- manual reservations;
- ARP / neighbor discovery;
- active network scanning;
- duplicate IP detection;
- MAC tracking;
- conflict detection;
- direct integration with the native Proxmox Web interface on port 8006.
