# Architecture

Detailed architecture documentation for the homelab infrastructure.

---

## Network Topology

```
                                 INTERNET
                                    |
                 +------------------+------------------+
                 |                                     |
          Hetzner VPS                           Cloudflare
       46.225.69.128                         CDN + WAF + DNS
     +-----------------+                    31 managed zones
     | Ubuntu Server   |                         |
     | Tailscale       |                  Cloudflare Tunnel
     | fail2ban + UFW  |               (731d9362-404f-4428-...)
     | Docker services |                         |
     | Port forwarding |                         |
     +--------+--------+                         |
              |                                  |
     Tailscale mesh (100.118.189.28)             |
     + SSH port forwarding                       |
     (port = 2000 + last octet)                  |
              |                                  |
  ============+==================================+=============
  |                HOME NETWORK                               |
  |              10.10.10.0/24                                |
  |                                                           |
  |    +---------------------+     +------------------------+ |
  |    |   PROXMOX NODE 1    |     |    PROXMOX NODE 2      | |
  |    |   10.10.10.100      |     |    10.10.10.101         | |
  |    |   Intel i5-7500T    |     |    AMD Ryzen 5 PRO      | |
  |    |   16 GB RAM         |     |    32 GB RAM            | |
  |    |                     |     |                         | |
  |    |   13 LXC containers |     |    6 LXC + 2 QEMU VMs  | |
  |    +---------------------+     +------------------------+ |
  |                                                           |
  |    DNS: Pi-hole x2 (10.10.10.2 + 10.10.10.3)            |
  |    Backup: PBS (10.10.10.200)                            |
  |    VPN: Tailscale mesh across all nodes                  |
  =============================================================
```

## Proxmox Node 1 (PXM1) -- 10.10.10.100

Intel i5-7500T, 16 GB RAM, SSD storage. Primary node for application workloads.

| VMID | Type | Hostname | IP | Purpose |
|------|------|----------|----|---------|
| 101 | LXC | pihole | 10.10.10.2 | Primary DNS (Pi-hole) |
| 102 | LXC | docker-infra | 10.10.10.102 | Infrastructure services |
| 103 | LXC | docker-apps | 10.10.10.103 | Application services |
| 104 | LXC | docker-monitoring | 10.10.10.104 | Monitoring stack |
| 105 | LXC | homeassistant | 10.10.10.105 | Home automation |
| 106 | LXC | docker-productivity | 10.10.10.106 | Productivity tools |
| 107 | LXC | coolify | 10.10.10.107 | PaaS platform |
| 108 | LXC | supabase | 10.10.10.108 | Self-hosted BaaS |
| 109 | LXC | docker-media-1 | 10.10.10.109 | Media stack (instance 1) |
| 110 | LXC | searxng | 10.10.10.183 | Meta search engine |
| 111 | LXC | shared-web | 10.10.10.111 | Nginx (static sites) |
| 112 | LXC | narezak | -- | Isolated (no network) |
| 200 | LXC | monitoring | (shared) | System monitoring |

## Proxmox Node 2 (PXM2) -- 10.10.10.101

AMD Ryzen 5 PRO 2400G, 32 GB RAM, SSD + HDD. Secondary node for VMs, media, and web hosting.

| VMID | Type | Hostname | IP | Purpose |
|------|------|----------|----|---------|
| 100 | QEMU | win22 | -- | Windows Server (RDP) |
| 101 | LXC | pihole-secondary | 10.10.10.3 | Secondary DNS (Pi-hole) |
| 102 | QEMU | truenas | -- | TrueNAS (ZFS NAS) |
| 110 | LXC | docker-media-2 | 10.10.10.110 | Media stack (instance 2) |
| 115 | LXC | docker-web | 10.10.10.115 | Web hosting (20+ sites) |
| 120 | LXC | ai-dev | 10.10.10.112 | AI development server |
| 200 | LXC | pbs | 10.10.10.200 | Proxmox Backup Server |

## DNS Architecture

