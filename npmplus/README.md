# NPMplus + CrowdSec WAF

I replaced the external Nginx Proxy Manager on my Debian VM (website hosting) with **NPMplus + CrowdSec AppSec** to strengthen homelab security.

## Architecture

```mermaid
flowchart TB
    Internet["Internet"] --> CF["Cloudflare CDN / DNS"]
    CF --> UCG["UniFi Cloud Gateway Ultra"]

    subgraph EDGE["Security Zone - VLAN 4"]
        NPM["NPMplus - Reverse Proxy"]
        CS["CrowdSec - WAF / AppSec / Bouncer"]
        NPM <--> CS
        Websites["Hugo / Portfolio / Web Apps"]
        NPM --> Websites
    end

    UCG --> NPM
    NPM --> NPM2

    subgraph LAN["Internal Network - VLAN 1"]
        NPM2["NPM2 - Internal Reverse Proxy"]

        subgraph SERVICES["Applications - Proxmox VMs and LXCs"]
            HA["Home Assistant"]
            NC["Nextcloud"]
            VW["Vaultwarden"]
            JF["Jellyfin"]
            GIT["Gitea"]
            IMM["Immich - Private"]
            FG["Frigate + Coral TPU"]
        end

        subgraph AI["Debian AI VM - RTX 4000"]
            OLL["Ollama"]
            OWUI["Open WebUI"]
            OWUI --> OLL
        end

        subgraph STORAGE["TrueNAS SCALE"]
            HOME["Home Data - RAIDZ1 4x4TB"]
            BACK["Backup Pool - RAIDZ1 3x4TB"]
        end

        WIN["Windows 11 VM"]
    end

    NPM2 --> HA & NC & VW & JF & GIT
    NC -->|"Photo storage"| HOME
    JF -->|"Media library"| HOME
    IMM -->|"Read-only photo library"| HOME
    HA -->|"AI requests"| OLL
    FG -->|"AI requests"| OLL
    HOME -.->|"Replication"| BACK

    subgraph INFRA["Infrastructure and Management"]
        PVE["Dell Precision T3620 - Main Proxmox"]
        LEN["Lenovo M710q - Secondary Node"]
        DNS["2x Pi-hole + dnscrypt-proxy"]
        UPS["Raspberry Pi 5 - NUT UPS"]
        SW["Network Switches + UniFi APs"]
    end

    UCG --> SW
    UCG -.-> DNS
    UCG -.->|"WireGuard VPN"| IMM
    PVE -.-> SERVICES
    PVE -.-> AI
    PVE -.-> STORAGE
    PVE -.-> WIN
    UPS -.-> PVE

    classDef security fill:#e6f4ea,stroke:#2d8659,color:#164b35
    classDef proxy fill:#e7f0fc,stroke:#477fc0,color:#163d6b
    classDef storage fill:#fff2df,stroke:#c38a2c,color:#754b12
    class NPM,CS security
    class UCG,NPM2 proxy
    class HOME,BACK storage
```

Requests pass through Cloudflare and the UniFi Gateway before reaching the external reverse proxy. NPMplus handles proxying, while CrowdSec AppSec provides web application firewall inspection at this entry point.

NPMplus serves the websites in the security zone (VLAN 4) and forwards application requests to NPM2 in the internal network (VLAN 1). NPM2 routes those requests to the appropriate homelab service. Private Immich access uses WireGuard VPN.

[Back to the homelab overview](../README.md)
