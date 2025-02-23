---
title: How to remain Ubuntu Server active and not enter sleep or suspend mode (DS)
author:
  name: 
  link: 
date: 2025-02-17 19:14:00 -0400
categories: [Privacy]
tags: [docker, gogs, git, mysql]
render_with_liquid: false
---

To ensure your **Ubuntu Server** does not sleep or suspend, you need to configure the system's power management settings. Servers are typically designed to run continuously, but sometimes power-saving features can interfere. Here's how to disable sleep and suspend on Ubuntu Server:

---

### **1. Check Current Power Management Settings**
First, check if any power management services are running:
```bash
systemctl status sleep.target suspend.target hibernate.target hybrid-sleep.target
```
If any of these services are active, you’ll need to disable them.

---

### **2. Disable Sleep and Suspend Services**
To prevent the system from sleeping or suspending, mask the relevant systemd targets:
```bash
sudo systemctl mask sleep.target suspend.target hibernate.target hybrid-sleep.target
```
This will prevent these services from being activated.

---

### **3. Configure Logind Settings**
The `logind` service manages power settings. Edit its configuration file to disable sleep and suspend:
1. Open the `logind.conf` file:
   ```bash
   sudo nano /etc/systemd/logind.conf
   ```
2. Find or add the following lines:
   ```ini
   HandleSuspendKey=ignore
   HandleHibernateKey=ignore
   HandleLidSwitch=ignore
   HandleLidSwitchDocked=ignore
   IdleAction=ignore
   ```
3. Save the file and exit (`Ctrl+O`, `Enter`, `Ctrl+X`).

4. Restart the `logind` service to apply changes:
   ```bash
   sudo systemctl restart systemd-logind
   ```

---

### **4. Disable Automatic Sleep (if using a desktop environment)**
If your Ubuntu Server has a desktop environment installed (e.g., GNOME), you may need to adjust additional settings:
1. Open **Settings** > **Power**.
2. Set **Blank Screen** and **Automatic Suspend** to **Never**.

---

### **5. Prevent Hard Disk Sleep**
If your server uses hard drives, ensure they don’t go to sleep:
1. Install `hdparm`:
   ```bash
   sudo apt install hdparm
   ```
2. Check the current power management settings for your disk:
   ```bash
   sudo hdparm -B /dev/sda
   ```
   Replace `/dev/sda` with your disk identifier.
3. Disable power management for the disk:
   ```bash
   sudo hdparm -B 255 /dev/sda
   ```

---

### **6. Disable ACPI Sleep States (Advanced)**
If the system still suspends, you may need to disable ACPI sleep states in the kernel:
1. Edit the GRUB configuration:
   ```bash
   sudo nano /etc/default/grub
   ```
2. Find the `GRUB_CMDLINE_LINUX_DEFAULT` line and add `acpi=off` or `acpi_sleep=nonvs`:
   ```bash
   GRUB_CMDLINE_LINUX_DEFAULT="quiet splash acpi=off"
   ```
3. Update GRUB:
   ```bash
   sudo update-grub
   ```
4. Reboot the system:
   ```bash
   sudo reboot
   ```

---

### **7. Verify Settings**
After making these changes, verify that the system no longer sleeps or suspends:
- Monitor the system for a while to ensure it stays active.
- Check logs for any power-related events:
  ```bash
  journalctl -xe | grep -i suspend
  ```

---

By following these steps, your Ubuntu Server should remain active and not enter sleep or suspend mode. Let me know if you need further assistance! 🚀
