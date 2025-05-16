---
title: Secure connection between ESP32 and a Mosquitto MQTT broker And how Certificate works
author:
  name: 
  link: 
date: 2025-04-30 10:35:00 -0400
categories: [Design]
tags: [qualtrics]
render_with_liquid: false
---

# Secure connection between ESP32 and a Mosquitto MQTT broker

To secure the connection between an **ESP32** and a **Mosquitto MQTT broker**, you need to implement encryption (TLS/SSL) and, optionally, authentication (username/password or client certificates). Below are the steps to achieve a secure connection:

---

### 1. **Set Up Mosquitto Broker with TLS/SSL**
Mosquitto needs to be configured to use TLS for encrypted communication.

#### a. **Obtain or Generate SSL/TLS Certificates**
- **Self-signed certificates** (for testing):
  - Use OpenSSL to generate a Certificate Authority (CA) certificate, server certificate, and private key.
  - Example commands:
    ```bash
    openssl req -new -x509 -days 365 -extensions v3_ca -keyout ca.key -out ca.crt
    openssl genrsa -out server.key 2048
    openssl req -new -out server.csr -key server.key
    openssl x509 -req -in server.csr -CA ca.crt -CAkey ca.key -CAcreateserial -out server.crt -days 365
    ```
- **Production**: Use a trusted CA (e.g., Let's Encrypt) to obtain certificates.

#### b. **Configure Mosquitto**
- Edit the Mosquitto configuration file (e.g., `/etc/mosquitto/mosquitto.conf`).
- Add the following to enable TLS:
  ```conf
  listener 8883
  cafile /path/to/ca.crt
  certfile /path/to/server.crt
  keyfile /path/to/server.key
  require_certificate false  # Set to true if using client certificates
  ```
- Optionally, enable username/password authentication:
  ```conf
  password_file /etc/mosquitto/passwd
  allow_anonymous false
  ```
- Create a password file:
  ```bash
  mosquitto_passwd -c /etc/mosquitto/passwd <username>
  ```
- Restart Mosquitto:
  ```bash
  sudo systemctl restart mosquitto
  ```

#### c. **Test Mosquitto Configuration**
- Use `mosquitto_sub` or `mosquitto_pub` with TLS:
  ```bash
  mosquitto_sub -h <broker_ip> -p 8883 --cafile ca.crt -t "test/topic" -u <username> -P <password>
  ```

---

### 2. **Configure ESP32 for Secure MQTT**
Use the **ESP-IDF** or **Arduino framework** to connect the ESP32 to the Mosquitto broker securely.

#### a. **Include Necessary Libraries**
- **Arduino**: Use the `PubSubClient` library for MQTT and `WiFiClientSecure` for TLS.
- **ESP-IDF**: Use the `esp-mqtt` component with TLS support.

#### b. **Load Certificates on ESP32**
- Embed the CA certificate (and optionally client certificate/key) in the ESP32 firmware.
- Example for Arduino:
  ```cpp
  const char* ca_cert = \
  "-----BEGIN CERTIFICATE-----\n" \
  "YOUR_CA_CERT_HERE\n" \
  "-----END CERTIFICATE-----\n";
  ```

#### c. **ESP32 Arduino Example Code**
Below is an example using the Arduino framework:

```cpp
#include <WiFi.h>
#include <WiFiClientSecure.h>
#include <PubSubClient.h>

// WiFi credentials
const char* ssid = "YOUR_WIFI_SSID";
const char* password = "YOUR_WIFI_PASSWORD";

// MQTT broker settings
const char* mqtt_server = "YOUR_BROKER_IP";
const int mqtt_port = 8883;
const char* mqtt_user = "YOUR_MQTT_USERNAME";
const char* mqtt_password = "YOUR_MQTT_PASSWORD";
const char* mqtt_client_id = "ESP32_Client";

// CA certificate
const char* ca_cert = \
"-----BEGIN CERTIFICATE-----\n" \
"YOUR_CA_CERT_HERE\n" \
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

  // Load CA certificate
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

#### d. **ESP-IDF Example**
For ESP-IDF, use the `esp-mqtt` component with TLS enabled:
1. Configure the MQTT client with the broker URL (`mqtts://<broker_ip>:8883`).
2. Provide the CA certificate in the `mqtt_config` structure:
   ```c
   esp_mqtt_client_config_t mqtt_cfg = {
       .broker.address.uri = "mqtts://YOUR_BROKER_IP",
       .broker.address.port = 8883,
       .broker.verification.certificate = ca_cert,
       .credentials.username = "YOUR_MQTT_USERNAME",
       .credentials.password = "YOUR_MQTT_PASSWORD",
   };
   ```
3. Initialize and start the MQTT client:
   ```c
   esp_mqtt_client_handle_t client = esp_mqtt_client_init(&mqtt_cfg);
   esp_mqtt_client_start(client);
   ```

---

### 3. **Optional: Client Certificate Authentication**
For stronger security, use client certificates instead of or in addition to username/password.

#### a. **Generate Client Certificates**
- Create a client key and certificate signed by the CA:
  ```bash
  openssl genrsa -out client.key 2048
  openssl req -new -out client.csr -key client.key
  openssl x509 -req -in client.csr -CA ca.crt -CAkey ca.key -CAcreateserial -out client.crt -days 365
  ```

#### b. **Update Mosquitto Configuration**
- Set `require_certificate true` in `mosquitto.conf`.
- Add the client certificate to the ESP32 firmware:
  ```cpp
  const char* client_cert = \
  "-----BEGIN CERTIFICATE-----\n" \
  "YOUR_CLIENT_CERT_HERE\n" \
  "-----END CERTIFICATE-----\n";

  const char* client_key = \
  "-----BEGIN PRIVATE KEY-----\n" \
  "YOUR_CLIENT_KEY_HERE\n" \
  "-----END PRIVATE KEY-----\n";
  ```
- In Arduino, set the client certificate and key:
  ```cpp
  espClient.setCertificate(client_cert);
  espClient.setPrivateKey(client_key);
  ```

#### c. **Update ESP32 Code**
- Ensure the client connects using the client certificate instead of or alongside username/password.

---

### 4. **Best Practices**
- **Use Strong Certificates**: For production, use certificates from a trusted CA.
- **Secure Certificate Storage**: Store certificates securely on the ESP32 (e.g., in SPIFFS or NVS).
- **Regularly Update Certificates**: Rotate certificates before they expire.
- **Enable Authentication**: Always use username/password or client certificates to prevent unauthorized access.
- **Firewall**: Restrict Mosquitto to listen only on necessary ports (e.g., 8883 for TLS).
- **Debugging**: Use `Serial` output on ESP32 and Mosquitto logs (`/var/log/mosquitto/`) to troubleshoot connection issues.

---

### 5. **Troubleshooting**
- **Connection Fails**: Verify the CA certificate, server IP, and port. Ensure the ESP32 has the correct time (use NTP for TLS).
- **Certificate Errors**: Check certificate validity and ensure the CA certificate is correctly embedded.
- **Authentication Issues**: Confirm username/password or client certificate settings match the broker configuration.
- **Logs**: Enable verbose logging in Mosquitto (`log_type all` in `mosquitto.conf`) for detailed debugging.

---







# More details about certifiate

To thoroughly explain how certificates work in the context of securing a connection between an **ESP32** and a **Mosquitto MQTT broker**, we’ll dive into the concepts of **TLS/SSL**, **Public Key Infrastructure (PKI)**, and the specific roles of certificate-related files (`cafile`, `certfile`, `keyfile`) and their formats (e.g., `.pem`, `.crt`, `.key`). I’ll also clarify why these files are necessary and how they’re used in the Mosquitto and ESP32 setup.

---

## **1. How Certificates Work in TLS/SSL**

**TLS (Transport Layer Security)**, the successor to SSL, is a cryptographic protocol used to secure communication over a network, such as between an ESP32 and a Mosquitto broker. Certificates are a core component of TLS, enabling **authentication**, **encryption**, and **data integrity**.

### **Key Concepts of TLS and Certificates**

#### a. **Public Key Infrastructure (PKI)**
PKI is the framework that manages digital certificates and public/private key pairs. It involves:
- **Certificates**: Digital documents that bind a public key to an identity (e.g., a server or client).
- **Certificate Authorities (CAs)**: Trusted entities that issue and sign certificates.
- **Public and Private Keys**: Asymmetric cryptography pairs where the public key encrypts data, and the private key decrypts it (or vice versa for signing).

#### b. **How TLS Works**
When an ESP32 connects to a Mosquitto broker using TLS:
1. **Handshake**:
   - The client (ESP32) and server (Mosquitto) negotiate a secure connection.
   - The server presents its **certificate** (containing its public key) to the client.
   - The client verifies the certificate’s authenticity using the **CA certificate**.
   - Optionally, the server may request a client certificate for mutual authentication.
2. **Key Exchange**:
   - The client and server agree on a symmetric session key (e.g., using Diffie-Hellman) for efficient encryption during the session.
3. **Secure Communication**:
   - Data is encrypted using the session key, ensuring confidentiality and integrity.

#### c. **Role of Certificates**
Certificates serve three main purposes:
- **Authentication**: Prove the identity of the server (or client, if mutual TLS is used).
- **Public Key Distribution**: Provide a trusted public key for encryption or signature verification.
- **Chain of Trust**: Link the certificate to a trusted CA, ensuring its legitimacy.

A certificate contains:
- The entity’s identity (e.g., server’s domain or IP).
- The public key.
- The CA’s signature, verifying the certificate’s authenticity.
- Metadata (e.g., validity period, issuer).

---

## **2. Certificate-Related Files in Mosquitto**

In the Mosquitto configuration, you specify three types of files: `cafile`, `certfile`, and `keyfile`. Each serves a distinct purpose in the TLS setup.

### a. **`cafile` (CA Certificate)**
- **What it is**: The **Certificate Authority (CA) certificate** contains the CA’s public key and identity. It is used to verify the authenticity of other certificates (e.g., the server’s certificate).
- **Role**:
  - The client (ESP32) uses the CA certificate to verify that the server’s certificate was signed by a trusted CA.
  - If the server requires client certificates (mutual TLS), the server uses the CA certificate to verify the client’s certificate.
- **Example**: `ca.crt` or `ca.pem`.
- **Content**: Contains the CA’s public key and metadata, not the private key.
- **Why needed**: Establishes the **chain of trust**. Without it, the client cannot confirm the server’s identity, and the connection may fail or be vulnerable to man-in-the-middle attacks.

### b. **`certfile` (Server or Client Certificate)**
- **What it is**: The **certificate** for the server (or client, in mutual TLS). It contains the entity’s public key and identity, signed by the CA.
- **Role**:
  - The server presents its certificate to the client during the TLS handshake to prove its identity.
  - In mutual TLS, the client presents its certificate to the server.
- **Example**: `server.crt`, `client.crt`, `server.pem`, or `client.pem`.
- **Content**: Includes the public key, the entity’s identity (e.g., server’s domain or IP), and the CA’s signature.
- **Why needed**: Authenticates the server (or client) to the other party. It’s the public-facing part of the entity’s identity.

### c. **`keyfile` (Private Key)**
- **What it is**: The **private key** corresponding to the certificate’s public key. It is kept secret and never shared.
- **Role**:
  - The server uses its private key to decrypt messages encrypted with its public key or to sign messages during the TLS handshake.
  - In mutual TLS, the client uses its private key for the same purposes.
- **Example**: `server.key`, `client.key`, `server.key.pem`.
- **Content**: Contains only the private key, in a format like RSA or ECDSA.
- **Why needed**: Enables the entity to prove ownership of the certificate and perform cryptographic operations (e.g., decrypting or signing).

---

## **3. File Formats (.pem, .crt, .key, etc.)**

The file extensions `.pem`, `.crt`, and `.key` indicate the **encoding** or **content type** of certificate-related files. These are not different types of certificates but different ways of storing the same cryptographic data.

### a. **Common File Formats**
- **`.pem` (Privacy-Enhanced Mail)**:
  - A versatile, text-based format that can store certificates, private keys, or both.
  - Uses Base64 encoding with headers like:
    - `-----BEGIN CERTIFICATE-----` for certificates.
    - `-----BEGIN PRIVATE KEY-----` or `-----BEGIN RSA PRIVATE KEY-----` for private keys.
  - Example content of a `.pem` file:
    ```pem
    -----BEGIN CERTIFICATE-----
    MII... (Base64-encoded certificate data)
    -----END CERTIFICATE-----
    ```
  - Usage: Can be used for `cafile`, `certfile`, or `keyfile`. A single `.pem` file may contain multiple certificates (e.g., a certificate chain).

- **`.crt` (Certificate)**:
  - Typically used for certificates (public key + metadata).
  - Can be in **PEM** (text, Base64) or **DER** (binary) format.
  - Often interchangeable with `.pem` for certificates.
  - Example: `server.crt` might be a PEM-encoded file identical to `server.pem`.

- **`.key`**:
  - Usually used for private keys.
  - Typically PEM-encoded but can be DER.
  - Example: `server.key` contains only the private key, e.g.:
    ```pem
    -----BEGIN PRIVATE KEY-----
    MII... (Base64-encoded private key data)
    -----END PRIVATE KEY-----
    ```

- **`.cer`**:
  - Similar to `.crt`, often used for certificates in PEM or DER format.
  - Commonly associated with Windows environments.

- **`.der` (Distinguished Encoding Rules)**:
  - A binary format for certificates or keys (not human-readable).
  - Less common in Mosquitto/ESP32 setups, as PEM is preferred for simplicity.

### b. **Why Different Extensions?**
- Extensions like `.pem`, `.crt`, and `.key` are **conventions** to indicate the file’s purpose or content:
  - `.pem` is generic and can hold certificates, keys, or both.
  - `.crt` typically indicates a certificate (public part).
  - `.key` indicates a private key.
- Tools like OpenSSL and Mosquitto are flexible and can handle these formats as long as the content is correctly formatted (e.g., PEM or DER).
- The choice of extension is often for clarity or compatibility with specific software.

### c. **Converting Between Formats**
You can convert between formats using OpenSSL:
- **PEM to DER** (certificate):
  ```bash
  openssl x509 -outform der -in server.pem -out server.der
  ```
- **DER to PEM**:
  ```bash
  openssl x509 -inform der -in server.der -out server.pem
  ```
- **Extract certificate from a `.pem` containing both certificate and key**:
  ```bash
  openssl x509 -in combined.pem -out certificate.crt
  ```

---

## **4. How Certificates Are Used in Mosquitto and ESP32**

### a. **Mosquitto Broker Configuration**
In the Mosquitto configuration (`mosquitto.conf`), you specify:
- **`cafile`**: Path to the CA certificate (e.g., `ca.crt` or `ca.pem`) to verify client certificates (if `require_certificate true`) or to establish trust for the server’s certificate.
- **`certfile`**: Path to the server’s certificate (e.g., `server.crt` or `server.pem`) that Mosquitto presents to clients.
- **`keyfile`**: Path to the server’s private key (e.g., `server.key`) used for cryptographic operations.

Example configuration:
```conf
listener 8883
cafile /path/to/ca.crt
certfile /path/to/server.crt
keyfile /path/to/server.key
require_certificate false  # Set to true for client certificate authentication
```

- During the TLS handshake:
  - Mosquitto sends `server.crt` to the ESP32.
  - The ESP32 uses `ca.crt` to verify `server.crt`.
  - Mosquitto uses `server.key` to prove it owns `server.crt`.

### b. **ESP32 Configuration**
On the ESP32:
- The **CA certificate** (`ca.crt` or `ca.pem`) is embedded in the firmware to verify the Mosquitto server’s certificate.
- If mutual TLS is enabled, the ESP32 also needs:
  - A **client certificate** (`client.crt` or `client.pem`).
  - A **client private key** (`client.key`).
- Example in Arduino:
  ```cpp
  const char* ca_cert = "-----BEGIN CERTIFICATE-----\nYOUR_CA_CERT_HERE\n-----END CERTIFICATE-----\n";
  const char* client_cert = "-----BEGIN CERTIFICATE-----\nYOUR_CLIENT_CERT_HERE\n-----END CERTIFICATE-----\n";
  const char* client_key = "-----BEGIN PRIVATE KEY-----\nYOUR_CLIENT_KEY_HERE\n-----END PRIVATE KEY-----\n";

  WiFiClientSecure espClient;
  espClient.setCACert(ca_cert);
  espClient.setCertificate(client_cert);  // Optional, for mutual TLS
  espClient.setPrivateKey(client_key);    // Optional, for mutual TLS
  ```

- During the TLS handshake:
  - The ESP32 verifies Mosquitto’s `server.crt` using `ca_cert`.
  - If mutual TLS is enabled, the ESP32 sends `client_cert` to Mosquitto, and Mosquitto verifies it using its `cafile`.

---

## **5. Why Different Files Are Needed**

Each file serves a specific role in the TLS handshake and PKI:
- **CA certificate (`cafile`)**: Establishes trust. Without it, the client cannot verify the server’s identity, making the connection vulnerable to impersonation.
- **Server certificate (`certfile`)**: Proves the server’s identity to the client. It’s the public part of the server’s identity.
- **Server private key (`keyfile`)**: Enables the server to authenticate itself and perform cryptographic operations. It’s the secret part that must be protected.
- **Client certificate and key** (optional): Used for mutual TLS to authenticate the ESP32 to the server, adding an extra layer of security.

The separation of these files ensures:
- **Security**: The private key is kept secret and never shared.
- **Modularity**: Certificates can be updated or replaced without affecting the private key.
- **Trust**: The CA certificate acts as a trusted anchor, linking all certificates in the chain.

---

## **6. Example Workflow: Generating and Using Certificates**

Here’s a practical example of generating certificates and configuring Mosquitto/ESP32:

### a. **Generate Certificates**
Using OpenSSL:
```bash
# Generate CA certificate and key
openssl req -new -x509 -days 365 -extensions v3_ca -keyout ca.key -out ca.crt

# Generate server key and certificate
openssl genrsa -out server.key 2048
openssl req -new -out server.csr -key server.key
openssl x509 -req -in server.csr -CA ca.crt -CAkey ca.key -CAcreateserial -out server.crt -days 365

# Generate client key and certificate (for mutual TLS)
openssl genrsa -out client.key 2048
openssl req -new -out client.csr -key client.key
openssl x509 -req -in client.csr -CA ca.crt -CAkey ca.key -CAcreateserial -out client.crt -days 365
```

### b. **Configure Mosquitto**
Add to `mosquitto.conf`:
```conf
listener 8883
cafile /path/to/ca.crt
certfile /path/to/server.crt
keyfile /path/to/server.key
require_certificate true  # For mutual TLS
```

### c. **Configure ESP32**
Embed certificates in the firmware (Arduino example):
```cpp
const char* ca_cert = R"(
-----BEGIN CERTIFICATE-----
YOUR_CA_CERT_HERE
-----END CERTIFICATE-----
)";

const char* client_cert = R"(
-----BEGIN CERTIFICATE-----
YOUR_CLIENT_CERT_HERE
-----END CERTIFICATE-----
)";

const char* client_key = R"(
-----BEGIN PRIVATE KEY-----
YOUR_CLIENT_KEY_HERE
-----END PRIVATE KEY-----
)";

WiFiClientSecure espClient;
espClient.setCACert(ca_cert);
espClient.setCertificate(client_cert);  // For mutual TLS
espClient.setPrivateKey(client_key);    // For mutual TLS
```

### d. **Test the Connection**
- Start Mosquitto: `sudo systemctl restart mosquitto`
- Run the ESP32 code and check for successful connection.
- Use `mosquitto_sub` to test:
  ```bash
  mosquitto_sub -h <broker_ip> -p 8883 --cafile ca.crt --cert client.crt --key client.key -t "test/topic"
  ```

---

## **7. Common Issues and Troubleshooting**

- **Certificate Verification Failure**:
  - Ensure the CA certificate matches the one used to sign the server/client certificates.
  - Check the certificate’s validity period (not expired).
  - Verify the ESP32 has the correct system time (use NTP for TLS).
- **File Format Issues**:
  - Confirm files are in PEM format (Mosquitto typically expects PEM).
  - Use `openssl x509 -in file.crt -text -noout` to inspect certificates.
- **Permission Errors**:
  - Ensure Mosquitto has read permissions for `cafile`, `certfile`, and `keyfile`.
  - Set secure permissions: `chmod 600 server.key`.
- **Mutual TLS Issues**:
  - Verify `require_certificate true` in Mosquitto if using client certificates.
  - Ensure the client certificate is signed by the same CA as the server’s certificate.

---

## **8. Best Practices**
- **Use Trusted CAs for Production**: Self-signed certificates are fine for testing but vulnerable to impersonation in production. Use Let’s Encrypt or another trusted CA.
- **Secure Private Keys**: Store `keyfile` and client keys securely (e.g., restrict permissions, use encrypted storage on ESP32).
- **Rotate Certificates**: Update certificates before they expire to avoid downtime.
- **Minimize CA Scope**: Use a dedicated CA for your MQTT setup to limit the impact of a compromised CA.
- **Enable Mutual TLS**: For high-security applications, use client certificates to authenticate ESP32 devices.

---




