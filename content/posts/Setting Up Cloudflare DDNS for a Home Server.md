---
title: Setting Up Cloudflare DDNS for a Home Server
date: 2026-06-12
draft: false
tags:
  - homelab
  - cloudflare
  - dynamic dns

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
The IP address is only temporary and will be updated automatically later.

## Step 2: Create a Cloudflare API Token
Navigate to:

**My Profile → API Tokens → Create Token**

Use the **Edit Zone DNS** template.

Grant permissions:

- Zone → DNS → Edit
- Zone → Zone → Read

Limit the token to your specific domain.

Save the token securely, as it will only be displayed once.

  
## Step 3: Deploying Cloudflare DDNS with Docker Compose
Since I already use Docker to manage services in my homelab, I chose to run Cloudflare DDNS as a container.

First create the following directory
```bash
mkdir ~/cloudflare-ddns 
cd ~/cloudflare-ddns

```

Create a `compose.yaml` file with your favorite text editor
```bash
nano compose.yaml
```

Now paste the following in the compose file
```yaml
services:
  cloudflare-ddns:
    image: favonia/cloudflare-ddns:latest
    # Choose the appropriate tag based on your need:
    # - "latest" for the latest stable version (which could become 2.x.y
    #   in the future and break things)
    # - "1" for the latest stable version whose major version is 1
    # - "1.x.y" to pin the specific version 1.x.y
    network_mode: host
    # This bypasses network isolation and makes IPv6 easier (optional; see below)
    restart: always
    # Restart the updater after reboot
    user: "1000:1000"
    # Run the updater with specific user and group IDs (in that order).
    # You can change the two numbers based on your need.
    read_only: true
    # Make the container filesystem read-only (optional but recommended)
    cap_drop: [all]
    # Drop all Linux capabilities (optional but recommended)
    security_opt: [no-new-privileges:true]
    # Another protection to restrict superuser privileges (optional but recommended)
    environment:
      - CLOUDFLARE_API_TOKEN="YOUR_CLOUDFLARE_API_TOKEN"
        # Your Cloudflare API token
      - DOMAINS=home.example.com
        # Your domains (separated by commas)
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