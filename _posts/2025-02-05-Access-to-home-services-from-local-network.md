---
title: Access to home services from local network (DS)
author:
  name: 
  link: 
date: 2025-02-05 20:35:00 -0400
categories: [Architecture]
tags: [privacy, docker, wireguard, local-network, local-services]
render_with_liquid: false
---

Yes, it is absolutely possible to access your **home services** (e.g., Nextcloud, Gogs) from your **local network** even when they are configured to be accessible only via WireGuard. However, there are a few considerations and configurations you need to address to make this work seamlessly.

---

## **Why This Happens**
When you configure services to bind to the WireGuard IP (e.g., `10.8.0.2`), they are only accessible through the WireGuard interface. If you try to access them from your local network (e.g., `192.168.1.x`), the traffic won't route correctly because:
1. The services are bound to the WireGuard IP (`10.8.0.2`), not the local network IP (`192.168.1.x`).
2. Your local devices don't know how to route traffic to the WireGuard subnet (`10.8.0.0/24`).

---

## **Solutions to Access Home Services Locally**

### **1. Bind Services to Both WireGuard and Local Network IPs**
Modify your Docker Compose configuration to bind services to both the WireGuard IP and the local network IP.

#### Example for Nextcloud:
```yaml
nextcloud:
  image: nextcloud:latest
  container_name: nextcloud
  ports:
    - "10.8.0.2:8080:80"  # Bind to WireGuard IP
    - "192.168.1.100:8080:80"  # Bind to local network IP
  volumes:
    - nextcloud_data:/var/www/html
  restart: unless-stopped
```

- Replace `192.168.1.100` with the local IP of your home server.
- Now, you can access Nextcloud:
  - From the local network: `http://192.168.1.100:8080`
  - From the WireGuard VPN: `http://10.8.0.2:8080`

---

### **2. Use Split DNS**
Set up a **local DNS server** (e.g., Pi-hole) to resolve your service domains (e.g., `nextcloud.home`) to the local IP (`192.168.1.100`) when accessed from the local network, and to the WireGuard IP (`10.8.0.2`) when accessed remotely.

#### Steps:
1. **Install Pi-hole**:
   Add Pi-hole to your Docker Compose:
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
       - "53:53/tcp"
       - "53:53/udp"
       - "80:80/tcp"
     restart: unless-stopped
   ```

2. **Add Local DNS Entries**:
   - In the Pi-hole web interface, go to **Local DNS > DNS Records**.
   - Add entries for your services:
     - `nextcloud.home` → `192.168.1.100`
     - `gogs.home` → `192.168.1.100`

3. **Configure Remote DNS**:
   - On the WireGuard client configuration, set the DNS server to the Pi-hole IP (`10.8.0.2`).
   - Add DNS entries in Pi-hole for the WireGuard subnet:
     - `nextcloud.home` → `10.8.0.2`
     - `gogs.home` → `10.8.0.2`

4. **Access Services**:
   - From the local network: `http://nextcloud.home:8080`
   - From the WireGuard VPN: `http://nextcloud.home:8080`

---

### **3. Use NAT (Network Address Translation)**
Configure your home server to route traffic between the local network (`192.168.1.0/24`) and the WireGuard subnet (`10.8.0.0/24`).

#### Steps:
1. **Enable IP Forwarding**:
   On the home server, edit `/etc/sysctl.conf`:
   ```bash
   net.ipv4.ip_forward=1
   ```
   Apply changes:
   ```bash
   sudo sysctl -p
   ```

2. **Add NAT Rules**:
   On the home server, add iptables rules to route traffic:
   ```bash
   sudo iptables -t nat -A POSTROUTING -s 192.168.1.0/24 -d 10.8.0.0/24 -j MASQUERADE
   sudo iptables -A FORWARD -s 192.168.1.0/24 -d 10.8.0.0/24 -j ACCEPT
   sudo iptables -A FORWARD -s 10.8.0.0/24 -d 192.168.1.0/24 -j ACCEPT
   ```

3. **Persist iptables Rules**:
   Save the rules so they persist after reboot:
   ```bash
   sudo apt install iptables-persistent
   sudo netfilter-persistent save
   ```

4. **Access Services**:
   - From the local network, use the WireGuard IP (`10.8.0.2`) to access services.
   - Example: `http://10.8.0.2:8080`

---

### **4. Use a Reverse Proxy**
Set up a reverse proxy (e.g., Nginx, Traefik) to handle routing for both local and WireGuard traffic.

#### Example with Nginx:
1. **Install Nginx**:
   Add Nginx to your Docker Compose:
   ```yaml
   nginx:
     image: nginx:latest
     container_name: nginx
     ports:
       - "80:80"
       - "443:443"
     volumes:
       - ./nginx.conf:/etc/nginx/nginx.conf
       - ./certs:/etc/ssl/certs
     restart: unless-stopped
   ```

2. **Configure Nginx**:
   Edit `nginx.conf` to route traffic based on the source IP:
   ```nginx
   server {
       listen 80;
       server_name nextcloud.home;

       location / {
           proxy_pass http://10.8.0.2:8080;  # WireGuard IP
       }
   }

   server {
       listen 80;
       server_name nextcloud.home;

       # Local network IP
       set $local_ip 192.168.1.100;

       if ($remote_addr ~ ^192\.168\.1\.) {
           proxy_pass http://$local_ip:8080;
       }

       location / {
           proxy_pass http://10.8.0.2:8080;  # WireGuard IP
       }
   }
   ```

3. **Access Services**:
   - From the local network: `http://nextcloud.home`
   - From the WireGuard VPN: `http://nextcloud.home`

---

## **Which Solution Should You Use?**
- **Bind to Both IPs**: Simplest solution if you don’t mind managing multiple bindings.
- **Split DNS**: Best for seamless access with domain names.
- **NAT**: Useful if you want to route all traffic through WireGuard.
- **Reverse Proxy**: Ideal for advanced setups with multiple services and domains.

---


