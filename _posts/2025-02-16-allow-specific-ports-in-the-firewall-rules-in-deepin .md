---
title: Allow Specific Ports In The Firewall Rules  
author:
  name: 
  link: 
date: 2025-02-16 13:35:00 -0400
categories: [Security]
tags: [ufw, firewall]
render_with_liquid: false
---

To allow specific ports in the firewall rules on Deepin OS, you can use **UFW (Uncomplicated Firewall)**, which is a user-friendly tool for managing firewall rules. Here's how to allow ports step by step:

---

### **1. Install UFW (if not already installed)**
UFW is usually pre-installed on Deepin, but if it’s not, you can install it:
```bash
sudo apt update
sudo apt install ufw
```

---

### **2. Check UFW Status**
Before making changes, check the status of UFW to ensure it’s active and see the current rules:
```bash
sudo ufw status
```
If UFW is inactive, you’ll need to enable it.

---

### **3. Enable UFW**
If UFW is not already enabled, activate it:
```bash
sudo ufw enable
```
This will start the firewall and apply default rules.

---

### **4. Allow Specific Ports**
To allow a specific port, use the following command:
```bash
sudo ufw allow <port_number>
```
For example, to allow port `80` (HTTP):
```bash
sudo ufw allow 80
```

---

### **5. Allow Ports with Specific Protocols**
If you want to allow a port for a specific protocol (TCP or UDP), specify it:
```bash
sudo ufw allow <port_number>/<protocol>
```
For example, to allow port `443` for HTTPS (TCP):
```bash
sudo ufw allow 443/tcp
```

---

### **6. Allow a Range of Ports**
To allow a range of ports, use:
```bash
sudo ufw allow <start_port>:<end_port>/<protocol>
```
For example, to allow ports `8000` to `9000` for TCP:
```bash
sudo ufw allow 8000:9000/tcp
```

---

### **7. Allow Services by Name**
UFW includes predefined rules for common services. You can allow a service by its name instead of specifying the port:
```bash
sudo ufw allow <service_name>
```
For example, to allow SSH:
```bash
sudo ufw allow ssh
```
To see a list of predefined services, check:
```bash
less /etc/services
```

---

### **8. Verify the Rules**
After adding rules, verify them by checking the UFW status:
```bash
sudo ufw status
```
This will display a list of all active rules.

---

### **9. Delete a Rule (if needed)**
If you need to remove a rule, use:
```bash
sudo ufw delete allow <port_number>
```
For example, to remove the rule allowing port `80`:
```bash
sudo ufw delete allow 80
```

---

### **10. Disable UFW (if needed)**
If you need to temporarily disable UFW, use:
```bash
sudo ufw disable
```

---

### **11. Reset UFW (if needed)**
To reset UFW to its default settings (removing all rules), use:
```bash
sudo ufw reset
```

---

### **Example Scenario**
If you want to allow:
- Port `22` for SSH,
- Port `80` for HTTP,
- Port `443` for HTTPS,
- Port `3000` for a custom app,

Run the following commands:
```bash
sudo ufw allow ssh
sudo ufw allow 80/tcp
sudo ufw allow 443/tcp
sudo ufw allow 3000/tcp
```

---

By following these steps, you can easily manage firewall rules on Deepin OS using UFW.
Always ensure you only allow necessary ports to minimize security risks.