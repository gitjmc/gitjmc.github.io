---
title: Dokerize mosquitto throw traefik
author:
  name: 
  link: 
date: 2025-04-30 12:35:00 -0400
categories: [Docker]
tags: [dockerize, mosquitto, mqtt, tarfeik]
render_with_liquid: false
---

Yes, it is possible to dockerize a **Mosquitto MQTT broker** with **TLS** (Transport Layer Security) using **Traefik** as a reverse proxy to handle TLS termination and certificate management (e.g., via Let’s Encrypt). This setup simplifies securing MQTT communication by offloading TLS handling to Traefik, allowing Mosquitto to operate on unencrypted ports internally while exposing secure endpoints externally. Below, I’ll explain the process in detail, including how certificates work in this context, why Traefik is useful, and provide a step-by-step guide with a sample configuration.

---

## **Why Use Traefik with Mosquitto for TLS?**

Traefik is a modern reverse proxy that integrates well with Docker, automatically discovers services, and simplifies TLS certificate management using Let’s Encrypt. Using Traefik with Mosquitto offers several benefits:
- **Centralized TLS Management**: Traefik handles TLS termination, so Mosquitto doesn’t need to manage certificates directly.
- **Automatic Certificate Renewal**: Traefik can obtain and renew Let’s Encrypt certificates, reducing maintenance overhead.
- **Scalability**: Traefik supports load balancing across multiple Mosquitto instances if needed.
- **Simplified Configuration**: Traefik’s dynamic configuration via Docker labels reduces the need for manual certificate setup in Mosquitto.
- **WebSocket Support**: Traefik can handle MQTT over WebSockets, useful for browser-based or IoT applications.

In this setup:
- **Mosquitto** runs in a Docker container, listening on unencrypted ports (e.g., 1883 for MQTT, 9001 for WebSockets).
- **Traefik** listens on secure ports (e.g., 8883 for MQTT, 443 for WebSockets), terminates TLS, and forwards traffic to Mosquitto’s unencrypted ports.
- **Let’s Encrypt** provides trusted certificates, eliminating the need for self-signed certificates (though self-signed certificates can be used for testing).

---

## **How Certificates Work in This Setup**

To understand how certificates are managed with Traefik and Mosquitto, let’s revisit the certificate concepts from your previous question and apply them to this context:

### **Certificate Roles**
- **CA Certificate (`cafile`)**: In a traditional Mosquitto TLS setup, the client (e.g., ESP32) uses the CA certificate to verify the server’s certificate. With Traefik, the CA certificate is typically the Let’s Encrypt root or intermediate certificate, which is already trusted by most clients (e.g., browsers, MQTT libraries).
- **Server Certificate (`certfile`)**: Traefik presents a server certificate (obtained from Let’s Encrypt) to clients during the TLS handshake. Mosquitto doesn’t need a server certificate since Traefik handles TLS.
- **Private Key (`keyfile`)**: Traefik uses the private key associated with the Let’s Encrypt certificate for cryptographic operations. Mosquitto doesn’t need a private key in this setup.
- **Client Certificates** (optional): If mutual TLS is required, Mosquitto can be configured to verify client certificates, but this is less common with Traefik setups and requires additional configuration.

### **File Formats**
- Traefik stores Let’s Encrypt certificates in a JSON file (`acme.json`) in a mounted volume, not as separate `.pem`, `.crt`, or `.key` files.
- Clients (e.g., ESP32) may need the Let’s Encrypt CA certificate (e.g., `ISRG Root X1`) in `.pem` format to verify Traefik’s certificate. This is often unnecessary for modern clients, as Let’s Encrypt’s CA is widely trusted.
- If you use self-signed certificates for testing, you’ll need `.pem` or `.crt` files for the CA and server certificates, similar to a traditional Mosquitto setup.

### **TLS Flow**
1. The client (e.g., ESP32) connects to Traefik on port 8883 (MQTT) or 443 (WebSocket).
2. Traefik presents its Let’s Encrypt certificate to the client.
3. The client verifies the certificate using the Let’s Encrypt CA (usually built into the client’s trust store).
4. Traefik decrypts the incoming traffic and forwards it to Mosquitto on an unencrypted port (e.g., 1883).
5. Mosquitto processes the MQTT messages and responds via Traefik, which re-encrypts the response.

