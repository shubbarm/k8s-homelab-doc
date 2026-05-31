# Home Lab Documentation
A complete, production‑style overview of my Kubernetes‑based home lab environment.
This documentation covers hardware, networking, cluster architecture, storage, applications, ingress, and operational patterns — all based on the live system.

## Hardware & Nodes
My cluster runs on three physical machines with mixed CPU architectures, storage tiers, and performance profiles. This diversity allows me to test scheduling, affinity rules, and multi‑arch container workloads.

### Control Plane — box178
Model: HP EliteDesk 800 G3 DM
CPU: Intel Core i5‑6500 (4 cores, 3.6 GHz boost)
RAM: 8 GB DDR4
Storage:
256 GB Samsung NVMe (OS + K3s datastore)
Network: Intel I219‑LM (1 Gbps)
OS: Ubuntu 24.04.4 LTS
Kernel: 6.8.0‑111
IP: 192.168.0.178
Role: Kubernetes control‑plane, runs critical system pods
Container Runtime: containerd 2.1.5

### Worker Node 1 — desktopbox
Model: HP h8‑1419
CPU: Intel Core i7‑3770 (4 cores / 8 threads, 3.9 GHz boost)
RAM: 12 GB DDR3
Storage:
240 GB Kingston SSD (OS)
3.6 TB Seagate HDD (/mnt/backup1)
931 GB WD Black HDD (/mnt/k8s-data)
Network: Atheros AR8161 (1 Gbps)
OS: Ubuntu 24.04.4 LTS
Kernel: 6.8.0‑110
IP: 192.168.0.179
Role: Primary workload node (media, cloud, DNS, ingress, databases)
Container Runtime: containerd 2.1.5

### Worker Node 2 — pibox
Model: Raspberry Pi 400
CPU: Broadcom BCM2711 (Cortex‑A72, quad‑core, 1.8 GHz)
RAM: 4 GB LPDDR4
Storage:
500 GB USB SSD
Network: Gigabit Ethernet (100 Mbps negotiated)
OS: Ubuntu 24.04.4 LTS (ARM64)
Kernel: 6.8.0‑1057‑raspi
IP: 192.168.0.177
Role: ARM worker node (media workloads, torrenting, lightweight services)
Container Runtime: containerd 2.1.5

## Network Architecture
A flat home network with Kubernetes overlay networking and MetalLB for bare‑metal load balancing.

LAN
Subnet: 192.168.0.0/24
Gateway: 192.168.0.7
DNS: systemd‑resolved (127.0.0.53)
DHCP: Router‑managed (Pi node receives secondary DHCP IP)
Kubernetes Pod Networking (Flannel VXLAN)

Each node receives a unique pod CIDR:

box178	10.42.0.0/24
desktopbox	10.42.1.0/24
pibox	10.42.5.0/24

VXLAN Details
Interface: flannel.1
Port: 8472/UDP
MTU: 1450
Encapsulation: VXLAN ID 1
This provides a stable overlay network across mixed hardware.

Load Balancing (MetalLB)
MetalLB assigns real LAN IPs to services:
Traefik	192.168.0.80
AdGuard	192.168.0.85 / Home DNS filteration
AdGuard‑2	192.168.0.86 / Backup DNS instance on a different node

This allows direct access to services without NodePorts.


## Cluster Architecture
A lightweight, multi‑architecture Kubernetes cluster powered by K3s.

Cluster Distribution
K3s v1.34.3+k3s1
Container Runtime: containerd
Control Plane: 1 node
Workers: 2 nodes (x86_64 + ARM64)

Core System Components
Traefik — Ingress controller
MetalLB — Bare‑metal load balancer
Cert‑Manager — Automatic TLS
CoreDNS — Cluster DNS
Metrics Server — Resource metrics
Local Path Provisioner — Default storage class

Ingress Routing
All apps are exposed through Traefik using my domain:

Examples:
ha.mustafa.ca
watch.mustafa.ca
drive.mustafa.ca
lens.mustafa.ca
plex.mustafa.ca
pulse.mustafa.ca

Traefik LoadBalancer IP: 192.168.0.80

## Storage & Backups
A hybrid storage model combining local PVs, NFS‑backed RWX volumes, and Velero backups.

Storage Classes
local-path (default)
Reclaim Policy: Delete
Binding Mode: WaitForFirstConsumer
Persistent Volumes
Your PVs fall into three categories:

1. Local PVs (manual hostPath)
Used for:

Postgres
Uptime Kuma
Grafana
AdGuard

2. NFS‑backed RWX Volumes
Hosted on desktopbox, used for:

Plex (500 GiB)
Nextcloud (100 GiB)
Jellyfin (20 GiB)
Media (500 GiB)
Downloads (500 GiB)

These allow multi‑pod access and large storage capacity.

3. Application‑specific PVs

AdGuard config/work
Immich storage

## Backups (Velero)
Velero deployment + node agents
Weekly metadata backup (30 0 * * 0)

## Monitoring & Observability

Lightweight monitoring only — intentionally minimal.

### Uptime Kuma
StatefulSet

Runs on control plane

Tracks service uptime and HTTP endpoints


### cupdate

Custom update‑checker service

Runs on worker node 1


## Services & Applications

All workloads grouped by namespace and purpose.

Networking & DNS
AdGuard Home (two deployments)

DNS filtering + DHCP replacement

Home Automation
Home Assistant

Exposed via Traefik (ha.mustafa.ca)

Media & Downloads
Jellyfin — Media server

Plex — Media server

qBittorrent + Gluetun — Torrenting behind VPN

Metube — YouTube downloader

Personal Cloud
Nextcloud — File sync + calendar + contacts

Photo Management
Immich — Full stack (server, ML, Redis, Postgres)

Databases
Postgres (StatefulSet)

Adminer (ImagePullBackOff)

Backups
Velero + node agents

MinIO (failing image pull)

Monitoring
Uptime Kuma

cupdate

## Experiments & Learning
This lab is used for hands‑on learning with:

Kubernetes operations

Multi‑architecture scheduling

Ingress routing & TLS

Persistent storage (local + NFS)

MetalLB load balancing

Home automation

Media streaming

Backup workflows

Lightweight monitoring

Running real apps in a cluster

## Summary
This home lab demonstrates practical experience with:

Linux system administration

Kubernetes (K3s)

Networking & overlay networks

Ingress + LoadBalancer setups

Persistent storage (local + NFS)

Running real applications in a cluster

Lightweight monitoring

Backup tooling

Multi‑node, multi‑architecture clusters

A compact, real‑world environment designed for learning, experimentation, and self‑hosting.



A “Future Improvements” section

Just tell me which one you want next.
