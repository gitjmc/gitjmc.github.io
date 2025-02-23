---
title: Setting up, contoling and minotoring an IoT system (G)
author:
  name: 
  link: 
date: 2025-02-14 19:12:00 -0400
categories: [Privacy]
tags: [iot, docker, esp32, spring-boot, angular]
render_with_liquid: false
---
Setting up an IoT system to control and monitor your garden irrigation, pool pump, indoor/outdoor lights, heater, and other devices with Angular, Spring Boot, and Docker is a great choice, as these technologies align well with a secure, scalable, and maintainable solution. Given your requirements and tech stack, I’ll recommend a microcontroller, outline a setup, and emphasize security considerations.

### Recommended Microcontroller: ESP32
The **ESP32** is my top pick for your IoT project due to its versatility, power, and compatibility with your goals:
- **Dual-Core Processor**: Handles multiple tasks (e.g., monitoring sensors and controlling devices) efficiently.
- **Wi-Fi and Bluetooth**: Built-in connectivity for remote control and integration with your Angular frontend via Spring Boot.
- **GPIO Pins**: Plenty of inputs/outputs (around 36) to manage irrigation valves, pool pumps, lights, heaters, and sensors.
- **Low Cost**: Around $5-$10 per unit, making it budget-friendly for scaling to multiple devices.
- **Community Support**: Extensive libraries and examples for Arduino IDE, MicroPython, or ESP-IDF, easing development.
- **Power Efficiency**: Sleep modes for battery-powered outdoor use if needed.

Compared to alternatives:
- **Raspberry Pi**: More powerful but overkill for basic control, costlier ($35+), and less energy-efficient for simple IoT tasks.
- **Arduino Uno**: Cheaper but lacks built-in Wi-Fi, requiring extra modules, and has fewer pins.
- **ESP8266**: A predecessor to ESP32, cheaper but less powerful and with fewer pins.

### System Architecture
Here’s how you can structure this with ESP32, Angular, Spring Boot, and Docker:

1. **Hardware Setup**
   - **ESP32 Modules**: One per control point (e.g., garden, pool, indoor lights). Each connects to:
     - **Relays**: For switching irrigation valves (e.g., 5V or 24V solenoids), pool pump (120V/240V AC via a suitable relay), lights, and heater.
     - **Sensors**: Soil moisture (capacitive type), water level (for pool/irrigation tank), temperature (DS18B20 for heater/pool), and light sensors (LDR or BH1750).
   - **Power Supply**: 5V USB for ESP32, with appropriate AC-DC converters for relays (e.g., HLK-PM01 for 220V to 5V).

2. **Firmware on ESP32**
   - Use **Arduino IDE** or **PlatformIO** with C++:
     - **Libraries**: `WiFi`, `HTTPClient` (for API calls), `OneWire` (temperature), `PubSubClient` (MQTT for real-time updates).
     - **Logic**: Read sensor data, control relays based on commands, and send status to Spring Boot via REST or MQTT.
   - Example: Toggle a relay for the pool pump when a Spring Boot API endpoint is called.

3. **Backend: Spring Boot**
   - **Purpose**: Centralized logic, API, and data storage.
   - **Features**:
     - REST API endpoints (e.g., `/api/devices/{id}/toggle`, `/api/sensors/{id}/data`).
     - MQTT broker integration (e.g., Eclipse Mosquitto) for real-time updates.
     - Database (e.g., PostgreSQL) for logging sensor data and schedules.
   - **Docker**: Containerize with `Dockerfile`:
     ```dockerfile
     FROM openjdk:17-jdk-slim
     WORKDIR /app
     COPY target/your-app.jar app.jar
     ENTRYPOINT ["java", "-jar", "app.jar"]
     ```
     Run with `docker run -p 8080:8080 your-app`.

4. **Frontend: Angular**
   - **Purpose**: User interface for control and monitoring.
   - **Features**:
     - Dashboard with device status (e.g., irrigation on/off, pool pump runtime, light states).
     - Real-time sensor data via WebSocket or polling.
     - Schedules (e.g., irrigate at 6 AM, turn heater on if temp < 20°C).
   - **Docker**: Use an Nginx base image to serve the Angular app:
     ```dockerfile
     FROM node:18 AS build
     WORKDIR /app
     COPY . .
     RUN npm install && ng build --prod
     FROM nginx:alpine
     COPY --from=build /app/dist/your-app /usr/share/nginx/html
     ```
     Run with `docker run -p 80:80 your-angular-app`.

5. **Docker Compose**
   - Tie it all together:
     ```yaml
     version: '3'
     services:
       backend:
         build: ./spring-boot
         ports:
           - "8080:8080"
         depends_on:
           - db
           - mqtt
       frontend:
         build: ./angular
         ports:
           - "80:80"
       db:
         image: postgres:14
         environment:
           - POSTGRES_DB=iot_db
           - POSTGRES_USER=user
           - POSTGRES_PASSWORD=password
         volumes:
           - pgdata:/var/lib/postgresql/data
       mqtt:
         image: eclipse-mosquitto:2
         ports:
           - "1883:1883"
           - "9001:9001" # WebSocket
     volumes:
       pgdata:
     ```

### Security Considerations
Security is critical since this system is internet-facing and controls physical devices:
- **ESP32**:
  - Use **HTTPS** for API calls (self-signed certs or Let’s Encrypt via Spring Boot).
  - Implement **OTA updates** with password protection (`ArduinoOTA.setPassword`).
  - Hardcode Wi-Fi credentials securely or use a provisioning step (e.g., Wi-Fi Manager).
- **Spring Boot**:
  - Enable **Spring Security**: JWT authentication for API access.
  - Validate all inputs to prevent injection attacks.
  - Use environment variables for secrets (e.g., database creds in `.env` with Docker).
- **Angular**:
  - Sanitize user inputs to avoid XSS.
  - Use HTTPS for all frontend-backend communication.
- **Network**:
  - Place devices behind a **VPN** (e.g., WireGuard, as in your previous setup) to restrict access.
  - Firewall rules (e.g., `iptables`): Allow only VPN subnet (e.g., `10.13.13.0/24`) to ports 80, 443, 1883.
  - Example:
    ```bash
    iptables -A INPUT -s 10.13.13.0/24 -p tcp -m multiport --dports 80,443,1883 -j ACCEPT
    iptables -A INPUT -p tcp -m multiport --dports 80,443,1883 -j DROP
    ```
- **General**:
  - Regularly update firmware, Docker images, and dependencies.
  - Monitor logs for anomalies (e.g., Spring Boot logs, Mosquitto logs).

### Implementation Steps
1. **Hardware**: Wire ESP32s to relays and sensors (e.g., 5V relay for lights, 24V solenoid for irrigation).
2. **Firmware**: Flash ESP32s with code to poll sensors and call Spring Boot APIs or publish to MQTT.
3. **Backend**: Set up Spring Boot with endpoints and MQTT subscription.
4. **Frontend**: Build Angular UI with controls and charts (e.g., Chart.js for sensor data).
5. **Docker**: Run `docker-compose up -d` to launch services.
6. **Secure**: Configure VPN, HTTPS, and firewall rules.

### Why ESP32 Fits?
It’s powerful enough to handle your diverse devices, integrates seamlessly with Spring Boot via REST/MQTT, and supports Angular’s real-time needs through a robust backend. Its low cost and flexibility make it ideal for a garden-to-indoor setup.

