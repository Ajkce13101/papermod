---
title: AdGuard setup for blocking ads
date: 2026-04-01
draft: false
tags:
  - homelab
  - networking
  - adguard
  - dns
  - proxmox
  - linux
---
## Introduction
Most residential internet connections use a dynamic public IP address. This means that your ISP can change your public IP at any time, making it difficult to access self-hosted services consistently.

I already used Cloudflare to manage the DNS for my personal website, so when I needed a Dynamic DNS solution for my homelab, it made sense to use the tools I already had. Rather than paying my ISP for a static public IP or signing up for another DDNS provider, Cloudflare's free API allowed me to keep my home server reachable through a permanent domain name. Whenever my home IP changes, a small script automatically updates the DNS record in Cloudflare.

In this guide, I'll show you how to set up **Cloudflare DDNS** for your homelab using a Linux server.

---
## Why Use Cloudflare DDNS?
I was already using Cloudflare as the authoritative DNS provider for my personal website by pointing my domain's nameservers to Cloudflare. Since my DNS records were already managed in one place, setting up Dynamic DNS through Cloudflare was the most logical choice.

Using Cloudflare for Dynamic DNS offers several benefits:
- Free to use
- Fast and reliable DNS
- Easy API integration
- Works well with homelab environments
- Supports proxying and additional security features

---
## Prerequisites
Before starting, ensure you have:
- A domain name added to Cloudflare
- Access to the Cloudflare Dashboard
- A Linux server that runs continuously
- Basic familiarity with the terminal


---
## Step 1: Create a DNS Record
In Cloudflare:

1. Navigate to **DNS**.
2. Create an **A Record**.
!![Image Description](/images/aRecord.png)

## Step 2: Create a Docker Compose File for AdGuard Home
On the Linux VM, create a directory for AdGuard Home:
```bash
mkdir -p ~/adguard && cd ~/adguard
```

Create docker-compose.yml:
```yaml
version: "3"

services:
  adguardhome:
    container_name: adguardhome
    image: adguard/adguardhome:latest
    restart: unless-stopped
    ports:
      - "53:53/tcp"
      - "53:53/udp"
      - "80:80/tcp"
      - "443:443/tcp"
      - "3000:3000/tcp" # optional admin port

    volumes:
      - ./workdir:/opt/adguardhome/work
      - ./config:/opt/adguardhome/conf

    networks:
      - adguard-net
        
networks:
  adguard-net:
    driver: bridge
```

  

## Step 3: Start AdGuard Home
Run:
```bash
docker-compose up -d

```

Check container logs:
```bash
docker logs -f adguardhome
```

Once started, the web UI is available at:
```bash
http://<vm-ip>:3000
```

Follow the setup wizard to configure:
- Admin credentials
- Upstream DNS servers (e.g., Cloudflare, Google, or Quad9)
- DHCP/DNS integration (optional)
- Blocklists (AdGuard defaults or custom lists)


## Step 4: Configure Your Network to Use AdGuard
To enable network-wide ad-blocking:
1. Set the DNS on your home router to point to your Linux VM IP.
2. Some routers might not allow you to setup custom DNS server, alternatively, configure individual devices to use the VM’s DNS.
This way, all devices on your network automatically benefit from AdGuard’s filtering.

![Image Description](/images/adguard.png)
## Conclusion
Running AdGuard Home in a Docker container inside a Linux VM on Proxmox gives you a robust, network-wide DNS-level ad-blocker. It’s lightweight, self-contained, and fully customizable, making it perfect for a homelab environment.

This setup allows me to:
- Block ads and trackers across all devices
- Manage DNS centrally
- Easily maintain and update via Docker
- Integrate with other homelab services like WireGuard and MeshCentral