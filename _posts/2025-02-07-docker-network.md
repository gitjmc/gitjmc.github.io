---
title: Docker Network 
author:
  name: 
  link: 
date: 2025-02-07 10:35:00 -0400
categories: [DEV, Docker]
tags: [network, cmd, docker]
render_with_liquid: false
---

Certainly! Docker networking allows containers to communicate with each other, the host system, and external networks. Docker provides several networking options, each suited for different use cases. Below, I’ll explain Docker networking in detail, including **network types**, **commands**, and **examples**.

---

### **Docker Network Types**

Docker supports several network drivers, each serving a specific purpose:

1. **Bridge Network**:
   - Default network driver.
   - Containers on the same bridge network can communicate with each other.
   - Isolated from the host and external networks by default.
   - Suitable for single-host deployments.

2. **Host Network**:
   - Containers share the host’s network stack.
   - No network isolation between the container and the host.
   - Suitable for high-performance use cases.

3. **Overlay Network**:
   - Enables multi-host communication in Docker Swarm.
   - Containers on different hosts can communicate securely.
   - Suitable for distributed applications.

4. **Macvlan Network**:
   - Assigns a MAC address to each container, making it appear as a physical device on the network.
   - Suitable for legacy applications that require a MAC address.

5. **None Network**:
   - Disables all networking for the container.
   - Useful for completely isolated containers.

---

### **Docker Network Commands**

#### **1. List Networks**
```bash
docker network ls
```
Output:
```
NETWORK ID     NAME      DRIVER    SCOPE
abc123def456   bridge    bridge    local
def456abc123   host      host      local
ghi789jkl012   none      null      local
```

#### **2. Create a Network**
```bash
docker network create <network_name>
```
Example:
```bash
docker network create my_bridge_network
```

#### **3. Inspect a Network**
```bash
docker network inspect <network_name>
```
Example:
```bash
docker network inspect my_bridge_network
```

#### **4. Connect a Container to a Network**
```bash
docker network connect <network_name> <container_name>
```
Example:
```bash
docker network connect my_bridge_network my_container
```

#### **5. Disconnect a Container from a Network**
```bash
docker network disconnect <network_name> <container_name>
```
Example:
```bash
docker network disconnect my_bridge_network my_container
```

#### **6. Remove a Network**
```bash
docker network rm <network_name>
```
Example:
```bash
docker network rm my_bridge_network
```

---

### **Examples of Docker Networking**

#### **Example 1: Bridge Network**
1. **Create a Bridge Network**:
   ```bash
   docker network create my_bridge
   ```

2. **Run Containers on the Bridge Network**:
   ```bash
   docker run -d --name container1 --network my_bridge nginx
   docker run -d --name container2 --network my_bridge nginx
   ```

3. **Verify Communication Between Containers**:
   ```bash
   docker exec -it container1 ping container2
   ```

#### **Example 2: Host Network**
1. **Run a Container on the Host Network**:
   ```bash
   docker run -d --name host_container --network host nginx
   ```

2. **Access the Container**:
   - Since the container uses the host’s network, you can access it directly via `localhost`:
     ```bash
     curl localhost
     ```

#### **Example 3: Overlay Network (Docker Swarm)**
1. **Initialize Docker Swarm**:
   ```bash
   docker swarm init
   ```

2. **Create an Overlay Network**:
   ```bash
   docker network create --driver overlay my_overlay
   ```

3. **Deploy a Service on the Overlay Network**:
   ```bash
   docker service create --name web --network my_overlay --replicas 3 nginx
   ```

4. **Verify Communication Between Containers on Different Hosts**:
   - Containers on different hosts can communicate using the overlay network.

#### **Example 4: Macvlan Network**
1. **Create a Macvlan Network**:
   ```bash
   docker network create -d macvlan --subnet=192.168.1.0/24 --gateway=192.168.1.1 -o parent=eth0 my_macvlan
   ```

2. **Run a Container on the Macvlan Network**:
   ```bash
   docker run -d --name macvlan_container --network my_macvlan nginx
   ```

3. **Access the Container**:
   - The container will have its own MAC address and IP on the physical network.

---

### **Advanced Networking Scenarios**

#### **1. Linking Containers**
- **Legacy Linking** (deprecated):
  ```bash
  docker run -d --name container1 nginx
  docker run -d --name container2 --link container1 nginx
  ```

- **Modern Alternative** (use custom bridge networks):
  ```bash
  docker network create my_network
  docker run -d --name container1 --network my_network nginx
  docker run -d --name container2 --network my_network nginx
  ```

#### **2. Exposing Ports**
- Expose a container port to the host:
  ```bash
  docker run -d --name web -p 8080:80 nginx
  ```
  - Access the container via `http://localhost:8080`.

#### **3. DNS Resolution**
- Docker provides automatic DNS resolution for containers on the same network:
  ```bash
  docker run -d --name container1 --network my_bridge nginx
  docker run -d --name container2 --network my_bridge busybox ping container1
  ```

#### **4. Network Aliases**
- Assign a custom DNS alias to a container:
  ```bash
  docker run -d --name container1 --network my_bridge --network-alias app nginx
  docker run -d --name container2 --network my_bridge busybox ping app
  ```

---

### **Troubleshooting Docker Networks**

1. **Check Container IP Address**:
   ```bash
   docker inspect -f '{{range .NetworkSettings.Networks}}{{.IPAddress}}{{end}}' <container_name>
   ```

2. **Test Connectivity Between Containers**:
   ```bash
   docker exec -it <container1> ping <container2>
   ```

3. **View Network Logs**:
   ```bash
   docker logs <container_name>
   ```

---

### **Summary of Docker Network Drivers**
| **Network Type** | **Use Case**                                                                 |
|------------------|-----------------------------------------------------------------------------|
| **Bridge**       | Default network for single-host communication.                              |
| **Host**         | High-performance use cases with no network isolation.                       |
| **Overlay**      | Multi-host communication in Docker Swarm.                                   |
| **Macvlan**      | Assign MAC addresses to containers for legacy applications.                 |
| **None**         | Disable networking for completely isolated containers.                      |

---

This guide covers the essentials of Docker networking. Let me know if you need further clarification or help with specific scenarios!

