---
title: IoT system for monitoring a dam water pumping station (G)
author:
  name: 
  link: 
date: 2025-02-14 09:35:00 -0400
categories: [Privacy]
tags: [iot, docker, esp32, spring-boot, angular]
render_with_liquid: false
---

It seems like you’re asking about an IoT system for monitoring a **dam water pumping station**, which I’ll interpret as a setup to manage and monitor water pumping operations at a dam—covering aspects like water levels, pump performance, and system health. Given your familiarity with Angular, Spring Boot, and Docker from your previous question, I’ll tailor this response to integrate those technologies with an IoT solution, focusing on the ESP32 microcontroller (as recommended earlier) for its versatility and fit for such a task. Let’s break this down into a practical system design.

### System Overview
An IoT system for a dam water pumping station would monitor water levels, pump status (on/off, runtime, faults), flow rates, and environmental conditions (e.g., pressure or temperature), while allowing remote control and real-time alerts. The ESP32 handles the hardware interfacing, Spring Boot serves as the backend, Angular provides the frontend, and Docker containerizes the setup for scalability and deployment ease.

### Recommended Hardware: ESP32
The **ESP32** remains ideal here:
- **Wi-Fi**: Connects to your network for data transmission.
- **GPIO Pins**: Interfaces with sensors and relays for pumps.
- **ADC Channels**: Reads analog sensor data (e.g., water level, pressure).
- **Low Power**: Suitable for continuous operation at a remote dam site.
- **Cost**: Affordable at $5-$10 per unit.

#### Hardware Components
- **ESP32 DevKit**: Core microcontroller.
- **Sensors**:
  - **Ultrasonic Sensor (HC-SR04)**: Measures water level in the dam or reservoir.
  - **Flow Sensor (YF-S201)**: Tracks water flow rate through the pump.
  - **Pressure Sensor (MPX5700)**: Monitors pump pressure.
  - **Current Sensor (ACS712)**: Detects pump power usage for health monitoring.
- **Relays**: Controls pump on/off (e.g., 5V relay for low-power pumps or SSR for high-power AC pumps).
- **Power Supply**: 5V for ESP32, with appropriate AC-DC converters for pumps (e.g., 24V or 120V/240V based on pump specs).

### System Architecture

1. **ESP32 Firmware**
   - **Platform**: Arduino IDE or ESP-IDF.
   - **Functionality**:
     - Poll sensors every few seconds (e.g., water level, flow, pressure, current).
     - Control pump via relay based on commands or thresholds (e.g., turn on if level > 50%, off if < 10%).
     - Send data to Spring Boot via REST API or MQTT for real-time updates.
   - **Sample Code Snippet** (Arduino):
     ```cpp
     #include <WiFi.h>
     #include <HTTPClient.h>
     const char* ssid = "your-wifi";
     const char* password = "your-password";
     const char* server = "http://your-server:8080/api/data";
     #define RELAY_PIN 26
     void setup() {
       Serial.begin(115200);
       WiFi.begin(ssid, password);
       pinMode(RELAY_PIN, OUTPUT);
     }
     void loop() {
       if (WiFi.status() == WL_CONNECTED) {
         HTTPClient http;
         http.begin(server);
         http.POST("{\"level\":" + String(getWaterLevel()) + ",\"flow\":" + String(getFlowRate()) + "}");
         http.end();
       }
       delay(5000); // Poll every 5 seconds
     }
     float getWaterLevel() { /* HC-SR04 logic */ }
     float getFlowRate() { /* YF-S201 logic */ }
     ```

2. **Backend: Spring Boot**
   - **Purpose**: Processes data, stores it, and serves the Angular frontend.
   - **Features**:
     - REST API: `/api/data` (POST for ESP32 data, GET for frontend).
     - MQTT Broker (e.g., Mosquitto): For real-time pump status.
     - Database: PostgreSQL for historical data (levels, pump runtime).
   - **Dockerfile**:
     ```dockerfile
     FROM openjdk:17-jdk-slim
     COPY target/dam-monitoring.jar /app.jar
     ENTRYPOINT ["java", "-jar", "/app.jar"]
     ```

3. **Frontend: Angular**
   - **Purpose**: Displays real-time data and controls pumps.
   - **Features**:
     - Dashboard: Water level gauge, pump status (on/off), flow rate graph.
     - Controls: Toggle pump manually or set automated thresholds.
     - Alerts: Notify if water level is critical or pump fails (via WebSocket/MQTT).
   - **Dockerfile**:
     ```dockerfile
     FROM node:18 AS build
     WORKDIR /app
     COPY . .
     RUN npm install && ng build --prod
     FROM nginx:alpine
     COPY --from=build /app/dist/dam-app /usr/share/nginx/html
     ```

4. **Docker Compose**
   - Combines all services:
     ```yaml
     version: '3'
     services:
       backend:
         build: ./spring-boot
         ports: [ "8080:8080" ]
         depends_on: [ db, mqtt ]
       frontend:
         build: ./angular
         ports: [ "80:80" ]
       db:
         image: postgres:14
         environment:
           - POSTGRES_DB=dam_db
           - POSTGRES_USER=user
           - POSTGRES_PASSWORD=password
         volumes:
           - pgdata:/var/lib/postgresql/data
       mqtt:
         image: eclipse-mosquitto:2
         ports: [ "1883:1883", "9001:9001" ]
     volumes:
       pgdata:
     ```

### Security
- **ESP32**:
  - Use HTTPS for REST calls (Let’s Encrypt via Spring Boot).
  - MQTT with TLS and username/password authentication.
- **Spring Boot**:
  - JWT for API security.
  - Store secrets in environment variables (e.g., `docker-compose.yml` with `.env`).
- **Network**:
  - Deploy behind a VPN (e.g., WireGuard, as in your prior setup).
  - Firewall: Restrict ports 80, 443, 1883 to VPN subnet:
    ```bash
    iptables -A INPUT -s 10.13.13.0/24 -p tcp -m multiport --dports 80,443,1883 -j ACCEPT
    iptables -A INPUT -p tcp -m multiport --dports 80,443,1883 -j DROP
    ```

### Implementation Steps
1. **Hardware**: Connect ESP32 to sensors and relays (e.g., ultrasonic to GPIO 12/13, relay to GPIO 26).
2. **Firmware**: Flash ESP32 with code to read sensors and send data.
3. **Backend**: Deploy Spring Boot with API and MQTT endpoints.
4. **Frontend**: Build Angular app with real-time UI.
5. **Docker**: Run `docker-compose up -d` in your project directory.
6. **Test**: Monitor via `http://your-subdomain.duckdns.org` over VPN, ensuring pump control works.

### Why This Works for a Dam Pumping Station?
- **Scalability**: Add more ESP32s for additional pumps or sensors.
- **Real-Time**: MQTT ensures instant updates for critical events (e.g., pump failure).
- **Security**: VPN and HTTPS protect against unauthorized access.
- **Integration**: Leverages your Angular/Spring Boot/Docker skills.

If you need specific code, hardware wiring diagrams, or help with deployment, let me know! This setup balances cost, functionality, and security for a dam water pumping station.