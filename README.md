###### [Homelab](https://linuxhandbook.com/homelab/)? it's a home server used as a laboratory for testing, developing or for home and functional usage.

# Homelab Setup

Personal homelab rig for media, storage, automation, and tinkering.
Primary host runs `Proxmox`, storage managed by `TrueNAS`. Rack idle ~150W.


## About

This is my personal homelab setup. Hopefully it sparks ideas for your own!
At the core is a Dell Precision T3620 customized for a quiet, efficient rack.

## Main Server ▸ Dell Precision T3620 (Proxmox)

Spec sheet

```
CPU    : Intel i7-7700
RAM    : 48 GB
GPU    : NVIDIA Quadro RTX 4000 (occasional gaming, LLMs)
TPU    : Coral USB (accelerates Frigate object detection)
PSU    : Cooler Master MWE 650 Bronze
Cooling: Added extra fan for airflow
Power  : ~65 W average

Storage Layout
  • 1 TB NVMe SSD (VM storage)
  • 3 × 256 GB SATA SSD
  • 4 TB 3.5" HDD (Frigate recording)
  • 4 × 4 TB 3.5" HDD (RAIDZ1 via SAS→SATA HBA, passed to TrueNAS VM)
```

Virtual Machines (VMs)
- [Home Assistant](https://www.home-assistant.io/)
- [TrueNAS](https://www.truenas.com/) for storage management
- Windows VM with (GPU passthrough) running:
  - [Steam](https://store.steampowered.com/?l=french)
- Pop!_OS VM with (GPU passthrough) running:
  - [Ollama](https://ollama.com/)
  - [Immich machine learning](https://docs.immich.app/guides/remote-machine-learning/)
  - [Open WebUi](https://docs.openwebui.com/) 
  - <img src="images/openwebui.png" alt="Description" style="box-shadow: 20px 20px 20px \#888;border-radius: 5px;" width="300" height="200">
  - [SearXNG](https://docs.searxng.org/)
  - [Redis](https://redis.io/docs/latest/operate/oss_and_stack/)
- Debian VM (website hosting)
  - [Nginx Proxy Manager](https://nginxproxymanager.com/): reverse proxy

Other Services (LXC)
- [Frigate](https://frigate.video/): real-time surveillance with Coral TPU. 
- <img src="images/frigate.png" alt="Description" style="box-shadow: 20px 20px 20px \#888;border-radius: 5px;" width="300" height="200" >
- [Jellyfin](https://jellyfin.org/): media streaming. 
- <img src="images/Jellyfin.jpg" alt="Description" style="box-shadow: 20px 20px 20px \#888;border-radius: 5px;" width="300" height="200">
- [Immich](https://immich.app/): AI photo management. 
- <img src="images/immich.png" alt="Description" style="box-shadow: 20px 20px 20px \#888;border-radius: 5px;" width="300" height="200">
- [NextcloudPi](https://ownyourbits.com/nextcloudpi/): self-hosted files/collab. 
- <img src="images/nextcloud.png" alt="Description" style="box-shadow: 20px 20px 20px \#888;border-radius: 5px;" width="300" height="200">
- [Gitea](https://gitea.io/en-us/): self-hosted Git.
- <img src="images/gitea.png" alt="Description" style="box-shadow: 0px 0px 10px \#888;border-radius: 5px;" width="300" height="200">
- [Docmost](https://docmost.com/docs/category/self-hosting/): an open-source collaborative wiki and documentation software. 
- <img src="images/docmost.png" alt="Description" style="box-shadow: 0px 0px 10px \#888;border-radius: 5px;" width="300" height="200"> 
- [karakeep](https://github.com/karakeep-app/karakeep): is a self-hostable bookmark-everything app with a touch of AI for the data hoarders out there.
- [Vikunja](https://vikunja.io/): Vikunja, the fluffy, open-source, self-hostable to-do app.
- [Pi-hole](https://pi-hole.net/): network-wide ad blocker
- [Nginx Proxy Manager](https://nginxproxymanager.com/): reverse proxy
- [Vaultwarden](https://github.com/dani-garcia/vaultwarden): password manager
- [MeTub](https://github.com/alexta69/metube): Web GUI for youtube-dl.

## Second Server ▸ Lenovo ThinkCenter 710q


```
CPU : Intel i3-7100T
RAM : 16 GB
Storage
  • 512 GB NVMe (OS)
  • 1 TB 2.5" HDD
```

Virtual Machines (VMs)
- [Proxmox Backup Server](https://www.proxmox.com/en/proxmox-backup-server)
- 3 vm [Ubuntu Server 24.04](https://cloud-images.ubuntu.com/releases/) as [K3s](https://k3s.io/) kube cluster.

## TrueNAS Backup Server ▸ HP ProDesk 400 G4


```
CPU : Intel i5-6500
RAM : 8 GB
Storage
  • 256 GB SSD (OS)
  • 3 × 2 TB 2.5" HDD (RAIDZ1)
```

## Raspberry Pi 4

```
RAM    : 4 GB
Storage: 128 GB SSD
```

Running Services (Docker)
- [Homer Dashboard](https://github.com/bastienwirtz/homer): A dead simple static homepage. 
- <img src="images/homer.png" alt="Description" style="box-shadow: 0px 0px 10px \#888;border-radius: 5px;" width="300" height="200"> 
- [Pi-hole](https://pi-hole.net/): network-wide ad blocker.
- [PiKVM](https://github.com/pikvm/pikvm): Raspberry Pi-based KVM (Keyboard-Video-Mouse).
- [CloudFlare DDNS](https://hub.docker.com/r/oznu/cloudflare-ddns/)
- [Dockpeek](https://github.com/dockpeek/dockpeek): self-hosted Docker dashboard for quick access to my containers.
- [Termix](https://github.com/Termix-SSH/Termix): Termix is web-based SSH terminal access.

## Raspberry Pi 5

```
RAM    : 4 GB
Storage: 120 GB SSD
```


## Second Raspberry Pi 4 as backup

```
RAM    : 4 GB
Storage: 128 GB SSD
```

## Networking Equipment

- Eaton Ellipse PRO 1200: UPS for core rack equipment
- UniFi Cloud Gateway Ultra — routing + management
- UniFi U6 LR — long-range Wi‑Fi 6 AP
- UniFi U6 IW — in‑wall Wi‑Fi 6 AP
- UniFi Switch Lite 16 PoE — PoE for APs and devices
- Netgear GS308PP (8‑port PoE) — PoE for devices like Raspberry Pi 5
