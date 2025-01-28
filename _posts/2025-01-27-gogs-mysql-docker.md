---
title: Dokerize Gogs with MySql 
author:
  name: 
  link: 
date: 2025-01-27 20:35:00 -0400
categories: [Privacy]
tags: [docker, gogs, git, mysql]
render_with_liquid: false
---

This post will guide and help you how to install Gogs with mysql database in docker.

Running **Gogs** (a self-hosted Git service) with **MySQL** in **Docker** and making it accessible via **SSH** involves several steps. Below are the **best practices** and a **step-by-step guide** to set this up securely and efficiently.

---

### **1. Prerequisites**
- **Docker** and **Docker Compose** installed on your server.
- A domain name (optional but recommended for HTTPS).
- Basic knowledge of Docker, MySQL, and SSH.

---

### **2. Directory Structure**
Create a directory structure for your Gogs setup:
```
gogs/
├── docker-compose.yml
├── gogs/
│   ├── data/
│   └── custom/conf/
└── mysql/
    └── data/
```

---

### **3. Docker Compose Configuration**
Create a `docker-compose.yml` file to define the Gogs and MySQL services.

```yaml
version: '3.8'

services:
  mysql:
    image: mysql:5.7
    container_name: gogs_mysql
    environment:
      MYSQL_ROOT_PASSWORD: your_root_password
      MYSQL_DATABASE: gogs
      MYSQL_USER: gogs
      MYSQL_PASSWORD: your_gogs_password
    volumes:
      - ./mysql/data:/var/lib/mysql
    networks:
      - gogs_network

  gogs:
    image: gogs/gogs
    container_name: gogs
    depends_on:
      - mysql
    ports:
      - "3000:3000"  # HTTP port
      - "2222:22"    # SSH port
    volumes:
      - ./gogs/data:/data
      - ./gogs/custom/conf:/data/gogs/conf
    environment:
      - DB_TYPE=mysql
      - DB_HOST=mysql:3306
      - DB_NAME=gogs
      - DB_USER=gogs
      - DB_PASSWD=your_gogs_password
    networks:
      - gogs_network

networks:
  gogs_network:
    driver: bridge
```

---

### **4. Configure Gogs**
1. Start the containers:
   ```bash
   docker-compose up -d
   ```
2. Access Gogs via `http://your-server-ip:3000` and complete the initial setup.
3. During setup:
   - Set **Database Type** to **MySQL**.
   - Use the following database details:
     - **Host**: `mysql:3306`
     - **User**: `gogs`
     - **Password**: `your_gogs_password`
     - **Database Name**: `gogs`
   - Configure the **Application URL** (e.g., `http://your-domain.com:3000`).
   - Set up the **SSH Server Port** to `2222`.

---

### **5. Enable SSH Access**
To make Gogs accessible via SSH:
1. **Configure SSH in Gogs:**
   - Go to **Admin Panel** > **Configuration** > **SSH**.
   - Set **SSH Port** to `2222`.
   - Set **Domain** to your server's domain or IP address.

2. **Update Docker Compose for SSH:**
   - Ensure the `gogs` service in `docker-compose.yml` maps port `2222` to the host.

3. **Test SSH Access:**
   - Clone a repository using SSH:
     ```bash
     git clone ssh://git@your-domain.com:2222/username/repo.git
     ```

---

### **6. Secure Your Setup**
1. **Use HTTPS:**
   - Use a reverse proxy (e.g., Nginx) to serve Gogs over HTTPS.
   - Obtain an SSL certificate using **Let's Encrypt**.

2. **Firewall Rules:**
   - Allow only necessary ports (e.g., 3000 for HTTP, 2222 for SSH, 443 for HTTPS).
   - Block all other ports.

3. **Backup Data:**
   - Regularly back up the `gogs/data` and `mysql/data` directories.

4. **Update Regularly:**
   - Keep Docker images and containers updated to the latest versions.

---

### **7. Example Nginx Reverse Proxy Configuration**
If you want to use HTTPS, set up Nginx as a reverse proxy:

1. Install Nginx and Certbot:
   ```bash
   sudo apt install nginx certbot python3-certbot-nginx
   ```

2. Create an Nginx configuration file for Gogs:
   ```nginx
   server {
       listen 80;
       server_name your-domain.com;

       location / {
           proxy_pass http://localhost:3000;
           proxy_set_header Host $host;
           proxy_set_header X-Real-IP $remote_addr;
           proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
           proxy_set_header X-Forwarded-Proto $scheme;
       }
   }
   ```

3. Obtain an SSL certificate:
   ```bash
   sudo certbot --nginx -d your-domain.com
   ```

4. Reload Nginx:
   ```bash
   sudo systemctl reload nginx
   ```

---

### **8. Backup and Restore**
1. **Backup:**
   - Backup the `gogs/data` and `mysql/data` directories:
     ```bash
     tar -czvf gogs_backup.tar.gz ./gogs/data ./mysql/data
     ```

2. **Restore:**
   - Extract the backup and restart the containers:
     ```bash
     tar -xzvf gogs_backup.tar.gz
     docker-compose up -d
     ```

---

### **9. Monitoring and Logs**
1. **View Logs:**
   - Check Gogs logs:
     ```bash
     docker logs gogs
     ```
   - Check MySQL logs:
     ```bash
     docker logs gogs_mysql
     ```

2. **Monitor Performance:**
   - Use tools like **Prometheus** and **Grafana** to monitor Gogs and MySQL performance.

---

### **10. Troubleshooting**
- **Port Conflicts:**
  - Ensure ports `3000` and `2222` are not in use.
- **Database Issues:**
  - Check MySQL logs for errors.
  - Verify database credentials in the Gogs configuration.
- **SSH Issues:**
  - Ensure the SSH port is open in your firewall.
  - Verify the SSH configuration in Gogs.

---

### **Conclusion**
By following these best practices, you can run **Gogs** with **MySQL** in **Docker** and make it accessible via **SSH** securely and efficiently. Let me know if you need further assistance! 🚀

## Learn More

For more knowledge about Jekyll posts, visit the [Jekyll Docs: Posts](https://jekyllrb.com/docs/posts/).
