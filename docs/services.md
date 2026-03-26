# Service Inventory

Complete list of all self-hosted services across the homelab infrastructure.

---

## docker-infra (10.10.10.102)

Core infrastructure services.

| Service | Description |
|---------|-------------|
| Portainer | Docker container management web UI |
| Nginx Proxy Manager | Reverse proxy with automatic SSL certificate management |
| ntfy | Push notification server (self-hosted) |

## docker-apps (10.10.10.103)

Application services and developer tools.

| Service | Description |
|---------|-------------|
| Gitea | Self-hosted Git server (+ PostgreSQL database) |
| n8n | Workflow automation platform |
| Vaultwarden | Bitwarden-compatible password manager |
| Planka | Kanban project management board (+ PostgreSQL database) |
| Homepage | Dashboard for all services |
| Uptime Kuma | Uptime and endpoint monitoring |
| Code Server | VS Code in the browser |
| Stirling PDF | PDF manipulation toolkit |
| IT Tools | Collection of IT utility tools |
| Excalidraw | Collaborative whiteboard and diagramming |
| CyberChef | Data encoding/decoding Swiss Army knife |
| FileBrowser | Web-based file manager |
| Speedtest Tracker | Internet speed monitoring over time |
| Change Detection | Website change monitoring |
| Listmonk | Newsletter and mailing list manager |
| Mautic | Marketing automation platform (+ database) |
| Adminer | Lightweight database admin UI |
| ArchiSteamFarm | Steam idle/farming bot |
| Epic Games Free | Automatic Epic Games free game claimer |
| whoami | Debug/test container |

## docker-monitoring (10.10.10.104)

Observability and monitoring stack.

| Service | Description |
|---------|-------------|
| Grafana | Metrics dashboards and visualization |
| Prometheus | Time-series metrics collection and alerting |
| Loki | Log aggregation engine |
| Promtail | Log collector and shipper for Loki |
| Pi-hole Exporter | Prometheus exporter for Pi-hole statistics |

## Home Assistant (10.10.10.105)

| Service | Description |
|---------|-------------|
| Home Assistant | Home automation and smart home hub |

Runs directly in an LXC container (not Docker). Configuration stored in `/opt/homeassistant/config/`.

## docker-productivity (10.10.10.106)

Productivity, knowledge management, and automation tools.

| Service | Description |
|---------|-------------|
| n8n | Workflow automation (second instance, 35+ workflows) |
| Paperless-ngx | Document management with OCR (+ Redis) |
| Umami | Privacy-focused web analytics (+ PostgreSQL) |
| Obsidian CouchDB | Obsidian notes sync backend |
| Mealie | Recipe management and meal planning |
| Actual Budget | Personal finance and budgeting |
| BookStack | Wiki and documentation platform (+ MariaDB) |

## Coolify (10.10.10.107)

| Service | Description |
|---------|-------------|
| Coolify | PaaS platform for deploying applications |

## Supabase (10.10.10.108)

Self-hosted Backend-as-a-Service.

| Service | Description |
|---------|-------------|
| PostgreSQL | Relational database |
| PostgREST | RESTful API auto-generated from database schema |
| GoTrue | Authentication and user management |
| Storage | S3-compatible object storage |
| Realtime | WebSocket server for live subscriptions |
| Studio | Database management web UI |

Active database schemas: `craftio` (CRM), `tastly` (recipes), `signage` (digital signage), `jindrich` (client).

## docker-media-1 (10.10.10.109) -- Proxmox Node 1

Media consumption and management stack.

| Service | Description |
|---------|-------------|
| Jellyfin | Media server (movies, TV shows) |
| Sonarr | TV series download automation |
| Radarr | Movie download automation |
| Bazarr | Subtitle download automation |
| Prowlarr | Indexer manager for Sonarr/Radarr |
| Overseerr | Media request and discovery |
| qBittorrent | Torrent client |
| qBittorrent (VPN) | Torrent client routed through Gluetun VPN |

## docker-media-2 (10.10.10.110) -- Proxmox Node 2

Extended media, cloud storage, AI, and additional services.

| Service | Description |
|---------|-------------|
| Jellyfin | Media server (second instance) |
| Sonarr | TV series management |
| Radarr | Movie management |
| Bazarr | Subtitle management |
| Prowlarr | Indexer manager |
| Jellyseerr | Media request and discovery |
| Transmission | Torrent client |
| Nextcloud | Cloud storage and collaboration (+ MariaDB, Redis, OnlyOffice) |
| Immich | Photo and video management (+ PostgreSQL, ML, Redis) |
| Ollama | Local LLM inference server |
| Open WebUI | Web chat interface for Ollama |
| Navidrome | Music streaming server |
| Audiobookshelf | Audiobook and podcast server |
| Calibre-web | E-book library and reader |
| FreshRSS | RSS feed reader and aggregator |
| Memos | Lightweight note-taking |
| Linkwarden | Bookmark management (+ PostgreSQL) |
| Syncthing | Peer-to-peer file synchronization |
| Dozzle | Real-time Docker log viewer |
| Watchtower | Automatic Docker container updates |
| coturn | TURN/STUN server for WebRTC |

