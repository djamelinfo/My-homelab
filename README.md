# **Home Lab Setup**

This is my personal homelab setup. I hope this setup can inspire you as you build your own homelab!

At the heart of it is an Dell Precision T3620, customized to handle everything from media streaming and file storage to home automation and security. Using Proxmox as the main OS and TrueNAS for managing storage. The entire rack idles at around 150W.

## **1\. Main Server: Dell Precision T3620 running Proxmox**

- **Hardware**:
    - **CPU**: Intel i7-7700
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
    - [**Home Assistant**](https://www.home-assistant.io/): Home automation platform!
    - [**TrueNAS**](https://www.truenas.com/) VM for storage management!
    - **Windows VM** for gaming with GPU passthrough
    - **Pop!\_OS VM** for running an Olama model
    - **Debian VM** for hosting my website
- **Other services (LXC)**:
    - [**Frigate**](https://frigate.video/): Real-time surveillance with object detection, accelerated by Coral USB TPU!
    - [**Jellyfin**](https://jellyfin.org/): Media streaming server!
    - [**PhotoPrism**](https://photoprism.app/): AI-based photo management
    - [**Nginx**](https://www.nginx.com/): Web server and reverse proxy
    - [**NextcloudPi**](https://ownyourbits.com/nextcloudpi/): Self-hosted file sharing and collaboration platform
    - [**Vaultwarden**](https://github.com/dani-garcia/vaultwarden): Self-hosted password manager
    - [**Gitea**](https://gitea.io/en-us/): Self-hosted Git service
    - [**Pi-hole**](https://pi-hole.net/): Network-wide ad blocker!

## **2\. Second Server: Lenovo ThinkCenter 710Q**

- **CPU**: Intel i3-7100T
- **RAM**: 16GB
- **Storage**:
    - 1TB NVMe for OS
    - 1TB 2.5" HDD
    - 4TB 2.5" HDD
    - 5TB 2.5" HDD
- **Virtual Machines (VMs)**:
    - [**Proxmox Backup Server**](https://www.proxmox.com/en/proxmox-backup-server): For managing backups

## **3\. Raspberry Pi 4**

- **RAM**: 4GB
- **Storage**: 128GB SSD
- **Running Services in Docker**:
    - [**Homer Dashboard**](https://github.com/bastienwirtz/homer): Docker containers, including a customizable dashboard![](https://github.com/djamelinfo/My-homelab/blob/main/images/Capture%20d'%C3%A9cran%202025-04-22%20152529.png)
    - [**Pi-hole**](https://pi-hole.net/): Network-wide ad blocker
    - [**PiKVM**](https://github.com/pikvm/pikvm): IP-KVM based on Raspberry Pi
    - [**CloudFlare DDNS**](https://hub.docker.com/r/oznu/cloudflare-ddns/): For advanced HDR lighting setups

## **4\. Raspberry Pi 5**

- **RAM**: 4GB
- **Storage**: 120GB SSD
- **Running Services**:

## **5\. Networking Equipment**

- **UniFi Cloud Gateway Ultra**: Network management and routing
- **UniFi U6 LR**: Long-range wireless access points for Wi-Fi 6 coverage
- **UniFi U6 IW**: Wireless access points for Wi-Fi 6 coverage
- **UniFi Switch Lite 16 PoE**: Power-over-Ethernet for devices like access points
- **Netgear 8-port PoE switch GS308PP**: Power-over-Ethernet for devices like Raspberry Pi 5

## **Overview**

This home lab setup allows me to experiment with various applications and services while maintaining an efficient and organized environment. It's designed for both learning and practical applications in home automation, media consumption, and network management.