```
Client device
      |
      v
  Pi-hole Primary (10.10.10.2)  <---failover--->  Pi-hole Secondary (10.10.10.3)
      |                                                |
      +---> Local DNS records (homelab domains)        |
      |                                                |
      +---> Ad/tracker blocking (blocklists)           |
      |                                                |
      +---> Upstream: Cloudflare (1.1.1.1) / Google (8.8.8.8)
```

- All devices on the network use Pi-hole as their DNS server
- Pi-hole primary and secondary provide redundancy
- Local DNS entries resolve internal hostnames to 10.10.10.x addresses
- Ad and tracker blocking via community-maintained blocklists
- Upstream resolution via Cloudflare DNS (1.1.1.1) and Google DNS (8.8.8.8)

## External Access

### Cloudflare Tunnels

The primary method for accessing internal services from the internet. A `cloudflared` daemon runs on the AI dev server and establishes an outbound-only tunnel to Cloudflare's edge network.

```
User (browser)
      |
      v
  Cloudflare Edge (HTTPS)
      |
  Cloudflare Access (email OTP)
      |
  Cloudflare Tunnel
      |
      v
  cloudflared (10.10.10.112) ---> Nginx Proxy Manager ---> target service
```

Configured tunnel hostnames:
- `terminal.ajtak.it` -- SSH access to AI dev server
- `ssh.ajtak.it` -- SSH access (alternate)

### VPS Port Forwarding

For direct SSH access to any host from the internet, the VPS runs `socat` forwarders as systemd services.

```
Formula: port = 2000 + last_octet_of_IP

Example: Pi-hole (10.10.10.2) -> ssh -p 2002 root@46.225.69.128
Example: AI Dev  (10.10.10.112) -> ssh -p 2222 root@46.225.69.128
```

Each forward runs as `ssh-forward-<name>.service` on the VPS.

### Tailscale Mesh

All nodes participate in a Tailscale mesh network, providing encrypted peer-to-peer connectivity regardless of NAT or firewall configuration.

- VPS Tailscale IP: `100.118.189.28`
- Used for reliable connectivity between VPS and home network
- MagicDNS for hostname resolution across the mesh

## Reverse Proxy Flow

```
Cloudflare (HTTPS, SSL termination at edge)
      |
      v
Cloudflare Tunnel (encrypted)
      |
      v
cloudflared (ai-dev, 10.10.10.112)
      |
      v
Nginx Proxy Manager (10.10.10.102)
      |
      +---> craftio.ajtak.it       -> docker-web:3070
      +---> beamcast.ajtak.it      -> docker-web:3130
      +---> tastly.cz              -> docker-web:3115
      +---> dentiqa.ajtak.it       -> docker-web:3095
      +---> educonnect.ajtak.it    -> docker-web:3105
      +---> juliusjoska.cz         -> docker-web:3125
      +---> (20+ more routes)
      +---> grafana.ajtak.it       -> docker-monitoring:3000
      +---> portainer.ajtak.it     -> docker-infra:9000
      +---> ...
```

SSL certificates are managed by Cloudflare (edge termination). Internal traffic between Cloudflare Tunnel and services uses HTTP within the trusted home network.

## Backup Architecture

### Tier 1: Proxmox Backup Server (local)

```
Proxmox Node 1 ---+
                  |---> PBS (10.10.10.200)
Proxmox Node 2 ---+         |
                            +---> Incremental, deduplicated
                            +---> Automatic verification jobs
                            +---> Garbage collection
                            +---> Email notifications on failure
```

- All VMs and LXC containers backed up to PBS
- Incremental and deduplicated -- only changed blocks transferred
- Verification jobs validate backup integrity
- Retention policy configured per datastore

### Tier 2: restic (encrypted, incremental)

```
Docker volumes --------+
AI dev server config ---+---> restic ---> Hetzner Storage Box
Critical databases -----+               (u540029.your-storagebox.de:23)
```

- restic provides encrypted, incremental backups
- Destination: Hetzner Storage Box (offsite, SFTP)
- Covers Docker volumes, configuration files, databases

