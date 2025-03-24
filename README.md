# Home Lab Setup

This is my personal homelab setup, awakening my home with various services and tools. I hope this setup can inspire you as you build your own homelab!

At the heart of it is an Dell Precision T3620, customized to handle everything from media streaming and file storage to home automation and security. Over the last 8 months, I’ve fine-tuned this setup for efficiency, using Proxmox as the main OS and TrueNAS for managing storage. The entire rack idles at around 90W, balancing performance and power consumption to keep everything running smoothly.

<img src="https://github.com/djamelinfo/myhomelab/raw/main/images/IMG_20241120_105814773.jpg" alt="Rack Setup" width="600"/>

## 1. Main Server: Dell Precision T3620 running Proxmox

<img src="https://github.com/djamelinfo/myhomelab/raw/main/images/Screenshot_20241101_135806_Chrome.jpg" alt="Proxmox" width="600"/>

- **Hardware**:
  - **CPU**:  Intel i7-7700
  - **RAM**: 48GB
  - **Storage Setup**:
    - 1TB NVMe SSD for VM storage
    - Three 256GB SATA SSDs
    - 2TB 2.5" HDD
    - Four 4TB 3.5" HDDs in RAIDZ1, connected via a SAS to SATA HBA card passed through to the [**TrueNAS**](https://www.truenas.com/) VM for storage management.
  - **Coral USB TPU**: For accelerating machine learning tasks (e.g., object detection in Frigate)
  - **GPU**: NVIDIA Quadro RTX 4000 (occasional gaming, and running LLMs)
  - **PSU**: Cooler Master MWE 650 Bronze
  - Cooling: Added an extra fan for improved airflow
  - Power Consumption: Averages around 65W
- **Virtual Machines (VMs)**:
  - [**Home Assistant**](https://www.home-assistant.io/): Home automation platform

    <img src="https://github.com/djamelinfo/myhomelab/raw/main/images/Homeassistat.jpg" alt="HomeAssistant" width="400"/>

  - [**TrueNAS**](https://www.truenas.com/) VM for storage management

    <img src="https://github.com/djamelinfo/myhomelab/raw/main/images/Screenshot_20241102_173951_Chrome.jpg" alt="TrueNas" width="400"/>

  - **Windows VM** for gaming with GPU passthrough
  - **Pop!_OS VM** for running an Olama model
- **Other services (LXC)**:
    - [**Frigate**](https://frigate.video/): Real-time surveillance with object detection, accelerated by Coral USB TPU
      
      <img src="https://github.com/djamelinfo/myhomelab/raw/main/images/Frigat.jpg" alt="Frigate" width="400"/>

    - [**Jellyfin**](https://jellyfin.org/): Media streaming server

      <img src="https://github.com/djamelinfo/myhomelab/raw/main/images/Jellyfin.jpg" alt="Jellyfin" width="400"/>

    - [**PhotoPrism**](https://photoprism.app/): AI-based photo management
    - [**Nginx**](https://www.nginx.com/): Web server and reverse proxy
    - [**NextcloudPi**](https://ownyourbits.com/nextcloudpi/): Self-hosted file sharing and collaboration platform
    - [**Vaultwarden**](https://github.com/dani-garcia/vaultwarden): Self-hosted password manager
    - [**Gitea**](https://gitea.io/en-us/): Self-hosted Git service
    - [**Pi-hole**](https://pi-hole.net/): Network-wide ad blocker

      <img src="https://github.com/djamelinfo/myhomelab/raw/main/images/pihole.jpg" alt="Pihole" width="400"/>



## 2. Second Server: Lenovo ThinkCenter 710Q
- **CPU**: Intel i3-7100T
- **RAM**: 16GB
- **Storage**:
  - 1TB NVMe for OS
  - 1TB 2.5" HDD
- **Virtual Machines (VMs)**:
  - [**Proxmox Backup Server**](https://www.proxmox.com/en/proxmox-backup-server): For managing backups

## 3. Raspberry Pi 4

<img src="https://github.com/djamelinfo/myhomelab/raw/main/images/IMG_20241105_160810802.jpg" alt="rpi4" width="300"/>

- **RAM**: 4GB
- **Storage**: 64GB SD Card
- **Running Services in Docker**:
  - [**Homer Dashboard**](https://github.com/bastienwirtz/homer): Docker containers, including a customizable dashboard

    <img src="https://github.com/djamelinfo/myhomelab/raw/main/images/homer.jpg" alt="Homer" width="400"/>

  - [**Pi-hole**](https://pi-hole.net/): Network-wide ad blocker
  - [**PiKVM**](https://github.com/pikvm/pikvm): IP-KVM based on Raspberry Pi

   

## 4. Raspberry Pi 5
- **RAM**: 4GB
- **Storage**: 120GB SSD
- **Running Services**:
  - [**HyperHDR**](https://github.com/awawa-dev/HyperHDR): For advanced HDR lighting setups
  

## 5. Networking Equipment
- **UniFi Cloud Gateway Ultra**: Network management and routing
- **2x UniFi AP LR**: Long-range wireless access points for Wi-Fi coverage
- **Netgear 8-port PoE switch GS308PP**: Power-over-Ethernet for devices like access points
- **TP-Link 8-port managed switch TL-SG608E**: For managing network traffic
- **TP-Link 8-port switch TL-SG108**: Basic network connectivity

## Overview
This home lab setup allows me to experiment with various applications and services while maintaining an efficient and organized environment. It's designed for both learning and practical applications in home automation, media consumption, and network management.
