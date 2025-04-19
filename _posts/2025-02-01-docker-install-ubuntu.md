---
title: Install Docker on ubuntu and Deepin
author: 
  name: 
  link:
date: 2025-02-01 01:03:00 -0700
categories: [DEV, Linux]
tags: [docker, ubuntu, deepin]
math: true
mermaid: true
image:
  src: /commons/devices-mockup.png
  width: 400
  height: 250
---

Cet article montre comment peut-on installer docker dans Ubuntu[^ref1].

The best way to install Docker on Ubuntu is to use Docker's official installation method, which ensures you get the latest stable version and proper configuration. Below is a step-by-step guide to install Docker on Ubuntu:

---
## Ubuntu

### **Step 1: Update Your System**
Before installing Docker, ensure your system is up to date:
```bash
sudo apt update
sudo apt upgrade -y
```

---

### **Step 2: Install Required Dependencies**
Install packages necessary for Docker installation:
```bash
sudo apt install -y apt-transport-https ca-certificates curl software-properties-common
```

---

### **Step 3: Add Docker’s Official GPG Key**
Add Docker's GPG key to ensure the authenticity of the packages:
```bash
curl -fsSL https://download.docker.com/linux/ubuntu/gpg | sudo gpg --dearmor -o /usr/share/keyrings/docker-archive-keyring.gpg
```

---

### **Step 4: Add Docker Repository**
Add Docker's official repository to your system's package sources:
```bash
echo "deb [arch=$(dpkg --print-architecture) signed-by=/usr/share/keyrings/docker-archive-keyring.gpg] https://download.docker.com/linux/ubuntu $(lsb_release -cs) stable" | sudo tee /etc/apt/sources.list.d/docker.list > /dev/null
```

---

### **Step 5: Install Docker Engine**
1. Update the package index:
   ```bash
   sudo apt update
   ```

2. Install Docker Engine, Docker CLI, and Containerd:
   ```bash
   sudo apt install -y docker-ce docker-ce-cli containerd.io
   ```

---

### **Step 6: Verify Docker Installation**
1. Check the Docker version to confirm the installation:
   ```bash
   docker --version
   ```

2. Start the Docker service and enable it to start on boot:
   ```bash
   sudo systemctl start docker
   sudo systemctl enable docker
   ```

3. Verify Docker is running:
   ```bash
   sudo systemctl status docker
   ```

---

### **Step 7: Run a Test Container**
Run a simple Docker container to verify everything is working:
```bash
sudo docker run hello-world
```
If Docker is installed correctly, you'll see a message confirming the installation.

---

### **Step 8: Manage Docker as a Non-Root User (Optional)**
By default, Docker requires `sudo` to run commands. To use Docker without `sudo`, add your user to the `docker` group:
```bash
sudo usermod -aG docker $USER
```
After running this command, log out and log back in for the changes to take effect.

---

### **Step 9: Install Docker Compose (Optional)**
Docker Compose is a tool for defining and running multi-container Docker applications. To install it:
1. Download the latest version of Docker Compose:
   ```bash
   sudo curl -L "https://github.com/docker/compose/releases/download/$(curl -s https://api.github.com/repos/docker/compose/releases/latest | grep -Po '"tag_name": "\K.*\d')" -o /usr/local/bin/docker-compose
   ```

2. Make the binary executable:
   ```bash
   sudo chmod +x /usr/local/bin/docker-compose
   ```

3. Verify the installation:
   ```bash
   docker-compose --version
   ```

---

### **Step 10: Configure Docker to Start on Boot (Optional)**
If you want Docker to start automatically when your system boots, enable the service:
```bash
sudo systemctl enable docker
```

---

### **Troubleshooting**
- **Permission Denied Errors**: If you encounter permission issues, ensure your user is in the `docker` group and restart your session.
- **Docker Service Not Running**: If Docker fails to start, check the logs with:
  ```bash
  sudo journalctl -u docker.service
  ```

---

By following these steps, you'll have Docker installed and ready to use on your Ubuntu system. Let me know if you need further assistance!

### Reverse Footnote

