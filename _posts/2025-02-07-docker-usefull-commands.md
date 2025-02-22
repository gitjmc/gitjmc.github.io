---
title: The most useful Docker commands 
author:
  name: 
  link: 
date: 2025-02-07 09:35:00 -0400
categories: [DEV, Docker]
tags: [cli, docker]
render_with_liquid: false
---

Here’s a **comprehensive list of the most useful Docker commands** for managing containers, images, networks, and volumes. These commands are essential for working with Docker in development, testing, and production environments.

---

### **Docker Basics**
1. **Check Docker Version**:
   ```bash
   docker --version
   ```

2. **Display System-Wide Information**:
   ```bash
   docker info
   ```

---

### **Working with Images**
3. **Pull an Image from Docker Hub**:
   ```bash
   docker pull <image_name>:<tag>
   ```
   Example:
   ```bash
   docker pull nginx:latest
   ```

4. **List Downloaded Images**:
   ```bash
   docker images
   ```

5. **Remove an Image**:
   ```bash
   docker rmi <image_name>:<tag>
   ```

6. **Build an Image from a Dockerfile**:
   ```bash
   docker build -t <image_name>:<tag> .
   ```
   Example:
   ```bash
   docker build -t myapp:1.0 .
   ```

7. **Tag an Image**:
   ```bash
   docker tag <image_id> <new_image_name>:<tag>
   ```

8. **Push an Image to Docker Hub**:
   ```bash
   docker push <image_name>:<tag>
   ```

---

### **Working with Containers**
9. **Run a Container**:
   ```bash
   docker run <image_name>:<tag>
   ```
   Example:
   ```bash
   docker run -d -p 8080:80 nginx:latest
   ```

10. **Run a Container in Interactive Mode**:
    ```bash
    docker run -it <image_name>:<tag> /bin/bash
    ```

11. **List Running Containers**:
    ```bash
    docker ps
    ```

12. **List All Containers (Running and Stopped)**:
    ```bash
    docker ps -a
    ```

13. **Stop a Running Container**:
    ```bash
    docker stop <container_id>
    ```

14. **Start a Stopped Container**:
    ```bash
    docker start <container_id>
    ```

15. **Restart a Container**:
    ```bash
    docker restart <container_id>
    ```

16. **Remove a Stopped Container**:
    ```bash
    docker rm <container_id>
    ```

17. **Force Remove a Running Container**:
    ```bash
    docker rm -f <container_id>
    ```

18. **View Container Logs**:
    ```bash
    docker logs <container_id>
    ```

19. **Follow Container Logs**:
    ```bash
    docker logs -f <container_id>
    ```

20. **Execute a Command in a Running Container**:
    ```bash
    docker exec -it <container_id> <command>
    ```
    Example:
    ```bash
    docker exec -it mycontainer /bin/bash
    ```

21. **Inspect a Container**:
    ```bash
    docker inspect <container_id>
    ```

22. **Copy Files Between Host and Container**:
    ```bash
    docker cp <file_path> <container_id>:<container_path>
    docker cp <container_id>:<container_path> <file_path>
    ```

23. **Pause/Unpause a Container**:
    ```bash
    docker pause <container_id>
    docker unpause <container_id>
    ```

---

### **Networking**
24. **List Networks**:
    ```bash
    docker network ls
    ```

25. **Create a Network**:
    ```bash
    docker network create <network_name>
    ```

26. **Inspect a Network**:
    ```bash
    docker network inspect <network_name>
    ```

27. **Connect a Container to a Network**:
    ```bash
    docker network connect <network_name> <container_id>
    ```

28. **Disconnect a Container from a Network**:
    ```bash
    docker network disconnect <network_name> <container_id>
    ```

29. **Remove a Network**:
    ```bash
    docker network rm <network_name>
    ```

---

### **Volumes**
30. **List Volumes**:
    ```bash
    docker volume ls
    ```

31. **Create a Volume**:
    ```bash
    docker volume create <volume_name>
    ```

32. **Inspect a Volume**:
    ```bash
    docker volume inspect <volume_name>
    ```

33. **Remove a Volume**:
    ```bash
    docker volume rm <volume_name>
    ```

34. **Mount a Volume in a Container**:
    ```bash
    docker run -v <volume_name>:<container_path> <image_name>
    ```

---

### **Docker Compose**
35. **Start Services**:
    ```bash
    docker-compose up
    ```

36. **Start Services in Detached Mode**:
    ```bash
    docker-compose up -d
    ```

37. **Stop Services**:
    ```bash
    docker-compose down
    ```

38. **Rebuild and Restart Services**:
    ```bash
    docker-compose up --build
    ```

39. **View Logs for a Service**:
    ```bash
    docker-compose logs <service_name>
    ```

40. **Scale Services**:
    ```bash
    docker-compose up --scale <service_name>=<num_instances>
    ```

---

### **System Cleanup**
41. **Remove All Stopped Containers**:
    ```bash
    docker container prune
    ```

42. **Remove All Unused Images**:
    ```bash
    docker image prune -a
    ```

43. **Remove All Unused Networks**:
    ```bash
    docker network prune
    ```

44. **Remove All Unused Volumes**:
    ```bash
    docker volume prune
    ```

45. **Remove All Unused Objects**:
    ```bash
    docker system prune -a
    ```

---

### **Advanced Commands**
46. **Save an Image to a Tar File**:
    ```bash
    docker save -o <file_name>.tar <image_name>
    ```

47. **Load an Image from a Tar File**:
    ```bash
    docker load -i <file_name>.tar
    ```

48. **View Resource Usage**:
    ```bash
    docker stats
    ```

49. **View Docker Events**:
    ```bash
    docker events
    ```

50. **Check Disk Usage**:
    ```bash
    docker system df
    ```

---

### **Docker Swarm (For Orchestration)**
51. **Initialize a Swarm**:
    ```bash
    docker swarm init
    ```

52. **Join a Swarm as a Worker**:
    ```bash
    docker swarm join --token <worker_token> <manager_ip>:<port>
    ```

53. **List Nodes in the Swarm**:
    ```bash
    docker node ls
    ```

54. **Deploy a Stack**:
    ```bash
    docker stack deploy -c <compose_file> <stack_name>
    ```

55. **List Services in a Stack**:
    ```bash
    docker stack services <stack_name>
    ```

---

This list covers the **most useful Docker commands** for day-to-day operations. Let me know if you need further clarification or examples! 🐳🚀

