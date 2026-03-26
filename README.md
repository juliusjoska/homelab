[![Proxmox](https://img.shields.io/badge/Proxmox-2x_Nodes-E57000?style=flat-square&logo=proxmox&logoColor=white)](https://www.proxmox.com/)
[![Docker](https://img.shields.io/badge/Docker-60%2B_containers-2496ED?style=flat-square&logo=docker&logoColor=white)](https://www.docker.com/)
[![Grafana](https://img.shields.io/badge/Grafana-Monitoring-F46800?style=flat-square&logo=grafana&logoColor=white)](https://grafana.com/)
[![Cloudflare](https://img.shields.io/badge/Cloudflare-31_zones-F38020?style=flat-square&logo=cloudflare&logoColor=white)](https://www.cloudflare.com/)
[![Supabase](https://img.shields.io/badge/Supabase-Self--hosted-3FCF8E?style=flat-square&logo=supabase&logoColor=white)](https://supabase.com/)
[![Home Assistant](https://img.shields.io/badge/Home_Assistant-Smart_Home-18BCF2?style=flat-square&logo=homeassistant&logoColor=white)](https://www.home-assistant.io/)
[![Terraform](https://img.shields.io/badge/Terraform-IaC-844FBA?style=flat-square&logo=terraform&logoColor=white)](https://www.terraform.io/)
[![Ansible](https://img.shields.io/badge/Ansible-Config_Mgmt-EE0000?style=flat-square&logo=ansible&logoColor=white)](https://www.ansible.com/)
[![Tailscale](https://img.shields.io/badge/Tailscale-VPN_Mesh-005AFF?style=flat-square&logo=tailscale&logoColor=white)](https://tailscale.com/)
[![License: MIT](https://img.shields.io/badge/License-MIT-yellow.svg?style=flat-square)](LICENSE)

# Homelab

My self-hosted infrastructure running on two Proxmox nodes in a home environment. Everything is containerized, monitored, and backed up. Over 60 services across 20+ LXC containers and VMs, serving 20+ production websites, a full media stack, development tools, and smart home automation.

---

## Architecture

```
                               INTERNET
                                  |
                    +-------------+-------------+
                    |                           |
              Hetzner VPS                 Cloudflare
           46.225.69.128                  31 DNS zones
          +-------------+              Tunnel + CDN
          | Gitea        |                  |
          | Vaultwarden  |                  |
          | Uptime Kuma  |          +-------+-------+
          | ntfy         |          | terminal.ajtak.it
          | cloudflared  |          | ssh.ajtak.it
          | ASF          |          +-------+-------+
          +------+------+                  |
                 |                         |
          Tailscale mesh            Cloudflare Tunnel
          + Port forwarding         (cloudflared)
          (ports 2002-2222)                |
                 |                         |
    =============+=========================+==============
    |            HOME NETWORK  10.10.10.0/24             |
    |                                                    |
    |  +-------------------+    +----------------------+ |
    |  | PROXMOX 1 (PXM1)  |    | PROXMOX 2 (PXM2)    | |
    |  | Intel i5-7500T     |    | AMD Ryzen 5 PRO      | |
    |  | 16 GB RAM          |    | 32 GB RAM            | |
    |  |                    |    |                      | |
    |  | CT  Pi-hole (DNS)  |    | CT  Pi-hole 2 (DNS) | |
    |  | CT  Docker Infra   |    | CT  Docker Media 2  | |
    |  | CT  Docker Apps    |    | CT  Docker Web       | |
    |  | CT  Docker Mon.    |    | CT  AI Dev Server    | |
    |  | CT  Home Assistant |    | CT  PBS (Backup)     | |
    |  | CT  Docker Prod.   |    | VM  TrueNAS          | |
    |  | CT  Coolify        |    | VM  Windows Server   | |
    |  | CT  Supabase       |    +----------------------+ |
    |  | CT  Docker Media 1 |                            |
    |  | CT  Shared Web     |                            |
    |  +-------------------+                             |
    |                                                    |
    |  DNS: Pi-hole primary + secondary (redundant)      |
    |  VPN: Tailscale mesh + WireGuard                   |
    |  Backup: PBS + restic + Hetzner Storage Box         |
    =====================================================
```

## Hardware

| Node | CPU | RAM | Storage | Role |
|------|-----|-----|---------|------|
| Proxmox 1 | Intel i5-7500T (4C/4T) | 16 GB | SSD | Containers, Apps, Monitoring |
| Proxmox 2 | AMD Ryzen 5 PRO 2400G (4C/8T) | 32 GB | SSD + HDD | VMs, Media, Web Hosting |
| Hetzner VPS | Shared vCPU | 4 GB | 40 GB SSD | Jump host, Reverse proxy |
| Proxmox Backup Server | -- | -- | HDD | Incremental VM/CT backups |
| TrueNAS | -- | -- | HDD pool | NAS, ZFS storage |
| Hetzner Storage Box | -- | -- | Remote | Offsite encrypted backups |

## Services (60+)

A summary of all self-hosted services. For a complete list with ports and details, see [docs/services.md](docs/services.md).

### Infrastructure

| Service | Purpose |
|---------|---------|
| Proxmox VE (x2) | Hypervisor / virtualization |
| Pi-hole (x2) | DNS filtering, ad blocking (primary + secondary) |
| Nginx Proxy Manager | Reverse proxy with automatic SSL |
| Portainer | Docker container management UI |
| Cloudflare Tunnels | Secure external access without port forwarding |
| Tailscale | Mesh VPN across all nodes + VPS |

### Development

| Service | Purpose |
|---------|---------|
| Gitea | Self-hosted Git server |
| Supabase | Backend-as-a-Service (PostgreSQL, Auth, Storage, Realtime) |
| Code Server | VS Code in the browser |
| Hoppscotch | API testing and development |
| Coolify | PaaS platform for app deployments |

### Monitoring and Observability

| Service | Purpose |
|---------|---------|
| Grafana | Dashboards and visualization |
| Prometheus | Metrics collection and alerting |
| Loki + Promtail | Centralized log aggregation |
| Uptime Kuma | Uptime and endpoint monitoring |
| Dozzle | Real-time Docker log viewer |
| Watchtower | Automatic container updates |

### Media

| Service | Purpose |
|---------|---------|
| Jellyfin (x2) | Media server (movies, TV shows) |
| Sonarr | TV series management |
| Radarr | Movie management |
| Bazarr | Subtitle management |
| Prowlarr | Indexer manager for the *arr stack |
| Jellyseerr / Overseerr | Media request management |
| Navidrome | Music streaming server |
| Audiobookshelf | Audiobook and podcast server |
| Calibre-web | E-book library |
| qBittorrent | Torrent client (with VPN via Gluetun) |

### Productivity

| Service | Purpose |
|---------|---------|
| Nextcloud | Cloud storage, file sync, collaboration |
| Immich | Photo and video management (Google Photos alternative) |
| Paperless-ngx | Document management and OCR |
| n8n (x2) | Workflow automation (35+ workflows) |
| BookStack | Wiki and documentation |
| Mealie | Recipe manager |
| FreshRSS | RSS feed reader |
| Actual Budget | Personal finance and budgeting |
| Memos | Lightweight note-taking |
| Linkwarden | Bookmark manager |
| Syncthing | Peer-to-peer file synchronization |

### Security

| Service | Purpose |
|---------|---------|
| Vaultwarden | Bitwarden-compatible password manager |
| fail2ban | Intrusion prevention (SSH, web) |
| UFW | Host-level firewall |
| Tailscale / WireGuard | Encrypted VPN tunnels |

### Smart Home

| Service | Purpose |
|---------|---------|
| Home Assistant | Home automation hub |
| Zigbee integration | IoT device control |

### AI and Machine Learning

| Service | Purpose |
|---------|---------|
| Ollama | Local LLM inference (multiple models) |
| Open WebUI | Chat interface for Ollama |
| SearXNG | Privacy-respecting meta search engine |
| Claude Code | AI-powered development server |

### Web Hosting

20+ production websites hosted on Docker with Cloudflare Tunnels providing HTTPS:

- **SaaS products** -- CraftIO CRM, Beamcast (digital signage), Tastly (recipes), SVJko (property management)
- **Client websites** -- Custom-built sites for local businesses
- **B2B marketing sites** -- 8 lead-generation websites
- **Personal portfolio** -- juliusjoska.cz

## Networking

```
Internet traffic
       |
       v
  Cloudflare (DNS + CDN + WAF)
       |
       +---> Cloudflare Tunnel ---> cloudflared (ai-dev) ---> NPM ---> services
       |
       +---> VPS (46.225.69.128)
                |
                +---> Port forwarding (2000 + last_octet) ---> all hosts
                |
                +---> Tailscale mesh ---> home network
```

- **Subnet:** `10.10.10.0/24`
- **DNS:** Pi-hole primary (10.10.10.2) + secondary (10.10.10.3), redundant
- **VPN:** Tailscale mesh network connecting all nodes + VPS
- **External access:** Cloudflare Tunnels for HTTPS, VPS port forwarding for SSH
- **Reverse proxy:** Nginx Proxy Manager with automatic Let's Encrypt SSL
- **Domains:** 31 Cloudflare zones, 20+ live production websites

For detailed network architecture, see [docs/architecture.md](docs/architecture.md).

## Backup Strategy

```
Proxmox VMs/CTs ----> Proxmox Backup Server (local, incremental, deduplicated)
                            |
                            +----> Hetzner Storage Box (offsite, encrypted)

Docker volumes -----> restic (incremental, encrypted) ----> Hetzner Storage Box

Supabase (PostgreSQL) ----> pg_dump (scheduled) ----> Storage Box

AI Dev Server ------> restic ----> Storage Box
```

- **Proxmox Backup Server** -- incremental, deduplicated VM/CT snapshots with verification
- **restic** -- encrypted incremental backups for Docker volumes and critical data
- **Hetzner Storage Box** -- offsite backup destination (SFTP, port 23)
- **Automated schedules** -- cron jobs for all backup tasks
- **Backup verification** -- PBS verify jobs, restic check

## Monitoring Stack

```
+------------+      +-----------+      +---------+      +------+
| Exporters  | ---> | Prometheus| ---> | Grafana | ---> | ntfy |
| (node, ct) |      | (metrics) |      | (viz)   |      | push |
+------------+      +-----------+      +---------+      +------+

+----------+      +----------+      +---------+
| Containers| --> | Promtail | ---> |  Loki   | ---> Grafana
| (stdout)  |     | (shipper)|      | (logs)  |
+----------+      +----------+      +---------+
```

- Prometheus scrapes metrics from all nodes and services
- Grafana provides dashboards and visualization
- Loki + Promtail handle centralized log aggregation
- Alerting via ntfy push notifications
- Uptime Kuma monitors all public endpoints
- Custom healthcheck scripts run every 30 minutes

## Automation

| Tool | Purpose |
|------|---------|
| n8n | 35+ workflow automations (monitoring, notifications, maintenance) |
| Ansible | Server provisioning and configuration management |
| Terraform | Infrastructure as Code for cloud resources |
| Shell scripts | Maintenance tasks, healthchecks, memory monitoring |
| Claude Code | AI-powered development and infrastructure management |
| cron | Scheduled backups, healthchecks, cleanup jobs |

Key automations include:

- Infrastructure healthcheck every 30 minutes (ping all hosts, check services, disk usage)
- Memory monitoring on resource-constrained hosts with automatic service restart
- Automated container updates via Watchtower
- Evening infrastructure reports
- Network scanning and security monitoring

## Tech Stack

| Category | Technologies |
|----------|-------------|
| Virtualization | Proxmox VE, LXC, QEMU/KVM |
| Containers | Docker, Docker Compose |
| Web | Next.js 15/16, Tailwind CSS, shadcn/ui |
| Backend | Supabase (PostgreSQL, Auth, Storage), Node.js |
| Mobile | Flutter, Dart, Riverpod |
| Monitoring | Grafana, Prometheus, Loki, Promtail |
| Networking | Cloudflare, Tailscale, WireGuard, Nginx |
| IaC | Terraform, Ansible |
| Backup | Proxmox Backup Server, restic |
| DNS | Pi-hole, Cloudflare |
| AI/ML | Ollama, Claude Code, SearXNG |

## Repository Structure

```
homelab/
  README.md              -- This file
  docs/
    architecture.md      -- Detailed network and system architecture
    services.md          -- Complete service inventory with ports
  LICENSE                -- MIT License
```

## License

This project is licensed under the MIT License. See [LICENSE](LICENSE) for details.

---

*Built and maintained as a personal project. This repository serves as documentation and a showcase -- it does not contain secrets, credentials, or deployment scripts.*
