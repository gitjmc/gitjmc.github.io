---
title: Setup For WireGuard, NextCloud, and DuckDNS using Docker (G)
author:
  name: 
  link: 
date: 2025-02-15 09:35:00 -0400
categories: [Privacy]
tags: [docker, nextcloud, duckDNS, wireguard]
render_with_liquid: false
mermaid: true
---

# Basic configuration

It seems like you meant "DuckDNS" instead of "dukdns"—I’ll assume that and proceed with a setup for WireGuard, NextCloud, and DuckDNS using Docker. This combination gives you a secure VPN (WireGuard), self-hosted cloud storage (NextCloud), and dynamic DNS (DuckDNS) to keep your domain updated with your IP. Here’s a streamlined, practical approach using Docker Compose for simplicity and maintainability, assuming you’re running this on a single machine (like a home server or VPS).

### Docker Compose Setup

First, make sure you have Docker and Docker Compose installed. Then, create a directory for this project (e.g., `~/cloud-vpn`), and inside it, create a `docker-compose.yml` file with the following content:

```yaml
version: '3'

services:
  duckdns:
    image: linuxserver/duckdns
    container_name: duckdns
    environment:
      - TOKEN=your-duckdns-token # Replace with your DuckDNS token
      - SUBDOMAINS=your-subdomain # Replace with your DuckDNS subdomain (e.g., mycloud)
    volumes:
      - ./duckdns:/config # Stores logs, not critical but useful
    restart: unless-stopped

  wireguard:
    image: linuxserver/wireguard
    container_name: wireguard
    cap_add:
      - NET_ADMIN
      - SYS_MODULE
    environment:
      - PUID=1000 # Your user ID, adjust if needed
      - PGID=1000 # Your group ID, adjust if needed
      - TZ=America/New_York # Adjust to your timezone
      - SERVERURL=your-subdomain.duckdns.org # Your DuckDNS domain
      - SERVERPORT=51820
      - PEERS=2 # Number of clients (adjust as needed)
    volumes:
      - ./wireguard:/config # Stores WireGuard config and keys
      - /lib/modules:/lib/modules:ro # For kernel module loading
    ports:
      - 51820:51820/udp # WireGuard port
    sysctls:
      - net.ipv4.conf.all.src_valid_mark=1 # Needed for WireGuard
    restart: unless-stopped

  db:
    image: mariadb:10.5
    container_name: nextcloud-db
    environment:
      - MYSQL_ROOT_PASSWORD=your-root-password # Replace with a secure password
      - MYSQL_DATABASE=nextcloud
      - MYSQL_USER=nextcloud
      - MYSQL_PASSWORD=your-db-password # Replace with a secure password
    volumes:
      - ./db:/var/lib/mysql # Persists database data
    restart: unless-stopped

  nextcloud:
    image: nextcloud:latest
    container_name: nextcloud
    depends_on:
      - db
    environment:
      - MYSQL_HOST=db
      - MYSQL_DATABASE=nextcloud
      - MYSQL_USER=nextcloud
      - MYSQL_PASSWORD=your-db-password # Match the db service password
      - NEXTCLOUD_ADMIN_USER=admin # Initial admin username
      - NEXTCLOUD_ADMIN_PASSWORD=your-admin-password # Replace with a secure password
    volumes:
      - ./nextcloud:/var/www/html # Persists NextCloud data
    ports:
      - 8080:80 # Maps NextCloud to port 8080 on host
    restart: unless-stopped
```

### Steps to Install and Configure