This approach simplifies Mosquitto’s configuration, as it doesn’t need to handle TLS directly, but it requires Traefik to be properly configured for TCP routing and TLS.

---

## **Challenges and Considerations**
- **SNI Support**: Traefik uses Server Name Indication (SNI) for routing, but many MQTT clients (e.g., `mosquitto_pub`) don’t support SNI. Traefik’s `HostSNI(`*`)` rule can bypass this for TCP services, but testing with such clients may require workarounds.[](https://community.traefik.io/t/mosquitto-mqtt-and-tls-without-client-hostsni/21912)
- **Port Exposure**: Mosquitto’s ports (1883, 9001) should not be exposed directly in Docker to ensure all traffic goes through Traefik. Use `expose` instead of `ports` in the Docker Compose file.[](https://gist.github.com/gimiki/628e2ca10f026975f00f34e4d1f4ff23?permalink_comment_id=3463714)
- **WebSocket Support**: If you need MQTT over WebSockets (e.g., for browser-based clients), Traefik must be configured to handle WebSocket traffic on port 443.[](https://medium.com/himinds/mqtt-broker-with-secure-websocket-using-traefik-docker-compose-and-lets-encrypt-2b8e32207555)
- **Certificate Verification**: Clients must trust the Let’s Encrypt CA. For self-signed certificates, you’ll need to provide the CA certificate to clients (e.g., ESP32).[](https://primalcortex.wordpress.com/2016/03/31/mqtt-mosquitto-broker-with-ssltls-transport-security/)
- **Mutual TLS**: If you require client certificate authentication, Mosquitto must handle it, as Traefik typically doesn’t verify client certificates for TCP services. This requires additional Mosquitto configuration.[](https://www.espboards.dev/blog/secure-mqtt-broker-docker-hass/)

---

## **Step-by-Step Guide to Dockerize Mosquitto with TLS via Traefik**

Below is a detailed guide to set up a Mosquitto MQTT broker with TLS using Traefik and Let’s Encrypt in a Docker environment. This setup assumes you have a publicly accessible domain (e.g., `mqtt.example.com`) for Let’s Encrypt.

### **Prerequisites**
- Docker and Docker Compose installed.
- A publicly accessible domain (e.g., `mqtt.example.com`) pointing to your server’s public IP.
- Ports 80, 443, and 8883 open on your server for HTTP, HTTPS, and MQTT.
- Basic understanding of Docker, Traefik, and Mosquitto.

### **1. Create Directory Structure**
Create a project directory with the following structure:
```
mosquitto-traefik/
├── traefik/
│   ├── traefik.yml
│   └── acme.json
├── mosquitto/
│   ├── config/
│   │   ├── mosquitto.conf
│   │   └── passwd
│   ├── data/
│   └── log/
├── .env
└── docker-compose.yml
```

### **2. Configure Environment Variables**
Create a `.env` file to store configuration variables:
```
MQTT_DOCKER_TAG=latest
TRAEFIK_DOCKER_TAG=latest
MQTT_LETSENCRYPT_EMAIL=your-email@example.com
MQTT_VIRTUAL_HOST=mqtt.example.com
```

- `MQTT_VIRTUAL_HOST`: Your domain for the MQTT broker (e.g., `mqtt.example.com`).
- `MQTT_LETSENCRYPT_EMAIL`: Email for Let’s Encrypt notifications.

### **3. Configure Traefik**
Create `traefik/traefik.yml` for Traefik’s static configuration:
```yaml
entryPoints:
  web:
    address: ":80"
  websecure:
    address: ":443"
  mqtt:
    address: ":8883"

providers:
  docker:
    exposedByDefault: false
    network: mqtt

certificatesResolvers:
  letsencrypt:
    acme:
      email: "${MQTT_LETSENCRYPT_EMAIL}"
      storage: /letsencrypt/acme.json
      tlsChallenge: true

log:
  level: DEBUG
```

- Defines entry points for HTTP (80), HTTPS (443), and MQTT (8883).
- Configures Let’s Encrypt for automatic certificate generation.
- Uses the `mqtt` Docker network for communication.

Create an empty `traefik/acme.json` and set permissions:
```bash
touch traefik/acme.json
chmod 600 traefik/acme.json
```

### **4. Configure Mosquitto**
Create `mosquitto/config/mosquitto.conf` for Mosquitto:
```conf
persistence true
persistence_location /mosquitto/data/
log_dest file /mosquitto/log/mosquitto.log

listener 1883
protocol mqtt
allow_anonymous false
password_file /mosquitto/config/passwd

listener 9001
protocol websockets
allow_anonymous false
password_file /mosquitto/config/passwd
```

- Listens on port 1883 for MQTT and 9001 for WebSockets.
- Disables anonymous access and uses a password file for authentication.

Create a password file for Mosquitto:
```bash
docker run -it eclipse-mosquitto:${MQTT_DOCKER_TAG} mosquitto_passwd -c /mosquitto/config/passwd <username>
```
Copy the generated `passwd` file to `mosquitto/config/passwd`.

### **5. Create Docker Compose File**
Create `docker-compose.yml` to define Traefik and Mosquitto services:
```yaml
version: "3.8"

services:
  traefik:
    image: traefik:${TRAEFIK_DOCKER_TAG}
    container_name: traefik
    ports:
      - "80:80"
      - "443:443"
      - "8883:8883"
    volumes:
      - ./traefik/traefik.yml:/etc/traefik/traefik.yml
      - ./traefik/acme.json:/letsencrypt/acme.json
      - /var/run/docker.sock:/var/run/docker.sock
    networks:
      - mqtt
    restart: always

  mosquitto:
    image: eclipse-mosquitto:${MQTT_DOCKER_TAG}
    container_name: mosquitto
    expose:
      - "1883"
      - "9001"
    volumes:
      - ./mosquitto/config:/mosquitto/config
      - ./mosquitto/data:/mosquitto/data
      - ./mosquitto/log:/mosquitto/log
    networks:
      - mqtt
    restart: always
    labels:
      - "traefik.enable=true"
      - "traefik.docker.network=mqtt"
      # MQTT TCP Service
      - "traefik.tcp.routers.mqtt.rule=HostSNI(`*`)"
      - "traefik.tcp.routers.mqtt.entrypoints=mqtt"
      - "traefik.tcp.routers.mqtt.tls=true"
      - "traefik.tcp.routers.mqtt.tls.certresolver=letsencrypt"
      - "traefik.tcp.services.mqtt.loadbalancer.server.port=1883"
      # WebSocket Service
      - "traefik.http.routers.mqtt-ws.rule=Host(`${MQTT_VIRTUAL_HOST}`)"
      - "traefik.http.routers.mqtt-ws.entrypoints=websecure"
      - "traefik.http.routers.mqtt-ws.tls=true"
      - "traefik.http.routers.mqtt-ws.tls.certresolver=letsencrypt"
      - "traefik.http.services.mqtt-ws.loadbalancer.server.port=9001"

networks:
  mqtt:
    driver: bridge
```

- **Traefik**: Exposes ports 80, 443, and 8883, mounts configuration and certificate files, and listens for Docker events.
- **Mosquitto**: Exposes internal ports 1883 and 9001 (not bound to the host), mounts configuration and data volumes, and uses Traefik labels for routing.
- **Labels**:
  - `HostSNI(`*`)` for MQTT TCP routing (bypasses SNI issues).
  - TLS enabled with Let’s Encrypt (`tls=true`, `certresolver=letsencrypt`).
  - WebSocket routing uses the domain (`Host(${MQTT_VIRTUAL_HOST})`).

### **6. Start the Services**
Run the Docker Compose stack:
```bash
docker-compose up -d
```

- Traefik will obtain a Let’s Encrypt certificate for `mqtt.example.com`.
- Mosquitto will start and listen on 1883 (MQTT) and 9001 (WebSockets) internally.
- Traefik will route external traffic from port 8883 (MQTT) to Mosquitto’s 1883 and from port 443 (WebSocket) to Mosquitto’s 9001.

### **7. Configure ESP32 Client**
Modify the ESP32 code to connect to the secure MQTT endpoint. Since Traefik uses Let’s Encrypt, most clients trust the CA by default, but you may need to provide the Let’s Encrypt CA certificate for some MQTT libraries.

Example Arduino code (using `WiFiClientSecure` and `PubSubClient`):
```cpp
#include <WiFi.h>
#include <WiFiClientSecure.h>
#include <PubSubClient.h>

// WiFi credentials
const char* ssid = "YOUR_WIFI_SSID";
const char* password = "YOUR_WIFI_PASSWORD";

// MQTT broker settings
const char* mqtt_server = "mqtt.example.com";
const int mqtt_port = 8883;
const char* mqtt_user = "YOUR_MQTT_USERNAME";
const char* mqtt_password = "YOUR_MQTT_PASSWORD";
const char* mqtt_client_id = "ESP32_Client";

// Let’s Encrypt CA certificate (optional, if not trusted by default)
const char* ca_cert = \
"-----BEGIN CERTIFICATE-----\n" \
"MIIFazCCA1OgAwIBAgIRAIIQz7DSQONZRGPgu2OCiwAwDQYJKoZIhvcNAQELBQAw\n" \
"... (ISRG Root X1 certificate, obtainable from https://letsencrypt.org/certs/) ...\n" \
"-----END CERTIFICATE-----\n";

WiFiClientSecure espClient;
PubSubClient client(espClient);

void setup() {
  Serial.begin(115200);

  // Connect to WiFi
  WiFi.begin(ssid, password);
  while (WiFi.status() != WL_CONNECTED) {
    delay(1000);
    Serial.println("Connecting to WiFi...");
  }
  Serial.println("Connected to WiFi");

  // Set CA certificate (optional if Let’s Encrypt is trusted)
  espClient.setCACert(ca_cert);

  // Configure MQTT
  client.setServer(mqtt_server, mqtt_port);
  client.setCallback(callback);

  // Connect to MQTT broker
  reconnect();
}

void callback(char* topic, byte* payload, unsigned int length) {
  Serial.print("Message received on topic: ");
  Serial.println(topic);
  Serial.print("Payload: ");
  for (int i = 0; i < length; i++) {
    Serial.print((char)payload[i]);
  }
  Serial.println();
}

void reconnect() {
  while (!client.connected()) {
    Serial.println("Attempting MQTT connection...");
    if (client.connect(mqtt_client_id, mqtt_user, mqtt_password)) {
      Serial.println("Connected to MQTT broker");
      client.subscribe("test/topic");
    } else {
      Serial.print("Failed, rc=");
      Serial.print(client.state());
      Serial.println(" Retrying in 5 seconds...");
      delay(5000);
    }
  }
}

void loop() {
  if (!client.connected()) {
    reconnect();
  }
  client.loop();
}
```

- Use `mqtt.example.com` as the broker address and port 8883.
- Include the Let’s Encrypt CA certificate if your MQTT library doesn’t trust it by default.

### **8. Test the Setup**
- **MQTT**: Use an MQTT client (e.g., MQTT Explorer) to connect to `mqtt.example.com:8883` with TLS enabled, using the username and password from the `passwd` file.
- **WebSocket**: Connect to `wss://mqtt.example.com:443` using a WebSocket-capable MQTT client (e.g., HiveMQ WebSocket client).
- **Command Line**:
  ```bash
  mosquitto_sub -h mqtt.example.com -p 8883 -t "test/topic" -u <username> -P <password> --tls-version tlsv1.2
  ```
  Note: Some clients (e.g., `mosquitto_sub`) may require the Let’s Encrypt CA certificate (`--cafile ca.crt`) if they don’t trust it by default.

### **9. Verify Logs**
- Check Traefik logs for certificate issuance and routing:
  ```bash
  docker logs traefik
  ```
- Check Mosquitto logs for client connections:
  ```bash
  docker logs mosquitto
  ```

---

## **Optional: Mutual TLS with Mosquitto**
If you need client certificate authentication (mutual TLS):
1. Configure Mosquitto to require client certificates by adding to `mosquitto.conf`:
   ```conf
   listener 1883
   cafile /mosquitto/config/certs/ca.crt
   certfile /mosquitto/config/certs/server.crt
   keyfile /mosquitto/config/certs/server.key
   require_certificate true
   use_identity_as_username true
   ```
2. Mount self-signed certificates into the Mosquitto container:
   ```yaml
   volumes:
     - ./mosquitto/config/certs:/mosquitto/config/certs
   ```
3. Generate client certificates and provide them to the ESP32 (as shown in the previous response).
4. Configure Traefik to pass through TLS traffic (`tls.passthrough=true`) to Mosquitto, as Traefik won’t terminate the client certificate authentication.

This approach requires Mosquitto to handle TLS, reducing Traefik’s role to routing, and is more complex due to certificate management.[](https://gist.github.com/natcl/ed8253a34e7b87d879baabeba82cb846)

---

## **Troubleshooting**
- **Certificate Errors**:
  - Ensure `mqtt.example.com` resolves to your server’s public IP.
  - Verify Let’s Encrypt certificate issuance in `traefik/acme.json`.
  - Use `openssl s_client -connect mqtt.example.com:8883` to inspect the presented certificate.
- **Connection Refused**:
  - Check that ports 80, 443, and 8883 are open in your firewall.
  - Ensure Mosquitto’s ports are not exposed directly (`expose` instead of `ports`).
  - Verify username/password in the `passwd` file.
- **SNI Issues**:
  - If clients don’t support SNI, ensure `HostSNI(`*`)` is used in Traefik’s TCP router.[](https://community.traefik.io/t/mosquitto-mqtt-and-tls-without-client-hostsni/21912)
- **WebSocket Issues**:
  - Confirm the WebSocket client uses `wss://` and port 443.
  - Check Traefik logs for WebSocket routing errors.[](https://medium.com/%40panda1100/mqtt-over-websocket-over-ssl-tls-behind-traefik-v2-6-with-lets-encrypt-cert-a88b19cb977)
- **Logs**:
  - Enable verbose logging in Mosquitto (`log_type all` in `mosquitto.conf`).
  - Use Traefik’s dashboard (add `--api.insecure=true` to `traefik.yml`) to inspect routing.

---

## **Best Practices**
- **Use Let’s Encrypt for Production**: Avoid self-signed certificates to simplify client configuration.[](https://medium.com/%40synoniem/running-mosquitto-mqtt-server-with-docker-traefik-and-lets-encrypt-a1f6cbb864cc)
- **Secure Passwords**: Use strong, unique passwords in Mosquitto’s `passwd` file.[](https://jurian.slui.mn/posts/smqttt-or-secure-mqtt-over-traefik/)
- **Network Isolation**: Place Traefik and Mosquitto in a dedicated Docker network (`mqtt`) to prevent direct access to Mosquitto.[](https://gist.github.com/gimiki/628e2ca10f026975f00f34e4d1f4ff23?permalink_comment_id=3463714)
- **Monitor Certificates**: Ensure Traefik renews Let’s Encrypt certificates automatically (stored in `acme.json`).
- **Backup `acme.json`**: Protect the Let’s Encrypt certificate store to avoid rate limit issues.
- **Update Regularly**: Keep Traefik and Mosquitto images up to date (`latest` or specific versions).

---

## **Example Sources**
This setup is inspired by community examples and best practices:
- A Docker Compose configuration for Mosquitto with Traefik and Let’s Encrypt: https://github.com/synoniem/mosquitto-traefik-letsencrypt[](https://gist.github.com/gimiki/628e2ca10f026975f00f34e4d1f4ff23?permalink_comment_id=3463714)
- Traefik documentation on TCP routing: https://docs.traefik.io/routing/routers/#rule_1
- Medium article on secure MQTT with Traefik: https://medium.com/himinds/mqtt-broker-with-secure-websocket-using-traefik-docker-compose-and-lets-encrypt-61441879f2f7[](https://medium.com/himinds/mqtt-broker-with-secure-websocket-using-traefik-docker-compose-and-lets-encrypt-2b8e32207555)

---

## **Conclusion**
Dockerizing Mosquitto with TLS through Traefik is not only possible but also a robust solution for securing MQTT communication. By leveraging Traefik’s TLS termination and Let’s Encrypt integration, you simplify certificate management and enhance scalability. The provided Docker Compose configuration supports both MQTT (port 8883) and WebSocket (port 443) with minimal Mosquitto configuration. For high-security use cases, mutual TLS can be added, though it requires Mosquitto to handle certificates directly.

If you need help with specific configurations, testing, or integrating with your ESP32, let me know!