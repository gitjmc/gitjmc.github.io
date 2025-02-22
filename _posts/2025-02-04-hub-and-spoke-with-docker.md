---
title: Hub and Spoke or Site to Site VPN on docker
author: 
  name: 
  link:
date: 2025-02-04 03:06:00 -0700
categories: [Archtecture, middle-ware]
tags: [docker-compose, ubuntu, wireguard, gogs, nextcoud, Cloud]
math: true
mermaid: true
image:
  src: /commons/devices-mockup.png
  width: 400
  height: 250
---

Absolutely! Below are **Docker Compose** examples for both the **cloud server** (DigitalOcean) and the **home server** (local network). These configurations will help you set up WireGuard on the cloud server and run your services (e.g., Nextcloud, Gogs) on the home server.

---

## **1. Docker Compose for the Cloud Server (DigitalOcean)**

This setup will run **WireGuard** on the cloud server to act as the VPN hub.

### **`docker-compose.yml` for Cloud Server**
```yaml
version: '3.8'

services:
  wireguard:
    image: linuxserver/wireguard:latest
    container_name: wireguard
    environment:
      - PUID=1000
      - PGID=1000
      - TZ=UTC
      - SERVERURL=<DROPLET_IP>  # Replace with your DigitalOcean Droplet IP
      - SERVERPORT=51820
      - PEERS=home  # Name of the peer (home server)
      - PEERDNS=1.1.1.1
    volumes:
      - wireguard_config:/config
      - /lib/modules:/lib/modules
    ports:
      - "51820:51820/udp"
    cap_add:
      - NET_ADMIN
      - SYS_MODULE
    sysctls:
      - net.ipv4.ip_forward=1
      - net.ipv6.conf.all.forwarding=1
    restart: unless-stopped

volumes:
  wireguard_config:
```

### **Steps to Run on Cloud Server**
1. **Install Docker and Docker Compose**:
   ```bash
   curl -fsSL https://get.docker.com | sh
   sudo usermod -aG docker $USER
   sudo apt install docker-compose
   ```

2. **Create the `docker-compose.yml` file**:
   Save the above configuration in a file named `docker-compose.yml`.

3. **Start WireGuard**:
   ```bash
   docker-compose up -d
   ```

4. **Get the Home Server Configuration**:
   - After starting the container, the WireGuard configuration for the home server will be generated in the `wireguard_config` volume.
   - Locate the `home.conf` file (for the home server peer) in the volume:
     ```bash
     docker-compose exec wireguard cat /config/peer_home/peer_home.conf
     ```
   - Copy this configuration to your home server.

---

## **2. Docker Compose for the Home Server (Local Network)**

This setup will run your services (e.g., Nextcloud, Gogs) and connect to the cloud server via WireGuard.

### **`docker-compose.yml` for Home Server**
```yaml
version: '3.8'

services:
  wireguard:
    image: linuxserver/wireguard:latest
    container_name: wireguard
    environment:
      - PUID=1000
      - PGID=1000
      - TZ=UTC
      - PEERDNS=1.1.1.1
    volumes:
      - wireguard_config:/config
      - /lib/modules:/lib/modules
    cap_add:
      - NET_ADMIN
      - SYS_MODULE
    sysctls:
      - net.ipv4.ip_forward=1
      - net.ipv6.conf.all.forwarding=1
    restart: unless-stopped

  nextcloud:
    image: nextcloud:latest
    container_name: nextcloud
    depends_on:
      - wireguard
    volumes:
      - nextcloud_data:/var/www/html
    ports:
      - "10.8.0.2:8080:80"  # Bind to WireGuard IP
    restart: unless-stopped

  gogs:
    image: gogs/gogs:latest
    container_name: gogs
    depends_on:
      - wireguard
    volumes:
      - gogs_data:/data
    ports:
      - "10.8.0.2:3000:3000"  # Bind to WireGuard IP
    restart: unless-stopped

volumes:
  wireguard_config:
  nextcloud_data:
  gogs_data:
```

### **Steps to Run on Home Server**
1. **Install Docker and Docker Compose**:
   ```bash
   curl -fsSL https://get.docker.com | sh
   sudo usermod -aG docker $USER
   sudo apt install docker-compose
   ```

2. **Create the `docker-compose.yml` file**:
   Save the above configuration in a file named `docker-compose.yml`.

3. **Add the WireGuard Configuration**:
   - Copy the `home.conf` file (from the cloud server) to the home server.
   - Place it in the `wireguard_config` volume (e.g., `./wireguard_config/peer_home/peer_home.conf`).

4. **Start the Services**:
   ```bash
   docker-compose up -d
   ```

---

## **3. Accessing Services**

### **From the Cloud Server**
- Use the WireGuard IP (`10.8.0.2`) to access services on the home server:
  - **Nextcloud**: `http://10.8.0.2:8080`
  - **Gogs**: `http://10.8.0.2:3000`

### **From Your Local Machine**
1. **Connect to the Cloud Server via WireGuard**:
   - Install WireGuard on your local machine.
   - Use the configuration file generated for your local machine (similar to the home server setup).

2. **Access Services**:
   - Once connected to the VPN, use the WireGuard IP (`10.8.0.2`) to access services.

---

## **4. Additional Services**

You can add more services to the home server's `docker-compose.yml` file. For example:

### **Add Pi-hole for Ad Blocking**
```yaml
pihole:
  image: pihole/pihole:latest
  container_name: pihole
  environment:
    - TZ=UTC
    - WEBPASSWORD=your_password_here
  volumes:
    - pihole_data:/etc/pihole
    - pihole_dnsmasq:/etc/dnsmasq.d
  ports:
    - "10.8.0.2:53:53/tcp"
    - "10.8.0.2:53:53/udp"
    - "10.8.0.2:8085:80"  # Web UI
  restart: unless-stopped

volumes:
  pihole_data:
  pihole_dnsmasq:
```

---

## **5. Security Considerations**
- **Firewall**: Ensure only necessary ports are open on the cloud server (e.g., `51820/udp` for WireGuard).
- **Backups**: Regularly back up your Docker volumes and configurations.
- **Updates**: Keep your Docker images and host system up to date.

---

### Reverse Footnote

[^ref1]: [Voir ce liensur youtube](https://www.youtube.com/watch?v=WoHffLF0jfU)
[^ref2]: [source in github docker machine](https://github.com/docker/machine/releases/tag/v0.16.2)
[^ref3]: [source in github docker compose](https://docs.docker.com/compose/install/)
[^ref4]: [source article](https://medium.com/@leonardo.lp2/installing-docker-on-deepin-15-9-2312f28f09b7)

 