1. **Prep Work**
   - Sign up at [DuckDNS](https://www.duckdns.org), get your token, and pick a subdomain (e.g., `mycloud` → `mycloud.duckdns.org`).
   - Replace placeholders in the `docker-compose.yml` file:
     - `your-duckdns-token` and `your-subdomain` under `duckdns`.
     - `your-subdomain.duckdns.org` under `wireguard`.
     - Passwords under `db` and `nextcloud` (use strong, unique ones).
   - Create the directory structure: `mkdir ~/cloud-vpn && cd ~/cloud-vpn`.

2. **Start the Services**
   - Run `docker-compose up -d` in the directory with your `docker-compose.yml`. This pulls the images and starts the containers.
   - Check logs to ensure everything’s running: `docker-compose logs`.

3. **Configure WireGuard**
   - After starting, WireGuard generates client configs in `./wireguard/qr_codes/` or `./wireguard/config/peerX/`. Copy these to your devices (e.g., via QR code for mobile or `.conf` file for desktop).
   - Install the WireGuard client on your device (see [wireguard.com/install](https://www.wireguard.com/install/)), import the config, and connect.

4. **Set Up NextCloud**
   - Open your browser to `http://localhost:8080` (or your server’s IP:8080 if remote). The first time, NextCloud finishes setup using the admin credentials from the compose file.
   - Once set up, configure NextCloud to trust your DuckDNS domain under Settings > Administration > General (e.g., `mycloud.duckdns.org`).

5. **Secure Access**
   - By default, NextCloud is on port 8080 and accessible locally. To restrict it to WireGuard:
     - Edit the `ports` line under `nextcloud` to remove `- 8080:80` (making it internal-only).
     - Access it via the VPN at `http://nextcloud:80` (Docker’s internal networking) or set up a reverse proxy if needed.
   - Alternatively, use your firewall (e.g., `ufw` or `iptables`) to allow only WireGuard subnet traffic (e.g., `10.13.13.0/24`, check your WireGuard config).

6. **Test It**
   - Connect to WireGuard on a client device.
   - Visit `http://your-subdomain.duckdns.org:8080` (or internal URL if ports are removed) to access NextCloud securely.
   - DuckDNS keeps your domain updated, so it works even if your public IP changes.

### Why This Is the Best Way
- **Docker Compose**: Manages all services in one file, making it easy to start, stop, or tweak settings.
- **Security**: WireGuard encrypts traffic, and restricting NextCloud to VPN access adds a layer of protection.
- **Simplicity**: DuckDNS handles dynamic IPs without complex DNS setups, and the chosen images (LinuxServer and official NextCloud) are reliable and well-documented.

### Tips and Tweaks
- **HTTPS**: For production, add a reverse proxy like Nginx or Traefik with Let’s Encrypt for SSL on NextCloud (port 443).
- **Storage**: Mount external drives to `./nextcloud` for more space.
- **Backup**: Regularly back up `./db` and `./nextcloud` directories.
- **Troubleshooting**: Use `docker-compose logs <service>` to debug if something fails.

# Let's enhance security

## Adding HTTPS for secure browsing and adjusting the firewall
Let’s enhance your WireGuard, NextCloud, and DuckDNS setup by adding HTTPS for secure browsing and adjusting the firewall to restrict NextCloud access to WireGuard VPN clients only. I’ll assume you’re using the Docker Compose setup from my previous response and build on it. We’ll use Nginx Proxy Manager for HTTPS (simpler than raw Nginx or Traefik for beginners) and configure `iptables` for the firewall.

---

### Step 1: Setting Up HTTPS with Nginx Proxy Manager

#### Why Nginx Proxy Manager?
It’s a Docker-friendly tool with a web UI to manage reverse proxies and SSL certificates via Let’s Encrypt, making HTTPS setup straightforward.

#### Updated Docker Compose
Add Nginx Proxy Manager to your `docker-compose.yml`. Here’s the full updated file (replace your existing one, keeping your custom values like tokens and passwords):

```yaml
version: '3'

services:
  duckdns:
    image: linuxserver/duckdns
    container_name: duckdns
    environment:
      - TOKEN=your-duckdns-token
      - SUBDOMAINS=your-subdomain
    volumes:
      - ./duckdns:/config
    restart: unless-stopped

  wireguard:
    image: linuxserver/wireguard
    container_name: wireguard
    cap_add:
      - NET_ADMIN
      - SYS_MODULE
    environment:
      - PUID=1000
      - PGID=1000
      - TZ=America/New_York
      - SERVERURL=your-subdomain.duckdns.org
      - SERVERPORT=51820
      - PEERS=2
    volumes:
      - ./wireguard:/config
      - /lib/modules:/lib/modules:ro
    ports:
      - 51820:51820/udp
    sysctls:
      - net.ipv4.conf.all.src_valid_mark=1
    restart: unless-stopped

  db:
    image: mariadb:10.5
    container_name: nextcloud-db
    environment:
      - MYSQL_ROOT_PASSWORD=your-root-password
      - MYSQL_DATABASE=nextcloud
      - MYSQL_USER=nextcloud
      - MYSQL_PASSWORD=your-db-password
    volumes:
      - ./db:/var/lib/mysql
    restart: unless-stopped

  nextcloud:
    image: nextcloud:latest
    container_name: nextcloud
    depends_on:
      - db
    environment:
      - MYSQL_HOST=db
      - MYSQL_DATABASE=nextcloud
      - MYSQL_USER=nextcloud
      - MYSQL_PASSWORD=your-db-password
      - NEXTCLOUD_ADMIN_USER=admin
      - NEXTCLOUD_ADMIN_PASSWORD=your-admin-password
    volumes:
      - ./nextcloud:/var/www/html
    restart: unless-stopped
    # Note: No ports exposed here; Nginx Proxy Manager will handle access

  nginx-proxy-manager:
    image: jc21/nginx-proxy-manager:latest
    container_name: nginx-proxy-manager
    ports:
      - 80:80    # HTTP
      - 443:443  # HTTPS
      - 81:81    # Web UI
    environment:
      - DB_MYSQL_HOST=db
      - DB_MYSQL_PORT=3306
      - DB_MYSQL_USER=nginxpm
      - DB_MYSQL_PASSWORD=nginxpm-password # Replace with a secure password
      - DB_MYSQL_DATABASE=nginxpm
    volumes:
      - ./nginx-proxy-manager/data:/data
      - ./nginx-proxy-manager/letsencrypt:/etc/letsencrypt
    depends_on:
      - db
    restart: unless-stopped
```

#### Steps to Set Up HTTPS
1. **Update and Restart**
   - Save the updated `docker-compose.yml` in your `~/cloud-vpn` directory.
   - Run `docker-compose up -d` to recreate containers with the new service.

2. **Initial Nginx Proxy Manager Setup**
   - Open `http://your-server-ip:81` in your browser (replace `your-server-ip` with your machine’s IP, e.g., `192.168.1.100` or `localhost` if local).
   - Log in with default credentials: `admin@example.com` / `changeme`.
   - Change the email and password immediately in the UI.

3. **Add a Proxy Host**
   - In the Nginx Proxy Manager UI, go to “Hosts” > “Proxy Hosts” > “Add Proxy Host”.
   - **Details**:
     - Domain Name: `your-subdomain.duckdns.org`
     - Scheme: `http`
     - Forward Hostname/IP: `nextcloud` (Docker service name)
     - Forward Port: `80`
   - Enable “Block Common Exploits” and “Websockets Support” for safety and compatibility.
   - Save this configuration.

4. **Set Up SSL**
   - Edit the proxy host you just created.
   - Go to the “SSL” tab, select “Request a new certificate”.
   - Check “Force SSL” and “HTTP/2 Support”.
   - Agree to Let’s Encrypt terms and save. It’ll fetch a certificate for `your-subdomain.duckdns.org`.
   - Wait a minute; if it fails, ensure port 80 is open to the internet (temporary for cert issuance).

5. **Test HTTPS**
   - Visit `https://your-subdomain.duckdns.org`. You should see NextCloud over HTTPS.
   - Update NextCloud’s trusted domains in `./nextcloud/config/config.php`:
     ```php
     'trusted_domains' => 
       array (
         0 => 'localhost',
         1 => 'your-subdomain.duckdns.org',
       ),
     ```

---

### Step 2: Adjusting the Firewall with iptables

#### Goal
Restrict NextCloud access (via Nginx Proxy Manager on ports 80 and 443) to only WireGuard VPN clients, using the WireGuard subnet (e.g., `10.13.13.0/24` from the default linuxserver/wireguard config).

#### Firewall Setup
1. **Check WireGuard Subnet**
   - Look at `./wireguard/config/wg0.conf` or client configs (e.g., `AllowedIPs = 10.13.13.0/24`). The default is often `10.13.13.0/24`; adjust commands below if yours differs.

2. **Install iptables (if needed)**
   - On Ubuntu/Debian: `sudo apt update && sudo apt install iptables`
   - On Fedora: `sudo dnf install iptables`

3. **Flush Existing Rules (Optional)**
   - Backup current rules: `sudo iptables-save > iptables-backup.txt`
   - Flush: `sudo iptables -F` (careful, this clears all rules; skip if you have other critical rules).

4. **Add Firewall Rules**
   - Allow WireGuard traffic (port 51820):
     ```bash
     sudo iptables -A INPUT -p udp --dport 51820 -j ACCEPT
     ```
   - Allow HTTP/HTTPS only from WireGuard subnet:
     ```bash
     sudo iptables -A INPUT -s 10.13.13.0/24 -p tcp -m multiport --dports 80,443 -j ACCEPT
     ```
   - Drop all other HTTP/HTTPS traffic:
     ```bash
     sudo iptables -A INPUT -p tcp -m multiport --dports 80,443 -j DROP
     ```
   - Allow loopback (local access):
     ```bash
     sudo iptables -A INPUT -i lo -j ACCEPT
     ```
   - Allow established/related connections (for general networking):
     ```bash
     sudo iptables -A INPUT -m conntrack --ctstate ESTABLISHED,RELATED -j ACCEPT
     ```

5. **Save Rules**
   - On Ubuntu/Debian: `sudo iptables-save > /etc/iptables/rules.v4`
   - On Fedora: `sudo iptables-save > /etc/sysconfig/iptables`
   - Ensure persistence: Install `iptables-persistent` (Debian/Ubuntu) or configure a systemd service if needed.

6. **Test Firewall**
   - Disconnect from WireGuard and try `https://your-subdomain.duckdns.org`—it should fail.
   - Connect to WireGuard and try again—it should work.

#### Alternative: Using ufw
If you prefer `ufw` (simpler firewall frontend):
```bash
sudo ufw allow 51820/udp
sudo ufw allow from 10.13.13.0/24 to any port 80
sudo ufw allow from 10.13.13.0/24 to any port 443
sudo ufw deny 80
sudo ufw deny 443
sudo ufw enable
```

---

### Final Notes
- **HTTPS**: Nginx Proxy Manager handles SSL termination, so NextCloud stays on HTTP internally, which is fine. Certificates auto-renew every 90 days.
- **Firewall**: These rules assume a basic setup. If you’re behind a router, forward ports 80, 443, and 51820/udp to your server, and tweak rules if other services need access.
- **Troubleshooting**:
  - SSL fails? Check port 80 is reachable during cert issuance (`docker logs nginx-proxy-manager`).
  - Firewall blocks too much? Review rules with `sudo iptables -L -v`.

You’re now running NextCloud over HTTPS, accessible only via WireGuard! Let me know if you hit any issues or want further refinements.


# Tempo jmcjmcjmc

### Key Points
- Install WireGuard, NextCloud, and DuckDNS using Docker for a secure, self-hosted cloud with dynamic DNS.
- Use official or trusted Docker images: linuxserver/duckdns, linuxserver/wireguard, and nextcloud/nextcloud.
- Configure firewall to restrict NextCloud access to WireGuard VPN clients for enhanced security.
- **Surprising detail**: DuckDNS is a service, not software, so you use a Docker image to update your IP dynamically.

---

### Getting Started with Docker Installations

#### Overview
This guide helps you set up WireGuard (a secure VPN), NextCloud (a self-hosted cloud storage), and DuckDNS (dynamic DNS) using Docker. These services work together to provide a secure way to access your cloud storage remotely, with DuckDNS ensuring your domain name always points to your changing IP address, and WireGuard securing the connection.

#### Step-by-Step Installation

1. **Set Up DuckDNS**
   - Sign up at [DuckDNS](https://www.duckdns.org) to get a token and subdomain.
   - Pull the Docker image: `docker pull linuxserver/duckdns`.
   - Run the container with your token and subdomain:
     ```bash
     docker run -d --name duckdns -e TOKEN=your_token -e SUBDOMAINS=your_subdomain -v /path/to/config:/config --restart unless-stopped linuxserver/duckdns
     ```
   - This keeps your domain updated with your current IP address.

2. **Set Up WireGuard**
   - Pull the Docker image: `docker pull linuxserver/wireguard`.
   - Run the container, ensuring network admin capabilities:
     ```bash
     docker run -d --name wireguard --cap-add NET_ADMIN --cap-add SYS_MODULE -v /path/to/wireguard/config:/etc/wireguard -v /lib/modules:/lib/modules -p 51820:51820/UDP --restart unless-stopped linuxserver/wireguard
     ```
   - Configure the server by editing `/etc/wireguard/wg0.conf` inside the container to set up VPN settings and add clients.

3. **Set Up NextCloud**
   - First, set up a MariaDB database:
     ```bash
     docker run -d --name mariadb -e MYSQL_ROOT_PASSWORD=your_password -e MYSQL_DATABASE=nextcloud -e MYSQL_USER=nextcloud -e MYSQL_PASSWORD=nextcloud_password -v /path/to/mariadb/data:/var/lib/mysql --restart unless-stopped mariadb:10.5
     ```
   - Pull the NextCloud image: `docker pull nextcloud/nextcloud`.
   - Run NextCloud, linking to the database:
     ```bash
     docker run -d --name nextcloud --link mariadb:mysql -e MYSQL_HOST=mysql -e MYSQL_DATABASE=nextcloud -e MYSQL_USER=nextcloud -e MYSQL_PASSWORD=nextcloud_password -v /path/to/nextcloud/data:/var/lib/nextcloud -v /path/to/nextcloud/config:/etc/nextcloud -p 80:80 -p 443:443 --restart unless-stopped nextcloud/nextcloud
     ```
   - Access NextCloud via your domain name, secured by the VPN.

4. **Configure Host Firewall**
   - Ensure only WireGuard VPN clients can access NextCloud by restricting ports 80 and 443. Assuming WireGuard subnet is `10.0.0.0/24`, use:
     ```bash
     iptables -A INPUT -s 10.0.0.0/24 -p tcp --dport 80 -j ACCEPT
     iptables -A INPUT -s 10.0.0.0/24 -p tcp --dport 443 -j ACCEPT
     iptables -A INPUT -p tcp --dport 80 -j DROP
     iptables -A INPUT -p tcp --dport 443 -j DROP
     ```
   - Adjust the subnet based on your WireGuard setup.

5. **Configure WireGuard Clients**
   - Install WireGuard on client devices (see [WireGuard](https://www.wireguard.com/install) for instructions).
   - Set up client configurations with server details and connect to the VPN.
   - Access NextCloud via the DuckDNS domain name; traffic will be secure through the VPN.

---

---

### Survey Note: Detailed Installation Guide for WireGuard, NextCloud, and DuckDNS on Docker

This comprehensive guide outlines the installation and configuration of WireGuard, NextCloud, and DuckDNS using Docker, ensuring a secure, self-hosted cloud storage solution with dynamic DNS capabilities. Each step is detailed to cater to users with varying levels of technical expertise, leveraging official and trusted Docker images for reliability.

#### Introduction
The integration of WireGuard, NextCloud, and DuckDNS via Docker provides a robust setup for remote access to personal cloud storage, secured by a VPN and managed through dynamic DNS. WireGuard offers a fast, secure VPN tunnel, NextCloud serves as a self-hosted cloud storage alternative to services like Dropbox, and DuckDNS ensures domain name resolution despite changing IP addresses. This setup is particularly useful for home servers or VPS environments where IP addresses may not be static.

#### Background and Rationale
WireGuard, introduced as a modern VPN protocol, utilizes state-of-the-art cryptography and is designed for simplicity and performance, often outperforming OpenVPN and IPsec. NextCloud, an open-source platform, offers features akin to Google Drive or Microsoft 365, with the advantage of hosting on your own server for enhanced privacy and control. DuckDNS, a free dynamic DNS service, is essential for maintaining domain accessibility with dynamic IPs, commonly encountered in residential internet connections.

The decision to use Docker for these installations stems from its containerization benefits, including ease of deployment, isolation, and portability. Docker images from trusted sources like LinuxServer.io and the official NextCloud repository ensure security and regular updates.

#### Detailed Installation Steps

##### 1. DuckDNS Setup
DuckDNS is not software to install but a service requiring a client to update your IP. The linuxserver/duckdns Docker image is recommended for this purpose, as it is well-maintained and documented.

- **Sign Up and Obtain Credentials**: Visit [DuckDNS](https://www.duckdns.org) to create an account, obtain a token, and choose a subdomain (e.g., `yourname.duckdns.org`).
- **Pull the Docker Image**: Execute `docker pull linuxserver/duckdns` to download the image.
- **Run the Container**: Use the following command, replacing `your_token` and `your_subdomain` with your credentials:
  ```bash
  docker run -d --name duckdns -e TOKEN=your_token -e SUBDOMAINS=your_subdomain -v /path/to/config:/config --restart unless-stopped linuxserver/duckdns
  ```
  - The `-v` flag mounts a configuration directory for persistence, and `--restart unless-stopped` ensures the container restarts on system reboot.
- **Verification**: Check logs with `docker logs duckdns` to ensure updates are occurring, typically every 5 minutes by default.

Documentation for this setup is available at [linuxserver/duckdns documentation](https://docs.linuxserver.io/images/docker-duckdns/).

##### 2. WireGuard Setup
WireGuard requires the kernel module to be loaded, which can be managed within Docker using the linuxserver/wireguard image, known for its reliability and support for multiple architectures.

- **Pull the Docker Image**: Run `docker pull linuxserver/wireguard` to fetch the image.
- **Run the Container**: Launch with necessary capabilities and port mappings:
  ```bash
  docker run -d --name wireguard --cap-add NET_ADMIN --cap-add SYS_MODULE -v /path/to/wireguard/config:/etc/wireguard -v /lib/modules:/lib/modules -p 51820:51820/UDP --restart unless-stopped linuxserver/wireguard
  ```
  - `--cap-add NET_ADMIN` and `--cap-add SYS_MODULE` are required for network administration and kernel module loading, respectively.
  - `-p 51820:51820/UDP` maps the default WireGuard port, adjustable based on your needs.
- **Configuration**: Access the container’s `/etc/wireguard/wg0.conf` to set up the server. Generate keys using `wg genkey` and `wg pubkey` for server and clients, defining peers and subnets (e.g., `10.0.0.0/24` for VPN IPs).
- **Client Setup**: Distribute client configurations, ensuring they connect to the server’s public IP on port 51820.

Refer to [linuxserver/wireguard documentation](https://docs.linuxserver.io/images/docker-wireguard/) for detailed configuration options.

##### 3. NextCloud Setup
NextCloud requires a database backend, commonly MariaDB, and can be deployed using the official nextcloud/nextcloud image for ease of use.

- **Database Setup (MariaDB)**:
  - Run the MariaDB container:
    ```bash
    docker run -d --name mariadb -e MYSQL_ROOT_PASSWORD=your_password -e MYSQL_DATABASE=nextcloud -e MYSQL_USER=nextcloud -e MYSQL_PASSWORD=nextcloud_password -v /path/to/mariadb/data:/var/lib/mysql --restart unless-stopped mariadb:10.5
    ```
  - Replace passwords and paths as needed, ensuring data persistence with the volume mount.
- **Pull NextCloud Image**: Execute `docker pull nextcloud/nextcloud` to get the latest image.
- **Run NextCloud Container**: Link to MariaDB and map ports:
  ```bash
  docker run -d --name nextcloud --link mariadb:mysql -e MYSQL_HOST=mysql -e MYSQL_DATABASE=nextcloud -e MYSQL_USER=nextcloud -e MYSQL_PASSWORD=nextcloud_password -v /path/to/nextcloud/data:/var/lib/nextcloud -v /path/to/nextcloud/config:/etc/nextcloud -p 80:80 -p 443:443 --restart unless-stopped nextcloud/nextcloud
  ```
  - Ports 80 and 443 are mapped for HTTP and HTTPS access, respectively, aligning with standard web traffic.
- **Initial Setup**: Access via your DuckDNS domain (e.g., `http://yourname.duckdns.org`) to complete the web-based installation, setting up admin credentials and additional configurations.

Consult [NextCloud official documentation](https://docs.nextcloud.com/server/latest/admin_manual/installation/index.html) for further customization.

##### 4. Firewall Configuration
To ensure NextCloud is only accessible via the WireGuard VPN, configure the host’s firewall to restrict access to ports 80 and 443 to the WireGuard subnet.

- **Example Configuration**: Assuming WireGuard subnet is `10.0.0.0/24`, use iptables:
  ```bash
  iptables -A INPUT -s 10.0.0.0/24 -p tcp --dport 80 -j ACCEPT
  iptables -A INPUT -s 10.0.0.0/24 -p tcp --dport 443 -j ACCEPT
  iptables -A INPUT -p tcp --dport 80 -j DROP
  iptables -A INPUT -p tcp --dport 443 -j DROP
  ```
- This setup blocks external access to NextCloud, allowing only VPN clients with IPs in the specified subnet.

Note: Ensure the host machine supports iptables or use a firewall manager like `ufw` if preferred, adjusting commands accordingly.

##### 5. Client Configuration and Access
Clients need WireGuard installed to connect to the VPN, then access NextCloud securely.

- **Install WireGuard**: Follow [WireGuard](https://www.wireguard.com/install) for client installation on Windows, macOS, Linux, or mobile devices.
- **Configure Client**: Use the client configuration file provided by the server, connecting to the server’s public IP on port 51820.
- **Access NextCloud**: Once connected, access via the DuckDNS domain (e.g., `http://yourname.duckdns.org`), with traffic routed through the VPN for security.

#### Integration and Security Considerations
The integration ensures that DuckDNS updates the domain to point to the server’s public IP, WireGuard secures the connection by requiring VPN access, and NextCloud serves as the cloud storage endpoint. The firewall configuration is crucial, restricting NextCloud access to VPN clients, enhancing security against unauthorized access.

A surprising detail is that DuckDNS, often misunderstood as software to install, is a service requiring a client like the Docker image to update IPs dynamically, which is essential for home servers with dynamic IPs.

#### Tables for Clarity

| Service       | Docker Image               | Key Configuration Steps                          | Ports Mapped       |
|---------------|----------------------------|--------------------------------------------------|--------------------|
| DuckDNS       | linuxserver/duckdns        | Set TOKEN, SUBDOMAINS, mount config volume       | None (background)  |
| WireGuard     | linuxserver/wireguard      | Configure wg0.conf, map 51820/UDP                | 51820/UDP          |
| NextCloud     | nextcloud/nextcloud        | Link to MariaDB, map 80:80, 443:443              | 80, 443            |

| Firewall Rule Example (iptables) | Description                                      |
|----------------------------------|--------------------------------------------------|
| -A INPUT -s 10.0.0.0/24 -p tcp --dport 80 -j ACCEPT | Allow HTTP from WireGuard subnet                 |
| -A INPUT -s 10.0.0.0/24 -p tcp --dport 443 -j ACCEPT | Allow HTTPS from WireGuard subnet                |
| -A INPUT -p tcp --dport 80 -j DROP                   | Drop HTTP from other sources                     |
| -A INPUT -p tcp --dport 443 -j DROP                  | Drop HTTPS from other sources                    |

#### Conclusion
This setup provides a secure, self-hosted cloud solution with dynamic DNS, leveraging Docker for ease of management. Ensure regular updates to Docker images and monitor logs for any issues, particularly with firewall rules and VPN connectivity.

#### Key Citations
- [linuxserver/duckdns documentation](https://docs.linuxserver.io/images/docker-duckdns/)
- [linuxserver/wireguard documentation](https://docs.linuxserver.io/images/docker-wireguard/)
- [NextCloud official documentation](https://docs.nextcloud.com/server/latest/admin_manual/installation/index.html)
- [WireGuard installation guide](https://www.wireguard.com/install/)