## docker-web (10.10.10.115) -- Proxmox Node 2

Production web hosting. 8 GB RAM limit with memory monitoring.

### SaaS Products

| Port | Service | Domain | Description |
|------|---------|--------|-------------|
| 3070 | CraftIO | craftio.ajtak.it | CRM platform (Next.js + Supabase) |
| 3071 | Gotenberg | -- | PDF generation service for CraftIO |
| 3110 | Sepot | sepot.cz | Encrypted messaging platform |
| 3115 | Tastly | tastly.cz | Recipe sharing platform |
| 3120 | Moje Obec | moje-obec.cz | Municipal information app |
| 3130 | Beamcast | beamcast.ajtak.it | Digital signage dashboard |
| 3131 | Beamcast Player | -- | Digital signage player (PWA) |
| 3180 | SVJko | svjko.cz | Property management for housing associations |

### Client Websites

| Port | Service | Domain | Description |
|------|---------|--------|-------------|
| 3095 | Dentiqa | dentiqa.ajtak.it | Dental clinic website |
| 3105 | EduConnect | educonnect.ajtak.it | Education platform website |
| 3135 | HTS Hasici | hasici.ajtak.it | Fire brigade website |
| 3140 | InexDesign | inex.ajtak.it | Window shading company website |
| 3145 | Jindrich Vcely | jindrich.joskovi.net | Beekeeper website |
| 3155 | Prodejny Zeman | prodejny.ajtak.it | Butcher shop chain website |
| 3150 | Zeman (old) | -- | Legacy website |

### B2B Marketing Websites

| Port | Service | Domain | Description |
|------|---------|--------|-------------|
| 3050 | Firemnisoft | firemnisoft.cz | Enterprise software services |
| 3055 | Vernostniaplikace | vernostniaplikace.cz | Loyalty app services |
| 3060 | Weboffka | weboffka.cz | Web development services |
| 3065 | Appkyprofirmy | appkyprofirmy.cz | Business app development |
| 3075 | Chciappku | chciappku.cz | App development services |
| 3080 | Vyvojaplikacinamiru | vyvojaplikacinamiru.cz | Custom app development |
| 3085 | Softnamiru | softnamiru.cz | Custom software development |
| 3090 | Vyvinuto | vyvinuto.cz | Software development services |

### Personal and Utility

| Port | Service | Domain | Description |
|------|---------|--------|-------------|
| 3100 | Hoppscotch | -- | API testing tool |
| 3125 | Julius Joska | juliusjoska.cz | Personal portfolio website |

## Shared Web Server -- CT 111 (10.10.10.111)

Nginx serving static sites (not Docker-based).

| Site | Description |
|------|-------------|
| ajtak.it | Company landing page |
| sepot.app | Sepot app download page |
| moje-obec | Municipal app page |
| tastly.net | Tastly alternate domain |

## VPS Services (46.225.69.128)

Services running on the Hetzner VPS jump host.

| Service | Description |
|---------|-------------|
| Gitea | Git server (mirror) |
| Vaultwarden | Password manager (accessible externally) |
| Uptime Kuma | External uptime monitoring |
| ntfy | Push notification relay |
| ArchiSteamFarm | Steam idle bot |
| cloudflared | Cloudflare Tunnel daemon |

## DNS Services

| Host | IP | Role |
|------|----|------|
| Pi-hole (primary) | 10.10.10.2 | DNS filtering, ad blocking, local DNS |
| Pi-hole (secondary) | 10.10.10.3 | Redundant DNS, failover |

## Other Services

| Host | IP | Service | Description |
|------|----|---------|-------------|
| SearXNG | 10.10.10.183 | SearXNG | Privacy-respecting meta search engine |
| AI Dev | 10.10.10.112 | Claude Code | AI development server (LXC on PXM2) |
| PBS | 10.10.10.200 | Proxmox Backup Server | VM/CT backup with deduplication |
| TrueNAS | -- | TrueNAS | ZFS NAS (QEMU VM on PXM2) |
| Windows Server | -- | Windows Server 2022 | Windows environment (QEMU VM on PXM2) |

---

## Service Count Summary

| Category | Count |
|----------|-------|
| Infrastructure | 6 |
| Development | 5 |
| Monitoring | 5 |
| Media | 18 |
| Productivity | 14 |
| Web Hosting | 24 |
| Security | 4 |
| Smart Home | 1 |
| AI / ML | 4 |
| DNS | 2 |
| Backup | 1 |
| **Total** | **84** |