[^ref1]: [Voir ce liensur youtube](https://www.youtube.com/watch?v=WoHffLF0jfU)
[^ref2]: [source in github docker machine](https://github.com/docker/machine/releases/tag/v0.16.2)
[^ref3]: [source in github docker compose](https://docs.docker.com/compose/install/)
[^ref4]: [source article](https://medium.com/@leonardo.lp2/installing-docker-on-deepin-15-9-2312f28f09b7)

 
## Deepin

Installing Docker and Docker Compose on **Deepin** (a Debian-based distribution) requires a few adjustments compared to Ubuntu. Here’s a step-by-step guide to ensure a correct installation:

---

### **1. Install Docker on Deepin**
#### **Step 1: Uninstall Old Docker Versions (if any)**
```bash
sudo apt remove docker docker-engine docker.io containerd runc
sudo rm -rf /var/lib/docker
```

#### **Step 2: Install Dependencies**
```bash
sudo apt update
sudo apt install -y ca-certificates curl gnupg lsb-release
```

#### **Step 3: Add Docker’s Official GPG Key**
```bash
curl -fsSL https://download.docker.com/linux/debian/gpg | sudo gpg --dearmor -o /usr/share/keyrings/docker-archive-keyring.gpg
```

#### **Step 4: Add Docker Repository**
Deepin is based on Debian, so use the **Debian repository** (replace `bullseye` with your Debian codename if needed):
```bash
echo "deb [arch=$(dpkg --print-architecture) signed-by=/usr/share/keyrings/docker-archive-keyring.gpg] https://download.docker.com/linux/debian bullseye stable" | sudo tee /etc/apt/sources.list.d/docker.list > /dev/null
```

#### **Step 5: Install Docker Engine**
```bash
sudo apt update
sudo apt install -y docker-ce docker-ce-cli containerd.io
```

#### **Step 6: Start and Enable Docker**
```bash
sudo systemctl start docker
sudo systemctl enable docker
```

#### **Step 7: Verify Installation**
```bash
sudo docker run hello-world
```
If you see "Hello from Docker!", it’s working!

#### **Step 8: Add User to Docker Group (Optional)**
To run Docker without `sudo`:
```bash
sudo usermod -aG docker $USER
newgrp docker  # Apply group changes without logout
```

---

### **2. Install Docker Compose on Deepin**
#### **Option 1: Install Docker Compose v2 (Recommended)**
Docker Compose v2 is included with Docker Engine for newer versions. Check if it’s already installed:
```bash
docker compose version
```
If not, install it manually:
```bash
sudo apt install -y docker-compose-plugin
```

#### **Option 2: Install Docker Compose v1 (Legacy)**
```bash
sudo curl -L "https://github.com/docker/compose/releases/download/1.29.2/docker-compose-$(uname -s)-$(uname -m)" -o /usr/local/bin/docker-compose
sudo chmod +x /usr/local/bin/docker-compose
```
Verify:
```bash
docker-compose --version
```

---

### **3. Fix Common Issues on Deepin**
#### **Issue 1: GPG Key Errors**
If you get GPG errors, manually download the key:
```bash
curl -fsSL https://download.docker.com/linux/debian/gpg | sudo apt-key add -
```

#### **Issue 2: Repository Mismatch**
Deepin may not recognize Debian’s `bullseye`. Check your Debian base:
```bash
lsb_release -a
```
Replace `bullseye` in the repository URL with your Debian codename (e.g., `buster`).

#### **Issue 3: Dependency Conflicts**
If `apt` complains about conflicts, use:
```bash
sudo apt --fix-broken install
```

---

### **4. Test Docker Compose**
Create a `docker-compose.yml` file:
```yaml
version: '3'
services:
  web:
    image: nginx
    ports:
      - "8080:80"
```
Run it:
```bash
docker compose up -d
```
Access NGINX at `http://localhost:8080`.

---

### **Summary**
1. **Docker**: Installed via Debian’s repository.
2. **Docker Compose**: Use the plugin (`docker compose`) or legacy (`docker-compose`).
3. **Deepin-Specific Fixes**: Adjust repositories for Debian base.

After this, you’ll have a fully functional Docker and Docker Compose setup on Deepin! 🐳🚀  

### Reverse Footnote

[^ref1]: [Voir ce liensur youtube](https://www.youtube.com/watch?v=WoHffLF0jfU)