### Tier 3: Database dumps

```
Supabase (PostgreSQL)
      |
      v
  pg_dump (scheduled via cron)
      |
      v
  Local storage ---> restic ---> Storage Box
```

- PostgreSQL dumps for Supabase schemas (craftio, tastly, signage, etc.)
- Scheduled via cron jobs
- Included in restic backup rotation

## Monitoring Architecture

```
+------------------+     +------------------+     +------------------+
|  Node Exporters  |     |    Promtail      |     |  Custom Scripts  |
|  (all hosts)     |     |  (log shipper)   |     |  (healthcheck)   |
+--------+---------+     +--------+---------+     +--------+---------+
         |                        |                        |
         v                        v                        v
  +-----------+            +----------+             +-----------+
  | Prometheus|            |   Loki   |             |  Log file |
  | (metrics) |            |  (logs)  |             | + cron    |
  +-----+-----+            +----+-----+             +-----+-----+
        |                       |                         |
        +----------+------------+                         |
                   |                                      |
                   v                                      |
            +-----------+                                 |
            |  Grafana  | <-------------------------------+
            | (viz+alert)|
            +-----+-----+
                  |
                  v
            +-----------+
            |   ntfy    |
            |  (push)   |
            +-----------+
                  |
                  v
            Mobile push
            notifications
```

### Components

| Component | Host | Port | Role |
|-----------|------|------|------|
| Prometheus | docker-monitoring | 9090 | Metrics collection, PromQL queries |
| Grafana | docker-monitoring | 3000 | Dashboards, alerting |
| Loki | docker-monitoring | 3100 | Log aggregation |
| Promtail | docker-monitoring | 9080 | Log shipping |
| ntfy | docker-infra | 80 | Push notifications |
| Uptime Kuma | docker-apps | 3001 | Endpoint monitoring |
| Pi-hole Exporter | docker-monitoring | 9617 | Pi-hole metrics for Prometheus |

### Healthcheck System

A custom healthcheck script runs every 30 minutes via cron on the AI dev server:

1. Pings all hosts in the network
2. Checks cloudflared tunnel status
3. Verifies VPS socat port forwarding
4. Monitors disk usage across all nodes
5. Logs results to `/var/log/infra-healthcheck.log`

## Security Layers

```
Layer 1: Cloudflare WAF + DDoS protection (edge)
Layer 2: fail2ban on VPS (SSH brute force prevention)
Layer 3: UFW firewall rules (per-host)
Layer 4: Cloudflare Access (email OTP for sensitive services)
Layer 5: Tailscale ACLs (mesh VPN access control)
Layer 6: SSH key-only authentication (no passwords)
Layer 7: Secrets management (~/.secrets/.env, chmod 600)
```

- No services exposed directly to the internet (everything behind Cloudflare or VPN)
- SSH access requires key authentication -- password auth disabled
- VPS whitelists home IP in fail2ban
- Sensitive admin panels protected by Cloudflare Access (email OTP)
- Secrets stored in encrypted files, never committed to repositories

## Docker Host Layout

Each Docker host runs a focused set of services to maintain separation of concerns and limit blast radius.

| Host | IP | Focus | RAM |
|------|-----|-------|-----|
| docker-infra | 10.10.10.102 | Core infrastructure (proxy, portainer) | Standard |
| docker-apps | 10.10.10.103 | Applications and tools | Standard |
| docker-monitoring | 10.10.10.104 | Observability stack | Standard |
| docker-productivity | 10.10.10.106 | Productivity and knowledge | Standard |
| docker-media-1 | 10.10.10.109 | Media stack (PXM1) | Standard |
| docker-media-2 | 10.10.10.110 | Media stack (PXM2) + Nextcloud + AI | Standard |
| docker-web | 10.10.10.115 | 20+ production websites | 8 GB (monitored) |

Docker-web has a memory monitoring script that runs every 5 minutes and automatically restarts services if swap usage exceeds 80%.
