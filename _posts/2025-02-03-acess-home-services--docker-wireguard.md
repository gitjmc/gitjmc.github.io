---
title: Secure home lab access via wireguard 
author:
  name: 
  link: 
date: 2025-02-03 20:35:00 -0400
categories: [Privacy]
tags: [docker, gogs, nextcloud,duckdns,wireguard]
render_with_liquid: false
---

This post will guide and help you how to set up home services accessed remotely via wireguard.

Absolutely! Let's design a secure, privacy-focused setup where you host services (Nextcloud, Gogs, etc.) in Docker containers on your home network and access them remotely **only via WireGuard**. Here's a complete step-by-step guide:

---

## **Architecture Overview**
1. **Home Server**: A machine (e.g., Raspberry Pi, old PC) running Docker and WireGuard.
2. **Services**: Nextcloud, Gogs, etc., running in Docker containers on the home server.
3. **WireGuard VPN**: Securely connects your laptop/mobile devices to the home network.
4. **Access**: Services are **only exposed to the WireGuard VPN IP range** (not the public internet).

---

## **Step 1: Prepare the Home Server**
### **1.1 Install Docker and Docker Compose**
```bash
# Install Docker
curl -fsSL https://get.docker.com | sh
sudo usermod -aG docker $USER

# Install Docker Compose
sudo apt install docker-compose
```

### **1.2 Create a Docker Network**
Create a dedicated Docker network for your services (to simplify routing):
```bash
docker network create --subnet=172.20.0.0/24 internal-net
```

---

## **Step 2: Set Up WireGuard on the Home Server**
### **2.1 Install WireGuard**
```bash
sudo apt install wireguard resolvconf
```

### **2.2 Generate Keys**
```bash
# Generate server keys
wg genkey | sudo tee /etc/wireguard/privatekey | wg pubkey | sudo tee /etc/wireguard/publickey
```

### **2.3 Create WireGuard Config**
Create `/etc/wireguard/wg0.conf`:
```ini
[Interface]
Address = 10.8.0.1/24
PrivateKey = <HOME_SERVER_PRIVATE_KEY>
ListenPort = 51820
PostUp = iptables -A FORWARD -i wg0 -j ACCEPT; iptables -t nat -A POSTROUTING -o eth0 -j MASQUERADE
PostDown = iptables -D FORWARD -i wg0 -j ACCEPT; iptables -t nat -D POSTROUTING -o eth0 -j MASQUERADE

[Peer]
# Laptop
PublicKey = <LAPTOP_PUBLIC_KEY>
AllowedIPs = 10.8.0.2/32

[Peer]
# Phone
PublicKey = <PHONE_PUBLIC_KEY>
AllowedIPs = 10.8.0.3/32
```

Replace:
- `<HOME_SERVER_PRIVATE_KEY>`: Contents of `/etc/wireguard/privatekey`.
- `<LAPTOP_PUBLIC_KEY>`/`<PHONE_PUBLIC_KEY>`: Public keys from your devices (generated later).

### **2.4 Enable IP Forwarding**
Edit `/etc/sysctl.conf`:
```bash
net.ipv4.ip_forward=1
```
Apply changes:
```bash
sudo sysctl -p
```

### **2.5 Start WireGuard**
```bash
sudo wg-quick up wg0
sudo systemctl enable wg-quick@wg0
```

---

## **Step 3: Configure Docker Services**
### **3.1 Create `docker-compose.yml`**
```yaml
version: '3'

services:
  nextcloud:
    image: nextcloud
    networks:
      - internal-net
    restart: unless-stopped
    volumes:
      - nextcloud_data:/var/www/html
    ports:
      - "10.8.0.1:8080:80"  # Expose only to WireGuard IP

  gogs:
    image: gogs/gogs
    networks:
      - internal-net
    restart: unless-stopped
    volumes:
      - gogs_data:/data
    ports:
      - "10.8.0.1:3000:3000"  # Expose only to WireGuard IP

networks:
  internal-net:
    external: true

volumes:
  nextcloud_data:
  gogs_data:
```

### **3.2 Start Services**
```bash
docker-compose up -d
```

