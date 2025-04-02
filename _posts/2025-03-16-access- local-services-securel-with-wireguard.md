---
title: Access local services securel with Wireguard 
author:
  name: 
  link: 
date: 2025-03-09 07:35:00 -0400
categories: [DEV]
tags: [wireguard]
render_with_liquid: false
---

To use WireGuard to access services running inside a set of containers on a remote host with a Point-to-Point (P2P) topology, follow these steps:

---

## Use case 1

---

### **1. Understand the Setup**
- **Remote Host**: The host running the containers.
- **Local Machine**: The machine from which you want to access the services.
- **WireGuard**: A VPN tool that creates a secure tunnel between the remote host and your local machine.
- **Point-to-Point Topology**: A direct connection between two endpoints (your local machine and the remote host).

---

### **2. Install WireGuard**
Install WireGuard on both the **remote host** and your **local machine**.

- **Linux**:
  ```bash
  sudo apt update
  sudo apt install wireguard
  ```
- **Windows/macOS**: Download and install WireGuard from the official website: [https://www.wireguard.com/install/](https://www.wireguard.com/install/).

---

### **3. Generate WireGuard Keys**
On both the remote host and local machine, generate private and public keys:

```bash
wg genkey | tee privatekey | wg pubkey > publickey
```

- Save the private key securely.
- Share the public key with the other party.

---

### **4. Configure WireGuard on the Remote Host**
Create a WireGuard configuration file on the remote host (e.g., `/etc/wireguard/wg0.conf`):

```ini
[Interface]
Address = 10.0.0.1/24
PrivateKey = <REMOTE_HOST_PRIVATE_KEY>
ListenPort = 51820

[Peer]
PublicKey = <LOCAL_MACHINE_PUBLIC_KEY>
AllowedIPs = 10.0.0.2/32
```

- Replace `<REMOTE_HOST_PRIVATE_KEY>` with the private key of the remote host.
- Replace `<LOCAL_MACHINE_PUBLIC_KEY>` with the public key of your local machine.
- `10.0.0.1/24` is the IP address assigned to the remote host in the VPN network.

---

### **5. Configure WireGuard on the Local Machine**
Create a WireGuard configuration file on your local machine (e.g., `/etc/wireguard/wg0.conf`):

```ini
[Interface]
Address = 10.0.0.2/24
PrivateKey = <LOCAL_MACHINE_PRIVATE_KEY>

[Peer]
PublicKey = <REMOTE_HOST_PUBLIC_KEY>
Endpoint = <REMOTE_HOST_PUBLIC_IP>:51820
AllowedIPs = 10.0.0.1/32
PersistentKeepalive = 25
```

- Replace `<LOCAL_MACHINE_PRIVATE_KEY>` with the private key of your local machine.
- Replace `<REMOTE_HOST_PUBLIC_KEY>` with the public key of the remote host.
- Replace `<REMOTE_HOST_PUBLIC_IP>` with the public IP address of the remote host.
- `10.0.0.2/24` is the IP address assigned to your local machine in the VPN network.

---

### **6. Start WireGuard**
- On the remote host:
  ```bash
  sudo wg-quick up wg0
  ```
- On the local machine:
  ```bash
  sudo wg-quick up wg0
  ```

---

### **7. Configure Routing for Containers**
Ensure the containers on the remote host can communicate over the WireGuard interface.

- If the containers are running on a bridge network, you may need to add a route to the WireGuard subnet (`10.0.0.0/24`) on the Docker bridge.
- Example:
  ```bash
  sudo ip route add 10.0.0.0/24 dev docker0
  ```

---

### **8. Access Services**
- Use the IP address of the remote host in the VPN network (`10.0.0.1`) to access services running inside the containers.
- Example: If a container is running a web server on port `8080`, you can access it from your local machine at `http://10.0.0.1:8080`.

---

### **9. Persist Configuration**
To ensure WireGuard starts automatically on boot:

- On Linux:
  ```bash
  sudo systemctl enable wg-quick@wg0
  ```

---

### **10. Test the Connection**
- Use `ping` to test connectivity:
  ```bash
  ping 10.0.0.1
  ```
- Check the WireGuard status:
  ```bash
  sudo wg
  ```

---

### **Optional: Firewall Configuration**
Ensure the WireGuard port (`51820` by default) is open on the remote host's firewall.

- Example for `ufw`:
  ```bash
  sudo ufw allow 51820/udp
  ```

---

## Use case 2

Yes, that's correct! By following the steps outlined above, you can run Nextcloud (or any other service) in a container on a remote host and securely access it from any device connected to your WireGuard VPN. Here's how it works specifically for your use case:

---

### **How It Works for Nextcloud in a Container**
1. **Nextcloud in a Container**: You run Nextcloud in a Docker container on the remote host, exposing it on a specific port (e.g., `8080`).
2. **WireGuard VPN**: You set up WireGuard to create a secure tunnel between the remote host and your devices (e.g., laptop, phone, etc.).
3. **Access Nextcloud**: Once connected to the WireGuard VPN, you can access Nextcloud using the private IP address assigned to the remote host in the VPN network (e.g., `10.0.0.1:8080`).

---

### **Steps Specific to Nextcloud**
1. **Run Nextcloud in a Container**:
   - On the remote host, start a Nextcloud container:
     ```bash
     docker run -d \
       --name nextcloud \
       -p 8080:80 \
       nextcloud
     ```
   - This exposes Nextcloud on port `8080` of the remote host.

2. **Configure WireGuard**:
   - Follow the steps in the previous answer to set up WireGuard on the remote host and your devices.

3. **Access Nextcloud**:
   - Once connected to the WireGuard VPN, open a browser on your device and navigate to:
     ```
     http://10.0.0.1:8080
     ```
   - This will load the Nextcloud interface.

---

### **Access from Any Device**
To allow any device to access Nextcloud via WireGuard:
1. **Install WireGuard on the Device**:
   - Install WireGuard on the device you want to use (e.g., laptop, phone, tablet).
   - Use the same WireGuard configuration (with the device's unique private/public key pair) to connect to the VPN.

2. **Add the Device as a Peer**:
   - On the remote host's WireGuard configuration (`/etc/wireguard/wg0.conf`), add the new device as a peer:
     ```ini
     [Peer]
     PublicKey = <DEVICE_PUBLIC_KEY>
     AllowedIPs = 10.0.0.3/32
     ```
   - Replace `<DEVICE_PUBLIC_KEY>` with the public key of the new device.
   - Assign a unique IP address (e.g., `10.0.0.3`) to the device.

3. **Restart WireGuard**:
   - Restart WireGuard on the remote host to apply the changes:
     ```bash
     sudo wg-quick down wg0
     sudo wg-quick up wg0
     ```

4. **Connect the Device**:
   - On the device, connect to the WireGuard VPN using its configuration.
   - Once connected, you can access Nextcloud at `http://10.0.0.1:8080`.

---

### **Benefits of This Setup**
- **Security**: WireGuard encrypts all traffic between your devices and the remote host.
- **Privacy**: Your Nextcloud instance is not exposed to the public internet.
- **Accessibility**: You can access Nextcloud from any device connected to the VPN, whether you're at home, at work, or on the go.

---

### **Important Considerations**
1. **Performance**: WireGuard is lightweight and fast, but the speed of your connection will depend on the internet speed of the remote host and your device.
2. **Dynamic IP Address**: If the remote host has a dynamic public IP address, consider using a Dynamic DNS (DDNS) service to map a domain name to the IP address.
3. **Firewall Rules**: Ensure the WireGuard port (`51820` by default) is open on the remote host's firewall.
4. **Container Networking**: If your containers are on a custom Docker network, ensure the WireGuard interface can route traffic to that network.

---

By following these steps, you can securely run Nextcloud in a container on a remote host and access it from any device connected to your WireGuard VPN.

