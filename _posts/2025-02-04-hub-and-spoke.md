---
title: Hub and Spoke or Site to Site VPN
author: 
  name: 
  link:
date: 2025-02-04 01:03:00 -0700
categories: [Architecture]
tags: [docker, cloud, ubuntu, wireguard, gogs, nextcloud]
math: true
mermaid: true
image:
  src: /commons/devices-mockup.png
  width: 400
  height: 250
---

Cet article montre comment mettre en place l'architecture connue comme Hub And Spoke [^ref1].

Yes, absolutely! You can set up **WireGuard on a cloud provider (like DigitalOcean)** and use it to securely connect to your **home local network**, where your services (e.g., Nextcloud, Gogs, etc.) are running. This setup is often called a **"hub-and-spoke"** or **"site-to-site" VPN**, where the cloud server acts as the central hub, and your home network is the spoke.

This approach has several advantages:
1. **Static IP**: The cloud server provides a static IP address, making it easier to connect to your home network.
2. **Accessibility**: You can access your home services from anywhere via the cloud server.
3. **Security**: All traffic between the cloud server and your home network is encrypted via WireGuard.

---

## **Step-by-Step Guide: WireGuard in the Cloud + Home Local Network**

---

### **Step 1: Set Up a Cloud Server (DigitalOcean)**
1. **Create a Droplet**:
   - Go to [DigitalOcean](https://cloud.digitalocean.com/) and create a new Droplet.
   - Choose Ubuntu or Debian as the OS.
   - Enable **IPv6** (optional but recommended for future-proofing).

2. **SSH into the Droplet**:
   ```bash
   ssh root@<DROPLET_IP>
   ```

3. **Install WireGuard**:
   ```bash
   sudo apt update
   sudo apt install wireguard resolvconf
   ```

4. **Generate WireGuard Keys**:
   ```bash
   wg genkey | sudo tee /etc/wireguard/privatekey | wg pubkey | sudo tee /etc/wireguard/publickey
   ```

---

### **Step 2: Configure WireGuard on the Cloud Server**
1. **Create the WireGuard Config**:
   Edit `/etc/wireguard/wg0.conf`:
   ```ini
   [Interface]
   Address = 10.8.0.1/24
   PrivateKey = <CLOUD_SERVER_PRIVATE_KEY>
   ListenPort = 51820
   PostUp = iptables -A FORWARD -i wg0 -j ACCEPT; iptables -t nat -A POSTROUTING -o eth0 -j MASQUERADE
   PostDown = iptables -D FORWARD -i wg0 -j ACCEPT; iptables -t nat -D POSTROUTING -o eth0 -j MASQUERADE

   [Peer]
   # Home Server
   PublicKey = <HOME_SERVER_PUBLIC_KEY>
   AllowedIPs = 10.8.0.2/32, 192.168.1.0/24  # Include your home LAN subnet
   ```

   Replace:
   - `<CLOUD_SERVER_PRIVATE_KEY>`: The private key generated on the cloud server.
   - `<HOME_SERVER_PUBLIC_KEY>`: The public key from your home server (generated in Step 3).

2. **Enable IP Forwarding**:
   Edit `/etc/sysctl.conf`:
   ```bash
   net.ipv4.ip_forward=1
   ```
   Apply changes:
   ```bash
   sudo sysctl -p
   ```

3. **Start WireGuard**:
   ```bash
   sudo wg-quick up wg0
   sudo systemctl enable wg-quick@wg0
   ```

---

### **Step 3: Set Up WireGuard on Your Home Server**
1. **Install WireGuard**:
   ```bash
   sudo apt update
   sudo apt install wireguard resolvconf
   ```

2. **Generate WireGuard Keys**:
   ```bash
   wg genkey | sudo tee /etc/wireguard/privatekey | wg pubkey | sudo tee /etc/wireguard/publickey
   ```

3. **Create the WireGuard Config**:
   Edit `/etc/wireguard/wg0.conf`:
   ```ini
   [Interface]
   Address = 10.8.0.2/24
   PrivateKey = <HOME_SERVER_PRIVATE_KEY>
   DNS = 1.1.1.1  # Optional

   [Peer]
   PublicKey = <CLOUD_SERVER_PUBLIC_KEY>
   Endpoint = <DROPLET_IP>:51820
   AllowedIPs = 10.8.0.0/24, 0.0.0.0/0  # Route all traffic through the VPN (optional)
   PersistentKeepalive = 25
   ```

   Replace:
   - `<HOME_SERVER_PRIVATE_KEY>`: The private key generated on the home server.
   - `<CLOUD_SERVER_PUBLIC_KEY>`: The public key from the cloud server.

4. **Start WireGuard**:
   ```bash
   sudo wg-quick up wg0
   sudo systemctl enable wg-quick@wg0
   ```

---

### **Step 4: Configure Routing on the Home Server**
1. **Add a Route to the Home LAN**:
   On the cloud server, ensure traffic to your home LAN subnet (`192.168.1.0/24`) is routed through the WireGuard interface:
   ```bash
   sudo ip route add 192.168.1.0/24 dev wg0
   ```

2. **Test Connectivity**:
   From the cloud server, ping your home server:
   ```bash
   ping 10.8.0.2
   ```
   From the home server, ping the cloud server:
   ```bash
   ping 10.8.0.1
   ```

---

### **Step 5: Access Home Services via the Cloud Server**
1. **Expose Services**:
   - Ensure your home services (e.g., Nextcloud, Gogs) are bound to the WireGuard IP (`10.8.0.2`) or your home LAN IP (`192.168.1.x`).
   - Example for Nextcloud:
     ```bash
     docker run -d \
       --name nextcloud \
       -p 10.8.0.2:8080:80 \
       nextcloud
     ```

2. **Access Services**:
   - From the cloud server, access your home services using the WireGuard IP (e.g., `http://10.8.0.2:8080`).
   - From your local machine, connect to the cloud server via WireGuard and access the services.

---

### **Step 6: Secure the Setup**
1. **Firewall Rules**:
   - On the cloud server, allow only WireGuard traffic:
     ```bash
     sudo ufw allow 51820/udp
     sudo ufw enable
     ```
   - On the home server, restrict access to services via the WireGuard interface.

2. **Disable Public Access**:
   - Ensure your home services are not exposed to the public internet.

---

### **Step 7: Optional - Add More Devices**
You can add more devices (e.g., laptop, phone) as peers to the WireGuard network:
1. Generate keys for each device.
2. Add them as peers in the cloud server's `wg0.conf`:
   ```ini
   [Peer]
   PublicKey = <DEVICE_PUBLIC_KEY>
   AllowedIPs = 10.8.0.3/32
   ```
3. Configure the devices to connect to the cloud server.

---

## **Benefits of This Setup**
- **Centralized Access**: Use the cloud server as a gateway to access your home services.
- **Privacy**: All traffic is encrypted via WireGuard.
- **Scalability**: Easily add more devices or services.

---

### Reverse Footnote

[^ref1]: [Voir ce liensur youtube](https://www.youtube.com/watch?v=WoHffLF0jfU)
[^ref2]: [source in github docker machine](https://github.com/docker/machine/releases/tag/v0.16.2)
[^ref3]: [source in github docker compose](https://docs.docker.com/compose/install/)
[^ref4]: [source article](https://medium.com/@leonardo.lp2/installing-docker-on-deepin-15-9-2312f28f09b7)

 
