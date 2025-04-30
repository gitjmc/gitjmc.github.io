---
title: Mosquitto with Docker Compose, Base Authentication, DuckDNS and Let's Encrypt (done)
author:
  name: 
  link: 
date: 2025-04-26 07:35:00 -0400
categories: [DEV]
tags: [cli]
render_with_liquid: false
---

# Base Authentication & Let's Encrypt (DuckDNS)

Dockerize Mosquitto with duckdns, base authentication and letsencrypt.

## Prerequisites

- A domain pointed to your droplet (via DuckDNS)
- Docker and Docker Compose installed
- Ports 80, 8883, 9001, and 1883 open in your firewall

## Step 1: Set Up 
Besmellah!

## Step 3: step 1 compose.yaml version 1

run mosquitto on 1883 wihout base authentication

Create a working directory:

```bash
mkdir -p ~/mosquitto ~/mosquitto/data ~/mosquitto/log
cd ~/mosquitto
vim mosquitto.conf
```

Create `mosquitto/mosquitto.conf`:
```
persistence true
persistence_location /mosquitto/data/
log_dest file /mosquitto/log/mosquitto.log

# Authentication
allow_anonymous true

# MQTT
listener 1883
protocol mqtt

# Websockets
listener 9001
protocol websockets
```

Create `mosquitto/compose.yaml`:
```yaml
services:
  mosquitto:
    image: eclipse-mosquitto:2
    container_name: mosquitto
    ports:
      - "1883:1883"
      - "9001:9001"
    volumes:
      - $PWD/mosquitto.conf:/mosquitto/config/mosquitto.conf
      - $PWD/data:/mosquitto/data
      - $PWD/log:/mosquitto/log
    environment:
      - TZ=UTC+5
    restart: always

```

### test mosquitto 


```bash
docker ps
```

check the logs
```bash
sudo cat log/mosquitto.log
```

check into container
```bash
docker exec -it mosquitto sh
```

check open ports
```bash
sudo ufw status
```

## 4. Create Authentication File

```bash
cd ~
docker run -it --rm -v $(pwd)/mosquitto:/mosquitto/config eclipse-mosquitto:2 mosquitto_passwd -c /mosquitto/config/passwd YOUR_USERNAME

```

Replace YOUR_USERNAME and Enter a strong password.

### How to validate

passwd file will be created
```bash
sudo cat passwd
```


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

tested version

Install Certbot:

```bash
sudo apt update
sudo apt install snapd
sudo snap install snap-store
sudo snap install --classic certbot
```

Run Certbot:

```bash
sudo certbot certonly --standalone --preferred-challenges http --email YOUR_EMAIL -d mqttsisi.duckdns.org
```
TODO how to test ...

Move certificates:

```bash
mkdir -p ~/mosquitto/certs
sudo cp /etc/letsencrypt/live/mqttsisi.duckdns.org/fullchain.pem ~/mosquitto/certs/
sudo cp /etc/letsencrypt/live/mqttsisi.duckdns.org/privkey.pem ~/mosquitto/certs/
sudo cp /etc/letsencrypt/live/mqttsisi.duckdns.org/cert.pem ~/mosquitto/certs/
```
TODO how to test ...


## 6. Create Docker Compose File

Go back to your home directory and create docker-compose.yml:

```bash
cd ~/mosquitto
vim compose.yaml
```

Paste:

```yaml
services:
  mosquitto:
    image: eclipse-mosquitto:2
    container_name: mosquitto
    ports:
      - "1883:1883"
      - "8883:8883"
      - "9001:9001"
    volumes:
      - $PWD/mosquitto.conf:/mosquitto/config/mosquitto.conf
      - $PWD/passwd:/mosquitto/config/passwd  
      - $PWD/data:/mosquitto/data
      - $PWD/log:/mosquitto/log
      - $PWD/certs:/mosquitto/letsencrypt
    environment:
      - TZ=UTC+5
    restart: always

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
cp /etc/letsencrypt/live/YOUR_SUBDOMAIN.duckdns.org/fullchain.pem ~/mosquitto/certs/
cp /etc/letsencrypt/live/YOUR_SUBDOMAIN.duckdns.org/privkey.pem ~/mosquitto/certs/
cp /etc/letsencrypt/live/mqttsisi.duckdns.org/cert.pem ~/mosquitto/certs/
docker restart mosquitto
```

Save and make it executable:

```bash
#!/bin/bash
sudo chmod +x /etc/cron.daily/certbot-renew
```
