---
title: Mosquitto with Docker Compose and Base Authentication & Let's Encrypt (DuckDNS)
author:
  name: 
  link: 
date: 2025-04-01 07:35:00 -0400
categories: [DEV]
tags: [cli]
render_with_liquid: false
---

# Base Authentication & Let's Encrypt (DuckDNS)

Here's a comprehensive guide to set up Mosquitto with Docker Compose using base authentication and Let's Encrypt with a DuckDNS domain on a clean droplet.

## Prerequisites

- A domain pointed to your droplet (via DuckDNS)
- Docker and Docker Compose installed
- Ports 80, 443, and 1883 open in your firewall

## Step 1: Set Up DuckDNS (if not already done)

1. Create a DuckDNS account and claim your subdomain
2. Install DuckDNS updater (optional for dynamic IP):
```bash
docker run -d --name duckdns \
  -e PUID=1000 \
  -e PGID=1000 \
  -e TZ=UTC \
  -e SUBDOMAINS=yourdomain \
  -e TOKEN=your-duckdns-token \
  -e LOG_FILE=false \
  --restart unless-stopped \
  linuxserver/duckdns
```

## Step 2: Create Docker Compose File

Create a `docker-compose.yml` file with the following content:

```yaml
version: '3.8'

services:
  mosquitto:
    image: eclipse-mosquitto:2
    container_name: mosquitto
    restart: unless-stopped
    ports:
      - "1883:1883"
      - "9001:9001"
    volumes:
      - ./mosquitto/config:/mosquitto/config
      - ./mosquitto/data:/mosquitto/data
      - ./mosquitto/log:/mosquitto/log
    environment:
      - TZ=UTC
    depends_on:
      - nginx-proxy

  nginx-proxy:
    image: nginxproxy/nginx-proxy
    container_name: nginx-proxy
    restart: unless-stopped
    ports:
      - "80:80"
      - "443:443"
    volumes:
      - ./certs:/etc/nginx/certs
      - ./vhost.d:/etc/nginx/vhost.d
      - /var/run/docker.sock:/tmp/docker.sock:ro
      - ./html:/usr/share/nginx/html
    environment:
      - DEFAULT_HOST=yourdomain.duckdns.org

  letsencrypt:
    image: nginxproxy/acme-companion
    container_name: nginx-proxy-letsencrypt
    restart: unless-stopped
    volumes:
      - ./certs:/etc/nginx/certs
      - ./acme.sh:/etc/acme.sh
      - /var/run/docker.sock:/var/run/docker.sock:ro
      - ./vhost.d:/etc/nginx/vhost.d
      - ./html:/usr/share/nginx/html
    environment:
      - DEFAULT_EMAIL=your@email.com
      - NGINX_PROXY_CONTAINER=nginx-proxy
```

## Step 3: Create Mosquitto Configuration

Create the directory structure and config file:
```bash
mkdir -p mosquitto/config mosquitto/data mosquitto/log
```

Create `mosquitto/config/mosquitto.conf`:
```
persistence true
persistence_location /mosquitto/data/
log_dest file /mosquitto/log/mosquitto.log

# Authentication
allow_anonymous false
password_file /mosquitto/config/passwd

# MQTT
listener 1883
protocol mqtt

# Websockets
listener 9001
protocol websockets
```

## Step 4: Set Up Authentication