---

## **Step 4: Configure Client Devices (Laptop/Phone)**
### **4.1 Install WireGuard**
- **Windows/macOS**: Download from [wireguard.com/install](https://www.wireguard.com/install/).
- **Android/iOS**: Install the WireGuard app.

### **4.2 Generate Client Keys**
On each device, generate a key pair:
```bash
# On Linux/macOS
wg genkey | tee privatekey | wg pubkey > publickey
```

### **4.3 Create Client Configs**
#### **Laptop Example (`laptop.conf`)**:
```ini
[Interface]
Address = 10.8.0.2/24
PrivateKey = <LAPTOP_PRIVATE_KEY>
DNS = 1.1.1.1  # Optional

[Peer]
PublicKey = <HOME_SERVER_PUBLIC_KEY>
Endpoint = <HOME_SERVER_PUBLIC_IP>:51820
AllowedIPs = 10.8.0.0/24
PersistentKeepalive = 25
```

#### **Phone Example (`phone.conf`)**:
```ini
[Interface]
Address = 10.8.0.3/24
PrivateKey = <PHONE_PRIVATE_KEY>
DNS = 1.1.1.1  # Optional

[Peer]
PublicKey = <HOME_SERVER_PUBLIC_KEY>
Endpoint = <HOME_SERVER_PUBLIC_IP>:51820
AllowedIPs = 10.8.0.0/24
PersistentKeepalive = 25
```

### **4.4 Add Clients to the Server**
On the home server, add each client as a peer in `/etc/wireguard/wg0.conf`:
```ini
[Peer]
PublicKey = <LAPTOP_PUBLIC_KEY>
AllowedIPs = 10.8.0.2/32

[Peer]
PublicKey = <PHONE_PUBLIC_KEY>
AllowedIPs = 10.8.0.3/32
```

Reload WireGuard:
```bash
sudo wg syncconf wg0 <(sudo wg-quick strip wg0)
```

---

## **Step 5: Access Services Securely**
Once connected to WireGuard:
- **Nextcloud**: `http://10.8.0.1:8080`
- **Gogs**: `http://10.8.0.1:3000`

---

## **Step 6: Firewall and Security**
### **6.1 Restrict Docker Ports**
Ensure Docker services **only bind to the WireGuard IP** (as done in `docker-compose.yml`):
```yaml
ports:
  - "10.8.0.1:8080:80"  # Not 0.0.0.0:8080:80
```

### **6.2 Configure UFW (Firewall)**
```bash
# Allow WireGuard traffic
sudo ufw allow 51820/udp

# Block all other incoming traffic
sudo ufw default deny incoming
sudo ufw enable
```

---

## **Step 7: Dynamic DNS (Optional)**
If your home IP changes frequently, use a DDNS service (e.g., DuckDNS):
1. Sign up at [duckdns.org](https://www.duckdns.org/).
2. Update the `Endpoint` in client configs to use your DDNS domain (e.g., `myhome.duckdns.org:51820`).

---

## **Step 8: Add More Services**
To add another service (e.g., Jellyfin):
```yaml
# In docker-compose.yml
jellyfin:
  image: jellyfin/jellyfin
  networks:
    - internal-net
  ports:
    - "10.8.0.1:8096:8096"
```

---

## **Troubleshooting**
1. **Check WireGuard Status**:
   ```bash
   sudo wg
   ```
2. **Test Connectivity**:
   ```bash
   ping 10.8.0.1  # From a client device
   ```
3. **Inspect Docker Logs**:
   ```bash
   docker logs nextcloud
   ```

---

## **Final Notes**
- **Security**: Only WireGuard port (`51820/udp`) is exposed to the internet. All services are hidden behind the VPN.
- **Privacy**: No reliance on third-party cloud providers.
- **Scalability**: Add more devices as WireGuard peers or deploy additional Docker services.

This setup ensures your self-hosted services remain private and accessible only to your trusted devices. Let me know if you need help with specific services!
## Learn More

For more knowledge about Jekyll posts, visit the [Jekyll Docs: Posts](https://jekyllrb.com/docs/posts/).
