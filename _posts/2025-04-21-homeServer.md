---
title: Set up and configure home server
author:
  name: 
  link: 
date: 2025-04-01 07:35:00 -0400
categories: [DEV, Linux]
tags: [home-server, dell3020, lenovoM900]
render_with_liquid: false
---
## Lenovo ThinkCentre M900 Tiny

The Lenovo ThinkCentre M900 Tiny is a solid candidate for a compact Ubuntu home server, and its 8GB RAM can be upgraded to boost performance. Below, I’ll address its suitability as an Ubuntu home server, RAM upgrade options, and other considerations based on available information.

### Suitability as an Ubuntu Home Server
The ThinkCentre M900 Tiny is a micro form factor PC with a 6th-generation Intel Core processor (typically an i5-6500T or similar, 4 cores/4 threads, 35W TDP), making it power-efficient and capable for home server tasks. Here’s why it’s a good fit for an Ubuntu-based home server:

1. **Hardware Compatibility with Ubuntu**:
   - The M900 Tiny is certified for Ubuntu, with Lenovo listing support for Ubuntu 14.04, and community reports confirm compatibility with newer versions like 18.04 and 22.04.[](https://ubuntuforums.org/showthread.php?t=2429683)[](https://www.reddit.com/r/homelab/comments/1b0dwpr/need_help_install_ubuntu_on_lenovo_thinkcentre/)
   - Users have successfully run Ubuntu Server or Desktop on the M900 Tiny for tasks like media servers (Plex, Jellyfin), NAS (OpenMediaVault), Docker containers (Sonarr, Radarr, Grafana), and virtualization (Proxmox, ESXi).[](https://www.reddit.com/r/homelab/comments/13h7btx/my_current_homeserver_setup_lenovo_thinkcentre/)[](https://www.reddit.com/r/HomeServer/comments/10abyme/lenovo_thinkcentre_minipc_for_hosting_plex_and/)
   - The integrated Intel HD Graphics 530 and Skylake architecture handle lightweight server tasks well, and the CPU supports hardware transcoding (Intel Quick Sync) for media streaming, though 6th-gen Quick Sync is slightly less efficient than 7th-gen or later.[](https://www.reddit.com/r/HomeServer/comments/10abyme/lenovo_thinkcentre_minipc_for_hosting_plex_and/)

2. **Compact and Power-Efficient**:
   - At ~1.1L in size (179 x 182.9 x 34.5mm), it’s ideal for space-constrained environments.[](https://www.servethehome.com/lenovo-thinkcentre-m900-tiny-project-tinyminimicro-guide/)
   - The 35W TDP CPU and efficient design result in low power consumption, with users reporting 18-30W under typical server loads, making it cost-effective for 24/7 operation.[](https://www.reddit.com/r/homelab/comments/13h7btx/my_current_homeserver_setup_lenovo_thinkcentre/)[](https://forums.homeseer.com/forum/homeseer-products-services/general-discussion-area/how-to-s/1337091-lenovo-m93p-tiny-homeseer-computer)

3. **Storage and Connectivity**:
   - It supports one M.2 NVMe SSD (e.g., 256GB-1TB) and one 2.5” SATA drive (HDD or SSD), sufficient for a NAS or media server. Users have added 4TB drives for storage.[](https://www.reddit.com/r/homelab/comments/13h7btx/my_current_homeserver_setup_lenovo_thinkcentre/)[](https://www.hardware-corner.net/desktop-models/Lenovo-ThinkCentre-M900-Tiny/)
   - Six USB 3.0 ports, DisplayPort, and an RJ-45 Ethernet port provide ample connectivity for external drives or peripherals.[](https://www.hardware-corner.net/desktop-models/Lenovo-ThinkCentre-M900-Tiny/)
   - An optional M.2 Wi-Fi/Bluetooth card can be added for wireless connectivity, though Ethernet is preferred for server stability.[](https://www.hardware-corner.net/desktop-models/Lenovo-ThinkCentre-M900-Tiny/)

4. **Current Windows Setup**:
   - The M900 Tiny likely ships with Windows 10 Pro (or a renewed license). Switching to Ubuntu is straightforward, as Ubuntu can be installed on systems with no pre-installed OS or over Windows. You can dual-boot or replace Windows entirely.[](https://www.servethehome.com/lenovo-thinkcentre-m900-tiny-project-tinyminimicro-guide/)[](https://www.reddit.com/r/homelab/comments/1b0dwpr/need_help_install_ubuntu_on_lenovo_thinkcentre/)
   - Ubuntu Server is lightweight and doesn’t require a GUI, maximizing resources for server tasks. Ensure you back up data before installing Ubuntu.

5. **Use Case Fit**:
   - **Media Server**: Suitable for Plex or Jellyfin with hardware transcoding, supporting 2-3 1080p streams or 1-2 4K streams with optimization (avoiding transcoding is ideal).[](https://www.reddit.com/r/HomeServer/comments/jd2n43/using_a_lenovo_thinkcentre_mini_for_a_nas_home/)[](https://www.reddit.com/r/HomeServer/comments/10abyme/lenovo_thinkcentre_minipc_for_hosting_plex_and/)
   - **NAS**: OpenMediaVault or TrueNAS Scale work well, with users running 4TB drives for SMB shares.[](https://www.reddit.com/r/homelab/comments/13h7btx/my_current_homeserver_setup_lenovo_thinkcentre/)
   - **Docker/Containers**: Portainer and Docker run smoothly for lightweight services like Sonarr, Radarr, or Immich.[](https://www.reddit.com/r/homelab/comments/13h7btx/my_current_homeserver_setup_lenovo_thinkcentre/)
   - **Virtualization**: Proxmox or ESXi can run with 16GB+ RAM, though the 4-core/4-thread CPU limits heavy VM workloads.[](https://www.reddit.com/r/homelab/comments/13h7btx/my_current_homeserver_setup_lenovo_thinkcentre/)[](https://www.reddit.com/r/HomeServer/comments/setiyf/thinkcentre_m900_tiny_max_supported_ram_capacity/)
   - **Game Servers**: Minecraft or similar servers are viable with 16GB RAM (e.g., 4GB per server for 4 players).[](https://www.reddit.com/r/HomeServer/comments/10abyme/lenovo_thinkcentre_minipc_for_hosting_plex_and/)

**Potential Issues**:
   - **Fan Noise**: Some users report loud fan noise under load, especially with older BIOS versions. Updating the BIOS (July 2022 version) or configuring fan control in Ubuntu (using `lm_sensors` and `fancontrol`) can mitigate this.[](https://bmannconsulting.com/notes/lenovo-m900-tiny-setup/)
   - **NVMe Issues**: Adding an NVMe drive may cause boot issues (fans at 100%, no video output) if the drive or firmware is incompatible. Updating the BIOS and ensuring drive compatibility (e.g., Samsung 950 Pro) helps.[](https://www.servethehome.com/lenovo-thinkcentre-m900-tiny-project-tinyminimicro-guide/2/)
   - **Windows 11 Incompatibility**: The 6th-gen CPU doesn’t meet Windows 11 requirements, but this is irrelevant for Ubuntu.[](https://www.hardware-corner.net/desktop-models/Lenovo-ThinkCentre-M900-Tiny/)

### RAM Upgrade Options
The M900 Tiny’s 8GB RAM (likely 1x8GB or 2x4GB DDR4 SODIMM) is sufficient for basic server tasks but limiting for virtualization, game servers, or multiple Docker containers. Upgrading is straightforward and recommended.

1. **Maximum RAM Capacity**:
   - The M900 Tiny has **two SO-DIMM slots** and officially supports up to **32GB** (2x16GB) DDR4-2133 per Lenovo’s specs.[](https://www.hardware-corner.net/desktop-models/Lenovo-ThinkCentre-M900-Tiny/)
   - Community reports confirm **64GB** (2x32GB) works on some units, depending on the motherboard, BIOS, and memory brand (e.g., Crucial, TeamGroup). Success with 64GB requires matched 2x32GB kits (e.g., TeamGroup T-Create DDR4 SODIMM 64GB Kit, 3200MHz, CL22) and BIOS update to FWKTBFA (06/23/2022).[](https://www.reddit.com/r/HomeServer/comments/setiyf/thinkcentre_m900_tiny_max_supported_ram_capacity/)
   - For reliability, **32GB** is safer and widely supported without compatibility concerns.

2. **Upgrade Recommendations**:
   - **For Basic Server Tasks** (Plex, NAS, lightweight Docker): Upgrade to **16GB** (2x8GB) DDR4-2133 or DDR4-2400. This is cost-effective (~$25-50) and sufficient for most home servers.[](https://www.servethehome.com/lenovo-thinkcentre-m900-tiny-project-tinyminimicro-guide/)[](https://www.amazon.com/OFFTEK-Replacement-IBM-Lenovo-ThinkCentre-DDR4-19200/dp/B07G5KTWSG)
   - **For Virtualization/Game Servers** (Proxmox, ESXi, Minecraft): Upgrade to **32GB** (2x16GB) DDR4-2133 (~$50-80). This supports multiple VMs or game servers.[](https://www.reddit.com/r/HomeServer/comments/10abyme/lenovo_thinkcentre_minipc_for_hosting_plex_and/)
   - **For Heavy Workloads**: Attempt **64GB** (2x32GB) (~$120-150) if running memory-intensive tasks (e.g., TrueNAS Scale with ZFS, multiple VMs). Verify compatibility with your unit’s serial number on Lenovo’s support site or test with a matched kit.[](https://www.reddit.com/r/HomeServer/comments/setiyf/thinkcentre_m900_tiny_max_supported_ram_capacity/)

3. **RAM Compatibility**:
   - Use **DDR4 SODIMM, 2133MHz or 2400MHz**, non-ECC, 1.2V. Higher speeds (e.g., 3200MHz) may downclock to 2133MHz.[](https://www.hardware-corner.net/desktop-models/Lenovo-ThinkCentre-M900-Tiny/)[](https://www.reddit.com/r/HomeServer/comments/setiyf/thinkcentre_m900_tiny_max_supported_ram_capacity/)
   - Install modules in pairs for dual-channel performance (e.g., 2x8GB instead of 1x16GB).[](https://www.mrmemory.co.uk/memory-ram-upgrades/lenovo/thinkcentre/m900-tiny)
   - Compatible brands: Crucial, OFFTEK, TeamGroup, G.Skill. Check QVL or Lenovo’s support site for tested modules.[](https://www.crucial.com/compatible-upgrade-for/lenovo/thinkcentre-m900-tiny)[](https://www.amazon.com/OFFTEK-Replacement-IBM-Lenovo-ThinkCentre-DDR4-19200/dp/B07G5KTWSG)[](https://www.reddit.com/r/HomeServer/comments/setiyf/thinkcentre_m900_tiny_max_supported_ram_capacity/)
   - Example: Crucial 16GB Kit (2x8GB) DDR4-2400 (CT2K8G4SFS824A) or TeamGroup 32GB Kit (2x16GB) DDR4-2133.

4. **Installation**:
   - Remove the back screw, slide the top panel forward, and lift the 2.5” drive caddy to access the SO-DIMM slots.[](https://tweaks.com/windows/67177/building-the-ultimate-windows-10-micro-pc-part-2/)
   - No configuration is needed; the system auto-detects new RAM. Update the BIOS first to avoid compatibility issues with larger modules.[](https://www.reddit.com/r/HomeServer/comments/setiyf/thinkcentre_m900_tiny_max_supported_ram_capacity/)

### Other Upgrade Considerations
To optimize the M900 Tiny as an Ubuntu home server:
1. **Storage**:
   - Replace the existing drive with a **256GB-1TB NVMe SSD** for the OS and apps (e.g., Samsung 970 EVO, ~$50-100). Add a **2.5” SATA SSD** (e.g., 1TB, ~$60) or **HDD** (e.g., 4TB Seagate IronWolf, ~$100) for data.[](https://www.reddit.com/r/homelab/comments/13h7btx/my_current_homeserver_setup_lenovo_thinkcentre/)[](https://www.servethehome.com/lenovo-thinkcentre-m900-tiny-project-tinyminimicro-guide/)
   - External USB 3.0 drives can expand storage, but internal drives are preferred for reliability. Avoid USB for critical data due to potential failures.[](https://www.reddit.com/r/homelab/comments/13h7btx/my_current_homeserver_setup_lenovo_thinkcentre/)[](https://www.reddit.com/r/HomeServer/comments/jd2n43/using_a_lenovo_thinkcentre_mini_for_a_nas_home/)

2. **BIOS Update**:
   - Update to the latest BIOS (FWKTBFA, 06/23/2022) to fix fan noise, improve RAM compatibility, and resolve NVMe issues. On Ubuntu, use Lenovo’s bootable USB method or extract files for Linux-based updates.[](https://bmannconsulting.com/notes/lenovo-m900-tiny-setup/)[](https://www.cyberciti.biz/faq/update-lenovo-bios-from-linux-usb-stick-pen/)

3. **Cooling**:
   - Monitor temperatures with `lm_sensors` on Ubuntu. If the fan is noisy, configure fan curves or replace the stock fan with a quieter model (check compatibility).[](https://bmannconsulting.com/notes/lenovo-m900-tiny-setup/)

4. **Networking**:
   - Use the onboard Gigabit Ethernet for stable server performance. If Wi-Fi is needed, add an M.2 Wi-Fi card (e.g., Intel AX200, ~$20).[](https://www.hardware-corner.net/desktop-models/Lenovo-ThinkCentre-M900-Tiny/)
   - For advanced setups (e.g., pfSense), a USB-to-Gigabit adapter can add a second NIC.[](https://www.reddit.com/r/homelab/comments/13h7btx/my_current_homeserver_setup_lenovo_thinkcentre/)[](https://forums.homeseer.com/forum/homeseer-products-services/general-discussion-area/how-to-s/1337091-lenovo-m93p-tiny-homeseer-computer)

### Ubuntu Installation Tips
1. **Download Ubuntu Server** (22.04 LTS or 24.04 LTS) from ubuntu.com for a lightweight, CLI-based OS. Use Ubuntu Desktop if you prefer a GUI.
2. **Create a Bootable USB** with tools like Rufus (on Windows) or `dd` (on Linux).
3. **Backup Data**: Save any critical files from the Windows setup before wiping the drive.
4. **Install Ubuntu**:
   - Boot from the USB, select “Install Ubuntu Server,” and follow the prompts.
   - Choose the NVMe or SATA drive for installation. Configure a static IP for server access.
   - Install updates post-installation (`sudo apt update && sudo apt upgrade`).
5. **Drivers**: Ubuntu typically auto-detects drivers for the M900’s Intel chipset, CPU, and networking. If issues arise (e.g., Wi-Fi or audio), check Ubuntu’s HCL or install proprietary drivers via `ubuntu-drivers`.[](https://www.reddit.com/r/homelab/comments/1b0dwpr/need_help_install_ubuntu_on_lenovo_thinkcentre/)
6. **Fan Control**: If fan noise persists, install `lm_sensors` and `fancontrol` (`sudo apt install lm-sensors fancontrol`) and configure via `sensors-detect`.[](https://bmannconsulting.com/notes/lenovo-m900-tiny-setup/)
7. **Server Software**:
   - **NAS**: Install OpenMediaVault (`sudo apt install openmediavault`) or TrueNAS Scale.
   - **Media Server**: Install Plex or Jellyfin (`sudo apt install plexmediaserver` or use Docker).
   - **Docker**: Install Docker (`sudo apt install docker.io`) and Portainer for container management.
   - **Virtualization**: Install Proxmox or KVM for VMs.

### Cost Estimate for Upgrades

- **16GB RAM (2x8GB)**: $25-50
- **32GB RAM (2x16GB)**: $50-80
- **64GB RAM (2x32GB)**: $120-150 (if compatible)
- **256GB NVMe SSD**: $30-50
- **4TB 2.5” HDD**: ~$100
- **M.2 Wi-Fi Card**: ~$20 (optional)
- **Total**: $75-300, depending on upgrades

### Recommendations

- **Minimum Upgrade**: Add 8GB RAM (total 16GB, ~$25) and a 256GB NVMe SSD (~$30). Install Ubuntu Server 22.04 LTS and OpenMediaVault for a NAS or Plex for media streaming.
- **Optimal Upgrade**: Upgrade to 32GB RAM (2x16GB, ~$60), add a 512GB NVMe SSD (~$50), and a 4TB 2.5” HDD (~$100). Run Proxmox with Docker containers or a TrueNAS Scale setup.
- **Verify 64GB RAM**: If pursuing 64GB, check your model’s serial number on Lenovo’s support site (https://pcsupport.lenovo.com) and test with a compatible kit (e.g., TeamGroup 2x32GB). Update BIOS first.[](https://www.reddit.com/r/HomeServer/comments/setiyf/thinkcentre_m900_tiny_max_supported_ram_capacity/)

### Final Notes
The ThinkCentre M900 Tiny is an excellent, budget-friendly platform for an Ubuntu home server due to its compact size, low power use, and Ubuntu compatibility. Upgrading to 16-32GB RAM and adding an NVMe SSD will make it versatile for most home server tasks. For heavy virtualization or 64GB RAM, confirm compatibility with your specific unit. If you encounter issues during Ubuntu installation or upgrades (e.g., NVMe boot problems or fan noise), update the BIOS and consult community resources like Reddit’s r/homelab or ubuntuforums.org.[](https://www.reddit.com/r/homelab/comments/13h7btx/my_current_homeserver_setup_lenovo_thinkcentre/)[](https://ubuntuforums.org/showthread.php?t=2429683)

If you share your intended server tasks (e.g., Plex, NAS, VMs) or specific model details (CPU, current storage), I can tailor recommendations further. Would you like help with Ubuntu installation steps or specific software setup?

Setting up the Lenovo ThinkCentre M900 Tiny with Ubuntu Server for reliable, always-on operation as a home server involves configuring Ubuntu to prevent sleep, optimize power settings, ensure network stability, and secure remote access. Below is a step-by-step guide to configure Ubuntu Server (22.04 LTS or 24.04 LTS recommended) to run continuously without sleeping, along with other essential server settings. These instructions assume you’re starting with a fresh Ubuntu Server installation on the M900 Tiny.

---

### Step 1: Install Ubuntu Server
1. **Download Ubuntu Server**:
   - Go to [ubuntu.com/download/server](https://ubuntu.com/download/server) and download the latest LTS version (e.g., 22.04 LTS or 24.04 LTS).
2. **Create a Bootable USB**:
   - Use a tool like Rufus (on Windows) or `dd` (on Linux) to create a bootable USB drive.
   - Example for `dd` on Linux: `sudo dd if=ubuntu-22.04.3-live-server-amd64.iso of=/dev/sdX bs=4M status=progress && sync` (replace `/dev/sdX` with your USB device).
3. **Install Ubuntu**:
   - Boot the M900 Tiny from the USB (press F12 at startup to select the boot device).
   - Follow the installer prompts:
     - Choose language, keyboard, and network settings (use a static IP for servers; see Step 5).
     - Select the NVMe or SATA drive for installation.
     - Create a user account (e.g., `admin` with a strong password).
     - Enable OpenSSH during installation for remote access.
     - Skip additional software unless needed (e.g., Docker, cloud tools).
   - After installation, reboot and log in via the console or SSH.

---

### Step 2: Prevent Sleep and Suspend
Ubuntu Server is designed to stay awake by default, but certain power-saving settings may enable sleep or suspend, especially if a desktop environment is installed or if the BIOS triggers it. Here’s how to ensure the system never sleeps:

1. **Disable Sleep in Ubuntu**:
   - Check current power settings:
     ```bash
     systemctl status systemd-suspend.service
     ```
     If it’s active, disable it:
     ```bash
     sudo systemctl mask suspend.target
     sudo systemctl mask sleep.target
     sudo systemctl mask hibernate.target
     sudo systemctl mask hybrid-sleep.target
     ```
   - Edit the logind configuration to prevent sleep on idle:
     ```bash
     sudo nano /etc/systemd/logind.conf
     ```
     Set the following lines (add or modify):
     ```ini
     HandleSuspendKey=ignore
     HandleLidSwitch=ignore
     HandleLidSwitchDocked=ignore
     HandlePowerKey=poweroff
     IdleAction=ignore
     ```
     Save and restart the service:
     ```bash
     sudo systemctl restart systemd-logind
     ```

2. **Disable Sleep in GRUB**:
   - Some systems may enter sleep due to kernel parameters. Edit GRUB:
     ```bash
     sudo nano /etc/default/grub
     ```
     Modify the `GRUB_CMDLINE_LINUX_DEFAULT` line to include `noacpi` or `acpi=off` if needed (test cautiously, as this disables ACPI features):
     ```bash
     GRUB_CMDLINE_LINUX_DEFAULT="quiet splash noacpi"
     ```
     Update GRUB:
     ```bash
     sudo update Karla update-grub
     ```
     Reboot to apply.

3. **BIOS Power Settings**:
   - Restart the M900 Tiny and enter the BIOS (press F1 at startup).
   - Navigate to **Power** or **Advanced** settings.
   - Disable options like:
     - **C-States** or **C1E** (low-power CPU states).
     - **System Idle Power Saving**.
     - **Suspend Mode** or **S3 Sleep State**.
   - Set **After Power Loss** to **Restore Last State** or **Turn On** to ensure the system restarts after power outages.
   - Save and exit (F10).

4. **Verify No Sleep**:
   - Monitor system state:
     ```bash
     systemd-analyze
     ```
   - Ensure the system stays in the `running` state. Test by leaving it idle for hours and checking if it remains accessible via SSH or console.

---

### Step 3: Optimize Power and Performance
To balance performance and efficiency on the M900 Tiny:

1. **Set CPU Governor**:
   - Use the `performance` governor for maximum CPU performance or `ondemand` for efficiency:
     ```bash
     sudo apt install cpufrequtils
     sudo cpufreq-set -g performance
     ```
     Verify:
     ```bash
     cpufreq-info
     ```

2. **Monitor Temperatures and Fan**:
   - Install `lm-sensors` to monitor CPU and system temperatures:
     ```bash
     sudo apt install lm-sensors
     sudo sensors-detect
     sensors
     ```
   - If the fan is noisy (common on M900 Tiny), configure fan control:
     ```bash
     sudo apt install fancontrol
     sudo pwmconfig
     ```
     Follow prompts to set fan thresholds. If `pwmconfig` fails, update the BIOS (see Step 8).

3. **Disable Unnecessary Services**:
   - Stop services that may consume resources (e.g., GUI if installed):
     ```bash
     sudo systemctl disable bluetooth
     sudo systemctl disable cups
     ```
   - Check running services:
     ```bash
     systemctl list-units --type=service --state=running
     ```

---

### Step 4: Configure Networking
A stable network is critical for a server. Configure a static IP and ensure constant connectivity:

1. **Set a Static IP**:
   - Edit the Netplan configuration:
     ```bash
     sudo nano /etc/netplan/01-netcfg.yaml
     ```
     Example configuration (adjust for your network):
     ```yaml
     network:
       version: 2
       ethernets:
         eno1:
           dhcp4: no
           addresses:
             - 192.168.1.100/24
           gateway4: 192.168.1.1
           nameservers:
             addresses: [8.8.8.8, 8.8.4.4]
     ```
     Apply changes:
     ```bash
     sudo netplan apply
     ```
     Verify:
     ```bash
     ip addr
     ```

2. **Enable Wake-on-LAN (Optional)**:
   - If you want to wake the server remotely:
     ```bash
     sudo apt install ethtool
     sudo ethtool -s eno1 wol g
     ```
     Enable in BIOS under **Power** > **Wake on LAN** > **Enabled**.

3. **Test Connectivity**:
   - Ping your router or an external site:
     ```bash
     ping 8.8.8.8
     ```
   - Ensure the server remains reachable after hours of operation.

---

### Step 5: Secure Remote Access
For reliable SSH access and security:

1. **Update SSH Configuration**:
   - Edit SSH settings:
     ```bash
     sudo nano /etc/ssh/sshd_config
     ```
     Modify or add:
     ```ini
     Port 22  # Change to a non-standard port (e.g., 2222) for security
     PermitRootLogin no
     PasswordAuthentication no  # Use SSH keys instead
     ```
   - Generate an SSH key pair on your client machine:
     ```bash
     ssh-keygen -t rsa -b 4096
     ```
     Copy the public key to the server:
     ```bash
     ssh-copy-id admin@192.168.1.100
     ```
     Restart SSH:
     ```bash
     sudo systemctl restart ssh
     ```

2. **Install Fail2Ban**:
   - Protect against brute-force attacks:
     ```bash
     sudo apt install fail2ban
     sudo systemctl enable fail2ban
     ```
     Default settings are sufficient for SSH protection.

3. **Firewall Setup**:
   - Install and configure UFW:
     ```bash
     sudo apt install ufw
     sudo ufw allow 22  # Or your custom SSH port
     sudo ufw enable
     sudo ufw status
     ```
   - Add ports for specific services (e.g., 80 for HTTP, 32400 for Plex).

---

### Step 6: Install Essential Server Software
Based on common home server use cases, install software tailored to your needs:

1. **NAS (OpenMediaVault)**:
   ```bash
   sudo wget -O - https://omv-extras.org/install | bash
   ```
   Access the web interface at `http://192.168.1.100`.

2. **Media Server (Plex)**:
   ```bash
   sudo apt install apt-transport-https
   wget -q https://downloads.plex.tv/plex-keys/PlexSign.key -O - | sudo apt-key add -
   echo deb https://downloads.plex.tv/repo/deb public main | sudo tee /etc/apt/sources.list.d/plexmediaserver.list
   sudo apt update
   sudo apt install plexmediaserver
   ```
   Access at `http://192.168.1.100:32400/web`.

3. **Docker (For Containers)**:
   ```bash
   sudo apt install docker.io
   sudo systemctl enable docker
   sudo usermod -aG docker admin
   ```
   Install Portainer for management:
   ```bash
   docker run -d -p 9000:9000 --name portainer --restart always -v /var/run/docker.sock:/var/run/docker.sock portainer/portainer-ce
   ```
   Access at `http://192.168.1.100:9000`.

4. **Virtualization (Proxmox)**:
   - If using Proxmox instead of Ubuntu Server, download the ISO from [proxmox.com](https://www.proxmox.com) and install it directly. It includes a web interface for VMs and containers.

---

### Step 7: Automate Updates and Backups
1. **Enable Automatic Updates**:
   ```bash
   sudo apt install unattended-upgrades
   sudo dpkg-reconfigure -plow unattended-upgrades
   ```
   Choose “Yes” to enable automatic security updates.

2. **Set Up Backups**:
   - Use `rsync` for local backups to an external drive:
     ```bash
     sudo rsync -avh --progress /home/admin /mnt/backup
     ```
   - For offsite backups, use `rclone` with Google Drive or similar:
     ```bash
     sudo apt install rclone
     rclone config
     ```

3. **Monitor Disk Health**:
   - Install `smartmontools`:
     ```bash
     sudo apt install smartmontools
     sudo smartctl -a /dev/sda
     ```

---

### Step 8: Update BIOS (Optional but Recommended)

To address potential fan noise, NVMe compatibility, or power issues on the M900 Tiny:
1. **Check Current BIOS Version**:
   - On Ubuntu:
     ```bash
     sudo dmidecode -s bios-version
     ```
2. **Download Latest BIOS**:
   - Visit [Lenovo’s support site](https://pcsupport.lenovo.com), enter your M900 Tiny’s serial number, and download the latest BIOS (e.g., FWKTBFA, 06/23/2022).
3. **Update BIOS**:
   - Create a bootable USB with Lenovo’s BIOS update tool (Windows-based) or extract the `.rom` file and use `flashrom` on Ubuntu (advanced).
   - Follow Lenovo’s instructions to flash the BIOS from a USB or Windows environment.

---

### Step 9: Monitor and Test
1. **Install Monitoring Tools**:
   - Use `htop` for resource usage:
     ```bash
     sudo apt install htop
     htop
     ```
   - Install `cockpit` for a web-based dashboard:
     ```bash
     sudo apt install cockpit
     ```
     Access at `https://192.168.1.100:9090`.

2. **Test Server Uptime**:
   - Check uptime:
     ```bash
     uptime
     ```
   - Leave the server running for 24-48 hours and verify it doesn’t sleep, lose network connectivity, or overheat.

3. **Log Monitoring**:
   - Check system logs for errors:
     ```bash
     sudo journalctl -xe
     ```

---

### Additional Tips
- **Fan Noise**: If the fan remains loud after BIOS update, replace it with a compatible Noctua fan (check M900 Tiny’s fan size, ~40mm) or adjust fan curves manually.
- **Power Outages**: Connect the M900 Tiny to a UPS (e.g., APC 600VA, ~$50) to prevent data loss during outages.
- **Specific Use Cases**: If you have a specific server goal (e.g., Plex, Minecraft, NAS), let me know, and I can provide tailored software setup instructions.
- **Community Resources**: Check [r/homelab](https://reddit.com/r/homelab) or [ubuntuforums.org](https://ubuntuforums.org) for M900 Tiny-specific tips.

---

### Verification
After configuration:
- SSH into the server after 24 hours to confirm it’s awake (`ssh admin@192.168.1.100`).
- Check temperatures (`sensors`) and fan speed to ensure the system isn’t overheating.
- Access any installed services (e.g., Plex, OpenMediaVault) to verify functionality.


This setup ensures your M900 Tiny runs Ubuntu Server reliably, stays awake, and is optimized for home server tasks. If you encounter issues (e.g., fan noise, network drops) or need help with specific software (e.g., Docker, Proxmox), let me know, and I’ll provide targeted assistance. Would you like detailed instructions for a specific server application or troubleshooting for any hardware concerns?

----

## Lenovo ThinkCentre M900 Tiny

The Dell OptiPlex 3020 (assuming a micro, SFF, or mini-tower model with a 4th-generation Intel Core processor) can be configured as an Ubuntu home server similarly to the Lenovo ThinkCentre M900 Tiny, but there are hardware differences that affect compatibility, setup, and performance. Below, I’ll evaluate the Dell 3020’s suitability as an Ubuntu server, confirm whether the same configuration and setup from the M900 Tiny can be applied, and highlight any adjustments needed to prevent sleep, optimize settings, and address hardware-specific considerations.

---

### Suitability of Dell OptiPlex 3020 as an Ubuntu Home Server
The Dell OptiPlex 3020 is a business-class desktop from 2014, typically equipped with a 4th-generation Intel Core processor (e.g., i5-4590, 4 cores/4 threads, 3.3GHz, 84W TDP in SFF/mini-tower; or i5-4590T, 4 cores/4 threads, 2.0-3.0GHz, 35W TDP in micro). It’s a capable platform for a home server, but here’s how it compares to the M900 Tiny:

1. **Hardware Compatibility with Ubuntu**:
   - The OptiPlex 3020 is well-supported by Ubuntu, with Dell providing Ubuntu certification for similar models (e.g., OptiPlex 3010, 3020) on versions like 14.04 LTS. Community reports confirm successful installations of Ubuntu 18.04, 20.04, and 22.04 on the 3020.
   - The Intel Haswell architecture (4th-gen CPUs) and Intel Q87 chipset work seamlessly with Ubuntu’s kernel, including integrated Intel HD Graphics 4600 for lightweight tasks or hardware transcoding (Quick Sync, though less efficient than the M900’s 6th-gen Skylake).
   - Common server tasks like Plex, OpenMediaVault, Docker, or Proxmox run well, though the 3020’s older CPU and higher power draw (especially in SFF/mini-tower) make it less efficient than the M900 Tiny.

2. **Form Factor and Power Efficiency**:
   - **Micro**: Similar to the M900 Tiny (~1.2L, 182 x 178 x 36mm), with a 35W TDP CPU (e.g., i5-4590T). Power consumption is ~30-50W under load, slightly higher than the M900 Tiny (~18-30W).
   - **SFF/Mini-Tower**: Larger (SFF: ~11.4L; mini-tower: ~16.7L) with 65-84W TDP CPUs (e.g., i5-4590). Power draw is ~50-100W, less efficient for 24/7 operation.
   - The micro form factor is ideal for space-constrained setups, but all models are less compact than the M900 Tiny.

3. **Storage and Connectivity**:
   - **Micro**: One 2.5” SATA bay (HDD/SSD, up to 4TB) and one M.2 SATA slot (for SSDs, not NVMe). No NVMe support limits storage performance compared to the M900 Tiny.
   - **SFF/Mini-Tower**: Multiple 3.5”/2.5” SATA bays (e.g., 2-4 drives) and one PCIe slot for additional storage controllers (e.g., HBA for ZFS/TrueNAS). M.2 SATA support may be absent or limited.
   - Connectivity: 4-6 USB ports (2.0/3.0 mix), VGA, DisplayPort/HDMI, and Gigabit Ethernet. No native Wi-Fi, but an M.2 or PCIe Wi-Fi card can be added.
   - Storage flexibility is better in SFF/mini-tower models for NAS setups compared to the M900 Tiny’s single 2.5” bay.

4. **RAM**:
   - The 3020 (all form factors) has **two DIMM slots** (non-ECC DDR3, 1600MHz) and supports up to **16GB** (2x8GB) officially. Unlike the M900 Tiny (32-64GB DDR4), 16GB is the maximum, which may limit virtualization or memory-intensive tasks.
   - Your current 8GB (likely 1x8GB or 2x4GB) is adequate for basic tasks (Plex, NAS) but should be upgraded to 16GB for Docker or VMs.

5. **Use Case Fit**:
   - **Media Server**: Plex/Jellyfin work well with Quick Sync for 1-2 1080p streams or 1 4K stream (avoid transcoding for best performance).
   - **NAS**: OpenMediaVault or TrueNAS Core run effectively, especially in SFF/mini-tower with multiple drives.
   - **Docker**: Lightweight containers (Sonarr, Radarr) are fine with 8-16GB RAM.
   - **Virtualization**: Proxmox or ESXi are viable with 16GB RAM, but the 4-core/4-thread CPU and RAM cap limit heavy VM workloads compared to the M900 Tiny.
   - **Game Servers**: Minecraft or similar servers work with 16GB (4-6GB allocated per server).

6. **Potential Issues**:
   - **Power Consumption**: SFF/mini-tower models use more power than the M900 Tiny, increasing electricity costs for 24/7 operation.
   - **No NVMe**: Micro models lack NVMe support, limiting storage speed. SFF/mini-tower can add PCIe NVMe adapters.
   - **Fan Noise**: The 3020’s stock cooling can be noisy under load, especially in micro models. BIOS updates or fan replacements may help.
   - **BIOS Limitations**: Older BIOS versions may have issues with Linux power management or fan control. Updating is recommended.

---

### Can You Use the Same Configuration and Setup?
The Ubuntu Server setup described for the M900 Tiny (preventing sleep, optimizing power, configuring networking, securing SSH, etc.) is **largely applicable** to the Dell OptiPlex 3020, as both systems use Intel x86-64 architecture and are Linux-compatible. However, hardware differences require adjustments. Below, I’ll adapt the M900 Tiny’s setup instructions for the 3020, noting deviations and specific considerations.

---

### Step 1: Install Ubuntu Server
The installation process is identical:
1. **Download Ubuntu Server** (22.04 LTS or 24.04 LTS) from [ubuntu.com/download/server](https://ubuntu.com/download/server).
2. **Create a Bootable USB** using Rufus or `dd` (e.g., `sudo dd if=ubuntu-22.04.3-live-server-amd64.iso of=/dev/sdX bs=4M status=progress && sync`).
3. **Install**:
   - Boot from USB (press F12 on the 3020 to enter the boot menu).
   - Configure a static IP, select the SATA/M.2 drive, enable OpenSSH, and create a user (e.g., `admin`).
   - Note: If using an M.2 SATA SSD, ensure it’s detected during installation. The 3020 micro doesn’t support NVMe, so avoid NVMe drives unless using a PCIe adapter in SFF/mini-tower.

**Adjustment**: If the installer fails to detect the M.2 SATA SSD (rare), update the BIOS first (see Step 8) or use a 2.5” SATA drive.

---

### Step 2: Prevent Sleep and Suspend
The steps to disable sleep are the same, but BIOS settings differ:

1. **Disable Sleep in Ubuntu**:
   - Disable suspend services:
     ```bash
     sudo systemctl mask suspend.target sleep.target hibernate.target hybrid-sleep.target
     ```
   - Edit `/etc/systemd/logind.conf`:
     ```bash
     sudo nano /etc/systemd/logind.conf
     ```
     Set:
     ```ini
     HandleSuspendKey=ignore
     HandleLidSwitch=ignore
     HandleLidSwitchDocked=ignore
     HandlePowerKey=poweroff
     IdleAction=ignore
     ```
     Restart:
     ```bash
     sudo systemctl restart systemd-logind
     ```

2. **Disable Sleep in GRUB**:
   - Edit `/etc/default/grub`:
     ```bash
     sudo nano /etc/default/grub
     ```
     Add `noacpi` if needed (test first):
     ```bash
     GRUB_CMDLINE_LINUX_DEFAULT="quiet splash noacpi"
     ```
     Update:
     ```bash
     sudo update-grub
     ```

3. **BIOS Power Settings**:
   - Enter the BIOS (press F2 at startup).
   - Navigate to **Power Management**:
     - Disable **Deep Sleep Control** or **S3 Sleep State**.
     - Set **AC Recovery** to **Power On** or **Last State** for power outage recovery.
     - Disable **C-States Control** or **Intel SpeedStep** to prevent low-power modes.
   - Save and exit (F10).
   - **Note**: The 3020’s BIOS may have fewer power-saving options than the M900 Tiny, but ensure all sleep-related settings are off.

4. **Verify**:
   - Check system state:
     ```bash
     systemd-analyze
     ```
   - Leave idle for hours and confirm it stays accessible via SSH.

---

### Step 3: Optimize Power and Performance
1. **CPU Governor**:
   - Install and set to `performance` or `ondemand`:
     ```bash
     sudo apt install cpufrequtils
     sudo cpufreq-set -g performance
     cpufreq-info
     ```
   - The 3020’s 4th-gen CPU (e.g., i5-4590) is less efficient than the M900’s 6th-gen, so `ondemand` may save power in micro models.

2. **Monitor Temperatures and Fan**:
   - Install `lm-sensors`:
     ```bash
     sudo apt install lm-sensors
     sudo sensors-detect
     sensors
     ```
   - For fan control:
     ```bash
     sudo apt install fancontrol
     sudo pwmconfig
     ```
     **Note**: The 3020’s fan control may be limited by the BIOS. If `pwmconfig` fails, update the BIOS or replace the fan (e.g., Noctua 40mm for micro).

3. **Disable Unnecessary Services**:
   - Same as M900 Tiny:
     ```bash
     sudo systemctl disable bluetooth cups
     systemctl list-units --type=service --state=running
     ```

**Adjustment**: The 3020’s higher TDP (especially SFF/mini-tower) may cause warmer operation. Monitor temperatures and consider adding case fans in larger models.

---

### Step 4: Configure Networking
The networking setup is identical:
1. **Static IP**:
   - Edit `/etc/netplan/01-netcfg.yaml`:
     ```bash
     sudo nano /etc/netplan/01-netcfg.yaml
     ```
     Example:
     ```yaml
     network:
       version: 2
       ethernets:
         eno1:
           dhcp4: no
           addresses: [192.168.1.100/24]
           gateway4: 192.168.1.1
           nameservers:
             addresses: [8.8.8.8, 8.8.4.4]
     ```
     Apply:
     ```bash
     sudo netplan apply
     ip addr
     ```

2. **Wake-on-LAN** (Optional):
   - Enable:
     ```bash
     sudo apt install ethtool
     sudo ethtool -s eno1 wol g
     ```
   - In BIOS, enable **Wake on LAN** under **Power Management**.

**Adjustment**: The 3020’s Intel I217-LM Ethernet controller is Linux-compatible but may require firmware updates for stability. Check:
```bash
sudo dmesg | grep ethernet
```

---

### Step 5: Secure Remote Access
The SSH and security setup is the same:
1. **SSH Configuration**:
   - Edit `/etc/ssh/sshd_config`:
     ```bash
     sudo nano /etc/ssh/sshd_config
     ```
     Set:
     ```ini
     Port 2222
     PermitRootLogin no
     PasswordAuthentication no
     ```
     Copy SSH keys and restart:
     ```bash
     ssh-copy-id admin@192.168.1.100
     sudo systemctl restart ssh
     ```

2. **Fail2Ban**:
   ```bash
   sudo apt install fail2ban
   sudo systemctl enable fail2ban
   ```

3. **Firewall**:
   ```bash
   sudo apt install ufw
   sudo ufw allow 2222
   sudo ufw enable
   sudo ufw status
   ```

**Adjustment**: No changes needed; the 3020’s networking stack supports these settings identically.

---

### Step 6: Install Essential Server Software
The software installation (Plex, OpenMediaVault, Docker, etc.) is identical to the M900 Tiny:
1. **NAS (OpenMediaVault)**:
   ```bash
   sudo wget -O - https://omv-extras.org/install | bash
   ```
   **Note**: SFF/mini-tower models support more drives, making them better for NAS.

2. **Plex**:
   ```bash
   sudo apt install apt-transport-https
   wget -q https://downloads.plex.tv/plex-keys/PlexSign.key -O - | sudo apt-key add -
   echo deb https://downloads.plex.tv/repo/deb public main | sudo tee /etc/apt/sources.list.d/plexmediaserver.list
   sudo apt update
   sudo apt install plexmediaserver
   ```

3. **Docker**:
   ```bash
   sudo apt install docker.io
   sudo systemctl enable docker
   sudo usermod -aG docker admin
   docker run -d -p 9000:9000 --name portainer --restart always -v /var/run/docker.sock:/var/run/docker.sock portainer/portainer-ce
   ```

4. **Proxmox**:
   - Install Proxmox directly from its ISO if preferred over Ubuntu Server.

**Adjustment**: The 3020’s 16GB RAM cap and 4th-gen CPU limit heavy virtualization. Allocate resources carefully (e.g., 4-6GB for Plex, 2-4GB per VM).

---

### Step 7: Automate Updates and Backups
Identical to the M900 Tiny:
1. **Automatic Updates**:
   ```bash
   sudo apt install unattended-upgrades
   sudo dpkg-reconfigure -plow unattended-upgrades
   ```

2. **Backups**:
   - Local: `sudo rsync -avh --progress /home/admin /mnt/backup`
   - Offsite: Install `rclone` and configure.

3. **Disk Health**:
   ```bash
   sudo apt install smartmontools
   sudo smartctl -a /dev/sda
   ```

**Adjustment**: If using multiple drives in SFF/mini-tower, monitor each with `smartctl`.

---

### Step 8: Update BIOS
1. **Check BIOS Version**:
   ```bash
   sudo dmidecode -s bios-version
   ```
2. **Update**:
   - Visit [Dell’s support site](https://www.dell.com/support), enter your 3020’s service tag, and download the latest BIOS (e.g., A20, 2019).
   - Use Dell’s bootable USB method or Windows-based updater (before wiping Windows). On Ubuntu, use `flashrom` (advanced) if supported.
   - **Why**: Fixes fan control, Linux compatibility, and power management issues.

**Adjustment**: The 3020’s BIOS updates are critical for Linux, as early versions (e.g., A01) have poor fan control and sleep handling.

---

### Step 9: Monitor and Test
Same as M900 Tiny:
1. **Monitoring**:
   ```bash
   sudo apt install htop cockpit
   ```
   Access Cockpit at `https://192.168.1.100:9090`.

2. **Test Uptime**:
   ```bash
   uptime
   ```

3. **Logs**:
   ```bash
   sudo journalctl -xe
   ```

**Adjustment**: The 3020 may run warmer (especially SFF/mini-tower). Ensure good airflow and monitor temperatures.

---

### Hardware-Specific Adjustments for Dell 3020
1. **RAM Upgrade**:
   - Upgrade to **16GB** (2x8GB DDR3-1600, ~$30-50) for optimal performance. Compatible brands: Crucial, Kingston.
   - Unlike the M900 Tiny (32-64GB DDR4), 16GB is the maximum, so prioritize lightweight workloads.
   - Install: Open the case (micro: slide panel; SFF/mini-tower: remove side panel) and insert DIMMs in paired slots.

2. **Storage**:
   - **Micro**: Use a 2.5” SATA SSD (e.g., 1TB, ~$60) or M.2 SATA SSD (e.g., 256GB, ~$30). Add external USB drives for extra storage.
   - **SFF/Mini-Tower**: Install multiple 3.5”/2.5” drives (e.g., 4TB Seagate IronWolf, ~$100 each) for NAS.
   - **No NVMe**: For NVMe, add a PCIe-to-NVMe adapter in SFF/mini-tower (~$20).

3. **Cooling**:
   - The 3020’s stock fan can be noisy. Replace with a quieter model (e.g., Noctua 80mm for SFF) or adjust fan curves post-BIOS update.
   - Ensure ventilation, especially for SFF/mini-tower with multiple drives.

4. **Power Supply**:
   - Micro: External 65W adapter (sufficient).
   - SFF/Mini-Tower: 255W/290W PSU. Consider a higher-efficiency PSU (~$50) for 24/7 operation if adding drives.

5. **Form Factor Choice**:
   - **Micro**: Best for space and power efficiency, similar to M900 Tiny, but limited storage.
   - **SFF/Mini-Tower**: Better for NAS or high-storage setups but less efficient and larger.

---

### Key Differences and Limitations
| Feature                | Lenovo ThinkCentre M900 Tiny | Dell OptiPlex 3020 (Micro/SFF) |
|------------------------|-----------------------------|-------------------------------|
| **CPU**                | 6th-gen Intel (e.g., i5-6500T, 35W) | 4th-gen Intel (e.g., i5-4590T, 35W; i5-4590, 84W) |
| **RAM**                | 32-64GB DDR4 (2x SODIMM)   | 16GB DDR3 (2x DIMM)          |
| **Storage**            | 1x M.2 NVMe + 1x 2.5” SATA  | Micro: 1x M.2 SATA + 1x 2.5” SATA; SFF: 2-4x SATA |
| **NVMe Support**       | Yes (M.2 NVMe)              | No (SFF can use PCIe adapter) |
| **Power Consumption**  | 18-30W (micro)              | 30-50W (micro); 50-100W (SFF) |
| **Size**               | 1.1L (micro)                | 1.2L (micro); 11.4L (SFF)    |
| **Fan Noise**          | Moderate, BIOS-fixable      | Potentially louder, BIOS-dependent |

- **RAM Limitation**: The 3020’s 16GB cap restricts heavy virtualization or multi-container Docker setups compared to the M900 Tiny’s 32-64GB.
- **Storage**: The 3020 SFF/mini-tower excels for NAS with multiple drives, but the micro is storage-limited like the M900 Tiny.
- **Efficiency**: The 3020 (especially SFF) consumes more power, increasing costs for 24/7 use.
- **NVMe**: Lack of native NVMe in the 3020 reduces storage performance unless using PCIe adapters in SFF/mini-tower.

---

### Recommendations
- **Use Same Setup**: Apply the M900 Tiny’s Ubuntu Server configuration (Steps 1-9) with the adjustments noted above. The core steps (disabling sleep, static IP, SSH, software) are identical due to Linux compatibility.
- **Form Factor**:
  - **Micro**: Ideal for lightweight tasks (Plex, Docker) in small spaces. Upgrade to 16GB RAM and add a 1TB 2.5” SSD.
  - **SFF/Mini-Tower**: Better for NAS or storage-heavy setups. Add 16GB RAM, multiple 4TB drives, and a PCIe NVMe adapter if needed.
- **Upgrades**:
  - **RAM**: Upgrade to 16GB (2x8GB DDR3-1600, ~$30).
  - **Storage**: Micro: 256GB M.2 SATA SSD (~$30) + 4TB 2.5” HDD (~$100). SFF: 512GB SATA SSD (~$50) + 2x 4TB HDD (~$200).
  - **Cooling**: Update BIOS and consider fan replacement if noisy.
- **BIOS Update**: Critical for fan control and Linux compatibility. Do this before Ubuntu installation if possible.
- **Workload**: Stick to lightweight tasks (Plex, OpenMediaVault, 1-2 VMs) due to RAM and CPU limits. Avoid heavy virtualization unless using SFF with maxed-out resources.

**Cost Estimate**:
- 16GB RAM: $30
- 256GB M.2 SATA SSD: $30
- 4TB 2.5” HDD: $100
- Total: ~$60-160 (micro); add $100-200 for SFF multi-drive setups.

---

### Verification
- After setup, test:
  - SSH accessibility after 24 hours (`ssh admin@192.168.1.100`).
  - No sleep (`systemd-analyze`).
  - Temperatures (`sensors`) and fan behavior.
  - Service functionality (e.g., Plex at `http://192.168.1.100:32400`).
- If issues arise (e.g., fan noise, network drops), check BIOS version, update firmware, or consult [r/homelab](https://reddit.com/r/homelab) or [Dell’s community forums](https://www.dell.com/community).

The Dell 3020 can use nearly the same Ubuntu Server setup as the M900 Tiny with minor tweaks for BIOS, storage, and cooling. It’s a solid choice for a home server, especially the SFF for NAS, though it’s less efficient and RAM-limited compared to the M900 Tiny. If you specify your 3020’s form factor (micro/SFF/mini-tower), CPU, or intended use (e.g., Plex, NAS), I can refine the setup further. Would you like help with a specific software installation or troubleshooting potential 3020 issues?