1. Create a password file (replace username/password):
```bash
docker run -it --rm -v $(pwd)/mosquitto/config:/mosquitto/config eclipse-mosquitto:2 mosquitto_passwd -c /mosquitto/config/passwd username
```
(You'll be prompted to enter and confirm the password)

2. For additional users (without -c flag to avoid overwriting):
```bash
docker run -it --rm -v $(pwd)/mosquitto/config:/mosquitto/config eclipse-mosquitto:2 mosquitto_passwd /mosquitto/config/passwd anotheruser
```

## Step 5: Set Up Environment

Create a `.env` file:
```
VIRTUAL_HOST=yourdomain.duckdns.org
LETSENCRYPT_HOST=yourdomain.duckdns.org
LETSENCRYPT_EMAIL=your@email.com
```

## Step 6: Start the Services

```bash
docker compose up -d
```

## Step 7: Verify Setup

1. Check logs for errors:
```bash
docker logs mosquitto
docker logs nginx-proxy
docker logs nginx-proxy-letsencrypt
```

2. Test MQTT connection:
```bash
mosquitto_sub -h yourdomain.duckdns.org -t test -u username -P password
```
(In another terminal)
```bash
mosquitto_pub -h yourdomain.duckdns.org -t test -m "hello" -u username -P password
```

## Common Issues and Solutions

1. **Certificate errors**: 
   - Ensure your domain resolves correctly to the server IP
   - Check port 80 is open and accessible
   - Wait a few minutes for Let's Encrypt to issue certificates

2. **Authentication failures**:
   - Verify the password file exists and has correct permissions
   - Check the password file path in mosquitto.conf

3. **Connection issues**:
   - Verify ports are open in firewall: `sudo ufw allow 80,443,1883/tcp`
   - Check Mosquitto is listening: `netstat -tulnp | grep mosquitto`

4. **DuckDNS updates**:
   - If your IP changes frequently, ensure the DuckDNS updater is running

This setup provides a secure Mosquitto broker with:
- TLS encryption via Let's Encrypt
- Basic authentication
- WebSockets support
- Automatic certificate renewal
- Reverse proxy for additional security

----

# Base Authentication & Let's Encrypt (Direct Setup)

Here's a simplified setup that integrates Let's Encrypt directly with Mosquitto without using Nginx as a reverse proxy.

## Prerequisites
- A domain pointed to your droplet (via DuckDNS)
- Docker and Docker Compose installed
- Ports 80 (temporary for certbot), 443, and 1883 open in your firewall

## Step 1: Create Docker Compose File

Create a `docker-compose.yml` file with:

```yaml
version: '3.8'

services:
  mosquitto:
    image: eclipse-mosquitto:2
    container_name: mosquitto
    restart: unless-stopped
    ports:
      - "1883:1883"
      - "8883:8883"  # SSL port
      - "80:80"      # Temporary for certbot
      - "9001:9001"  # WebSocket
    volumes:
      - ./mosquitto/config:/mosquitto/config
      - ./mosquitto/data:/mosquitto/data
      - ./mosquitto/log:/mosquitto/log
      - ./letsencrypt:/etc/letsencrypt
    environment:
      - TZ=UTC
    command: /bin/sh -c "(if [ ! -f /etc/letsencrypt/live/yourdomain.duckdns.org/cert.pem ]; then sleep 30; fi) && /usr/sbin/mosquitto -c /mosquitto/config/mosquitto.conf"

  certbot:
    image: certbot/certbot
    container_name: certbot
    volumes:
      - ./letsencrypt:/etc/letsencrypt
      - ./mosquitto/config:/mosquitto/config
    entrypoint: "/bin/sh -c"
    command: |
      "
      if [ ! -f /etc/letsencrypt/live/yourdomain.duckdns.org/cert.pem ]; then
        certbot certonly --standalone --non-interactive --agree-tos --email your@email.com -d yourdomain.duckdns.org;
        ln -sf /etc/letsencrypt/live/yourdomain.duckdns.org/cert.pem /mosquitto/config/server.crt;
        ln -sf /etc/letsencrypt/live/yourdomain.duckdns.org/privkey.pem /mosquitto/config/server.key;
      fi &&
      while :; do
        sleep 12h & wait $${!};
        certbot renew;
      done
      "
    depends_on:
      - mosquitto
```

## Step 2: Create Mosquitto Configuration

Create the directory structure:
```bash
mkdir -p mosquitto/{config,data,log} letsencrypt
```

Create `mosquitto/config/mosquitto.conf`:
```conf
persistence true
persistence_location /mosquitto/data/
log_dest file /mosquitto/log/mosquitto.log

# Authentication
allow_anonymous false
password_file /mosquitto/config/passwd

# MQTT
listener 1883
protocol mqtt

# MQTT over SSL
listener 8883
protocol mqtt
certfile /mosquitto/config/server.crt
keyfile /mosquitto/config/server.key

# WebSockets
listener 9001
protocol websockets
```

## Step 3: Set Up Authentication

Create password file (replace username/password):
```bash
docker run -it --rm -v $(pwd)/mosquitto/config:/mosquitto/config eclipse-mosquitto:2 mosquitto_passwd -c /mosquitto/config/passwd username
```

## Step 4: Start the Services

```bash
docker compose up -d
```

## Step 5: Verify Setup

1. Check certificates were generated:
```bash
docker exec mosquitto ls -la /mosquitto/config/
```
(Should see server.crt and server.key symlinks)

2. Test SSL connection:
```bash
mosquitto_sub -h yourdomain.duckdns.org -t test -p 8883 --capath /etc/ssl/certs/ -u username -P password
```
(In another terminal)
```bash
mosquitto_pub -h yourdomain.duckdns.org -t test -m "hello" -p 8883 --capath /etc/ssl/certs/ -u username -P password
```

## Step 6: Clean Up Temporary Port

After certificates are generated (check with `docker logs certbot`), you can:
1. Remove port 80 from mosquitto service in docker-compose.yml
2. Update the file and run:
```bash
docker compose down && docker compose up -d
```

## Certificate Renewal

The certbot container will automatically:
1. Check for certificates on startup
2. Renew certificates when needed (every 12 hours)
3. Update the symlinks Mosquitto uses

## Security Considerations

1. After setup, close port 80 if not needed:
```bash
sudo ufw deny 80/tcp
```

2. Consider adding additional security to your Mosquitto configuration:
```conf
# Add to mosquitto.conf
require_certificate false
tls_version tlsv1.2
```

This direct approach is simpler than using Nginx as a reverse proxy while still providing:
- Automatic Let's Encrypt certificate management
- Secure MQTT over SSL (port 8883)
- Basic authentication
- WebSockets support
- Automatic certificate renewal



---

# Base authentication ... From chatGpt
To set up Mosquitto with Docker Compose, basic authentication, and Let's Encrypt on a clean droplet using a DuckDNS domain, follow these optimized steps:

## 1. Install Docker & Docker Compose

Ensure your droplet has Docker and Docker Compose installed:

```bash
sudo apt update && sudo apt upgrade -y
sudo apt install docker.io -y
sudo systemctl enable --now docker
sudo usermod -aG docker $USER
```

Install Docker Compose:

sudo apt install docker-compose -y

## 2. Set Up DuckDNS

Register a free subdomain on DuckDNS and get your token.

Create a DuckDNS update script:

```bash
mkdir ~/duckdns && cd ~/duckdns
nano update.sh
```

Add:

#!/bin/bash
echo url="https://www.duckdns.org/update?domains=YOUR_SUBDOMAIN&token=YOUR_DUCKDNS_TOKEN&ip=" | curl -k -o ~/duckdns/duck.log -K -

Save and exit.

Make it executable:

```bash
chmod +x update.sh
```

Run it manually to test:

./update.sh

Set up a cron job to update it every 5 minutes:

(crontab -l ; echo "*/5 * * * * ~/duckdns/update.sh >/dev/null 2>&1") | crontab -

## 3. Prepare Mosquitto Configuration

Create a working directory:

```bash
mkdir -p ~/mosquitto/config ~/mosquitto/data ~/mosquitto/log
cd ~/mosquitto/config
nano mosquitto.conf
```

Paste this config:

```conf
listener 1883
allow_anonymous false
password_file /mosquitto/config/passwords

listener 8883
certfile /mosquitto/config/certs/fullchain.pem
keyfile /mosquitto/config/certs/privkey.pem
```

Save and exit.

## 4. Create Authentication File

docker run --rm eclipse-mosquitto mosquitto_passwd -c /mosquitto/config/passwords YOUR_USERNAME

Enter a strong password.

## 5. Set Up Let's Encrypt SSL (Using Certbot)

Install Certbot:

```bash
sudo apt install certbot -y
```

Run Certbot:

```bash
sudo certbot certonly --standalone --preferred-challenges http --agree-tos --email YOUR_EMAIL -d YOUR_SUBDOMAIN.duckdns.org
```

Move certificates:

```bash
mkdir -p ~/mosquitto/config/certs
sudo cp /etc/letsencrypt/live/YOUR_SUBDOMAIN.duckdns.org/fullchain.pem ~/mosquitto/config/certs/
sudo cp /etc/letsencrypt/live/YOUR_SUBDOMAIN.duckdns.org/privkey.pem ~/mosquitto/config/certs/
```

## 6. Create Docker Compose File

Go back to your home directory and create docker-compose.yml:

```bash
cd ~/mosquitto
nano docker-compose.yml
```

Paste:

```yaml
version: '3'
services:
  mosquitto:
    image: eclipse-mosquitto
    container_name: mosquitto
    restart: always
    volumes:
      - ./config:/mosquitto/config
      - ./data:/mosquitto/data
      - ./log:/mosquitto/log
    ports:
      - "1883:1883"
      - "8883:8883"
```

Save and exit.

## 7. Start Mosquitto

Run:

```bash
docker-compose up -d
```

## 8. Auto-Renew SSL

Create a script:

```bash
sudo nano /etc/cron.daily/certbot-renew
```

Add:

```bash
#!/bin/bash
certbot renew --quiet
cp /etc/letsencrypt/live/YOUR_SUBDOMAIN.duckdns.org/fullchain.pem ~/mosquitto/config/certs/
cp /etc/letsencrypt/live/YOUR_SUBDOMAIN.duckdns.org/privkey.pem ~/mosquitto/config/certs/
docker restart mosquitto
```

Save and make it executable:

```bash
#!/bin/bash
sudo chmod +x /etc/cron.daily/certbot-renew
```

This setup ensures secure MQTT connections with authentication and automatic SSL renewal.

Are you getting specific errors? I can help debug.


---

# From grok
Setting up Mosquitto with Docker Compose, basic authentication, Let's Encrypt certificates, and a DuckDNS domain on a clean DigitalOcean droplet can be a powerful way to create a secure MQTT broker. Below, I’ll guide you through a streamlined, optimal approach to achieve this. I’ll assume you’re using Ubuntu (common for DigitalOcean droplets) and aim to avoid common pitfalls that might be causing your errors. If you share specific errors you're encountering, I can refine this further!

---

### Prerequisites
1. **Clean Droplet**: Ubuntu 22.04 (or similar) installed.
2. **Domain**: A DuckDNS subdomain (e.g., `yourname.duckdns.org`).
3. **Docker & Docker Compose**: Installed on the droplet.
4. **Ports**: Ensure ports 80, 443, and 8883 are open in your droplet’s firewall and DigitalOcean networking settings.

---

### Step-by-Step Guide

#### 1. Install Docker and Docker Compose
Log in to your droplet via SSH and run the following commands to install Docker and Docker Compose:

```bash
sudo apt update && sudo apt upgrade -y
sudo apt install -y docker.io docker-compose
sudo systemctl start docker
sudo systemctl enable docker
sudo usermod -aG docker $USER  # Log out and back in after this
```

Verify installation:
```bash
docker --version
docker-compose --version
```

#### 2. Set Up DuckDNS
- Go to [DuckDNS.org](https://www.duckdns.org/), log in, and create a subdomain (e.g., `yourname.duckdns.org`).
- Note your DuckDNS token (you’ll need it later).
- Update your DuckDNS domain to point to your droplet’s public IP. You can automate this with a script or Docker container, but for now, manually set it on the DuckDNS website.

#### 3. Create Project Directory
Create a directory to hold your Mosquitto setup:
```bash
mkdir ~/mosquitto && cd ~/mosquitto
```

#### 4. Configure Docker Compose
Create a `docker-compose.yml` file with Mosquitto and a Certbot service for Let's Encrypt certificates:

```yaml
version: '3.8'

services:
  mosquitto:
    image: eclipse-mosquitto:latest
    container_name: mosquitto
    restart: unless-stopped
    ports:
      - "1883:1883"  # Unsecured MQTT (optional, for testing)
      - "8883:8883"  # Secured MQTT
      - "80:80"      # Temporary for Certbot HTTP challenge
    volumes:
      - ./mosquitto.conf:/mosquitto/config/mosquitto.conf
      - ./passwd:/mosquitto/config/passwd
      - ./letsencrypt:/etc/letsencrypt
    depends_on:
      - certbot

  certbot:
    image: certbot/certbot
    container_name: certbot
    volumes:
      - ./letsencrypt:/etc/letsencrypt
    command: certonly --standalone -d yourname.duckdns.org --email your.email@example.com --agree-tos --non-interactive
    depends_on:
      - mosquitto
```

**Notes**:
- Replace `yourname.duckdns.org` with your DuckDNS subdomain.
- Replace `your.email@example.com` with your email for certificate notifications.
- Port 80 is exposed temporarily for Certbot’s HTTP-01 challenge. After certificates are obtained, you can remove it.

#### 5. Configure Mosquitto
Create a `mosquitto.conf` file in the `~/mosquitto` directory:

```bash
nano mosquitto.conf
```

Add the following configuration:
```
# General settings
persistence true
persistence_location /mosquitto/data/
log_dest file /mosquitto/log/mosquitto.log

# Unsecured listener (optional, for testing)
listener 1883
allow_anonymous true

# Secured listener with TLS
listener 8883
allow_anonymous false
password_file /mosquitto/config/passwd
cafile /etc/letsencrypt/live/yourname.duckdns.org/chain.pem
certfile /etc/letsencrypt/live/yourname.duckdns.org/fullchain.pem
keyfile /etc/letsencrypt/live/yourname.duckdns.org/privkey.pem
```

- **1883**: Unsecured MQTT for local testing (optional; remove later for security).
- **8883**: Secured MQTT with TLS and authentication.
- Replace `yourname.duckdns.org` with your DuckDNS domain.

#### 6. Set Up Authentication
Create a password file for Mosquitto:
```bash
touch passwd
docker run --rm -it -v $(pwd)/passwd:/passwd eclipse-mosquitto mosquitto_passwd -b /passwd username password
```
- Replace `username` and `password` with your desired credentials.
- This adds a hashed password to the `passwd` file.

#### 7. Open Firewall Ports
Ensure the necessary ports are open:
```bash
sudo ufw allow 80
sudo ufw allow 443
sudo ufw allow 8883
sudo ufw status
```
Also, configure DigitalOcean’s firewall (via the control panel) to allow these ports.

#### 8. Start Docker Compose
Run the setup:
```bash
docker-compose up -d
```

- Certbot will obtain the Let's Encrypt certificates and store them in the `letsencrypt` directory.
- Mosquitto will start with TLS enabled once certificates are available.

#### 9. Verify Mosquitto
Test the unsecured port (if enabled):
```bash
docker run --rm -it eclipse-mosquitto mosquitto_sub -h your_droplet_ip -p 1883 -t test/topic
```
In another terminal:
```bash
docker run --rm -it eclipse-mosquitto mosquitto_pub -h your_droplet_ip -p 1883 -t test/topic -m "Hello"
```

Test the secured port:
```bash
docker run --rm -it eclipse-mosquitto mosquitto_sub -h yourname.duckdns.org -p 8883 -u username -P password --cafile /etc/ssl/certs/ca-certificates.crt -t test/topic
```
- You may need to adjust the `--cafile` path depending on your client system’s CA store.

#### 10. Automate Certificate Renewal
Edit `docker-compose.yml` to remove the `80:80` port mapping from Mosquitto (after certificates are obtained). Then, set up a cron job for renewal:
```bash
crontab -e
```
Add:
```
0 0 * * * docker-compose -f /home/youruser/mosquitto/docker-compose.yml run --rm certbot renew --quiet && docker-compose -f /home/youruser/mosquitto/docker-compose.yml restart mosquitto
```
- Replace `/home/youruser/mosquitto` with your project path.

#### 11. Secure the Setup
- Remove the `listener 1883` section from `mosquitto.conf` if you don’t need unsecured access.
- Restart the service:
```bash
docker-compose down && docker-compose up -d
```

---

### Troubleshooting Common Errors
- **"Connection refused"**: Check if ports are open (`ufw` and DigitalOcean firewall), certificates are in place, and the domain resolves to your droplet’s IP.
- **"TLS error"**: Ensure the certificate paths in `mosquitto.conf` match the files in `./letsencrypt/live/yourname.duckdns.org/`.
- **Certbot fails**: Verify port 80 is accessible and the domain points to your droplet’s IP.

---

### Why This Approach?
- **Docker Compose**: Simplifies managing Mosquitto and Certbot together.
- **Basic Authentication**: Adds a security layer beyond TLS.
- **Let's Encrypt**: Provides free, trusted certificates.
- **DuckDNS**: Free dynamic DNS that integrates well with home servers or droplets.
- **Minimal Overhead**: No reverse proxy (e.g., Nginx/Traefik) unless needed for additional services.

Let me know your specific errors, and I’ll tailor this further!


