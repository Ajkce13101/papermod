---
title: Setting up wireguard vpn server with docker
date: 2026-06-11
draft: false
tags:
  - networking
  - wireguard
  - vpn
  - docker
  - linux
---
## Introduction
WireGuard has quickly become one of the most popular VPN solutions thanks to its simplicity, speed, and modern cryptography. Running it inside Docker makes deployment and maintenance even easier.

In this guide, we'll set up a WireGuard VPN server using Docker and Docker Compose.

---
## Prerequisites
Before you begin, ensure you have:
- A Linux server or VPS
- Docker installed
- Docker Compose installed
- A public IP address or Dynamic DNS hostname

You can verify Docker is installed:
```bash
docker --version
docker compose version
```

---
## Homelab Environment
Here’s my setup:
- **Proxmox VE**: Virtualization host for all lab VMs and containers
- **Linux VM**: Debian/Ubuntu virtual machine running Docker
- **Docker with Docker compose**: For ease of starting up container and shutting down with docker compose
- **Network**: Static IP assigned to the VM, integrated with home router for DNS resolution


---
## Step 1: Create a docker-compose file
It is a good idea to place it in a separate directory. So lets create a following directory for our wireguard setup

```bash
mkdir ~/wireguard 
cd ~/wireguard
```

## Step 2: Create a Docker Compose File for Wireguard
Now on the wireguard directory create a compose file with the text editor of your choice. Here I have used nano
```bash
nano compose.yaml
```

Here is what the compose file should look like: 
```yaml
services:
  wireguard:
    image: lscr.io/linuxserver/wireguard:latest
    container_name: wireguard
    cap_add:
      - NET_ADMIN
      - SYS_MODULE #optional
    environment:
      - PUID=1000
      - PGID=1000
      - TZ=Etc/UTC
      - SERVERURL=auto #optional
      - SERVERPORT=51820 #optional
      - PEERS=peer1,peer2 #optional
      - PEERDNS=1.1.1.2,1.0.0.2 #optional
      - INTERNAL_SUBNET=10.13.13.0 #optional
      - ALLOWEDIPS=0.0.0.0/0 #optional
    volumes:
      - /home/ajaya/wireguard:/config
      - /lib/modules:/lib/modules #optional
    ports:
      - 51820:51820/udp
    sysctls:
      - net.ipv4.conf.all.src_valid_mark=1
    restart: unless-stopped
```

  

## Step 3: Start the server
Run:
```bash
docker-compose up -d

```

Verify if it is running:
```bash
docker ps
```
You should see the wireguard container is listed



## Step 4: Retrieve Client Configurations
The LinuxServer image automatically generates client configurations.

List generated peer files:
```bash
ls config/peer1/
```

Display the client configuration:
```bash
cat config/peer/peer1/peer1.config
```

You should see something like this:
```bash
[Interface]
PrivateKey = CLIENT_PRIVATE_KEY 
Address = 10.13.13.2/32 
DNS = 1.1.1.1 

[Peer] 
PublicKey = SERVER_PUBLIC_KEY 
Endpoint = your-domain.example.com:51820 
AllowedIPs = 0.0.0.0/0 
PersistentKeepalive = 25
```

## Generate a QR Code
For mobile devices, importing via QR code is much easier.

Generate a QR code:
```bash
docker exec -it wireguard /app/show-peer 1
```
Open the WireGuard app and scan the displayed QR code.


## Configure Port Forwarding
On your router, forward:
```bash
UDP 51820
```
to the internal IP address of your Docker host.

Without port forwarding, clients outside your network will not be able to connect


## Testing the VPN
Connect using your WireGuard client and verify connectivity:

Check your public IP:
```bash
curl ifconfig.me
```
The returned IP should match your home network or VPS.


## Final Thoughts
Using Docker dramatically simplifies the deployment of WireGuard. With only a single Compose file, you can quickly build a secure VPN solution for remote access, homelab management, or protecting your traffic while travelling.

WireGuard's lightweight design and excellent performance make it one of the best self-hosted VPN solutions available today. Combined with Docker, maintaining and updating your VPN server becomes almost effortless.

Happy self-hosting!