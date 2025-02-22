---
title: How to Secure API 
author:
  name: 
  link: 
date: 2025-02-10 22:35:00 -0400
categories: [DEV]
tags: [api, security]
render_with_liquid: false
---

Protecting your API to ensure it can only be accessed by your Single Page Application (SPA) is a common challenge, as SPAs run on the client side and cannot securely store secrets like API keys. However, there are several strategies you can implement to enhance the security of your API and restrict access to only your SPA:

---

### **1. Use CORS (Cross-Origin Resource Sharing)**
- **What it does**: CORS allows you to specify which domains are allowed to access your API.
- **How to implement**:
  - Configure your API to only accept requests from the domain where your SPA is hosted.
  - Example (in Express.js):
    ```javascript
    const cors = require('cors');
    app.use(cors({
      origin: 'https://your-spa-domain.com'
    }));
    ```
- **Limitations**: CORS can be bypassed if someone manually makes requests from a non-browser environment (e.g., Postman or cURL).

---

### **2. Use API Keys**
- **What it does**: Require an API key to access your API.
- **How to implement**:
  - Generate a unique API key for your SPA.
  - Include the API key in the headers of every request from your SPA.
  - Validate the API key on the server side.
  - Example (in Express.js):
    ```javascript
    app.use((req, res, next) => {
      const apiKey = req.headers['x-api-key'];
      if (apiKey !== 'your-secret-api-key') {
        return res.status(403).json({ error: 'Unauthorized' });
      }
      next();
    });
    ```
- **Limitations**: API keys exposed in client-side code can be easily extracted and misused.

---

### **3. Use OAuth 2.0 with PKCE (Proof Key for Code Exchange)**
- **What it does**: OAuth 2.0 with PKCE is a secure way to authenticate SPAs without exposing secrets.
- **How to implement**:
  - Use an OAuth 2.0 provider (e.g., Auth0, Okta, or your own implementation).
  - Implement the PKCE flow in your SPA to obtain an access token.
  - Include the access token in the `Authorization` header of API requests.
  - Validate the token on the server side.
- **Advantages**: More secure than API keys, as tokens are short-lived and can be scoped to specific permissions.

---

### **4. Use a Backend-for-Frontend (BFF) Pattern**
- **What it does**: Introduce a lightweight backend server that acts as a proxy between your SPA and API.
- **How to implement**:
  - Host your SPA and BFF on the same domain.
  - The SPA makes requests to the BFF, which then forwards them to the API.
  - The BFF can securely store and manage API keys or tokens.
  - Example: Use Node.js or a serverless function as the BFF.
- **Advantages**: Keeps sensitive credentials out of the client-side code.

---

### **5. Use HTTPS**
- **What it does**: Encrypts data in transit to prevent eavesdropping and tampering.
- **How to implement**:
  - Ensure your API and SPA are served over HTTPS.
  - Use TLS certificates from a trusted Certificate Authority (CA).

---

### **6. Validate Referer and Origin Headers**
- **What it does**: Check the `Referer` and `Origin` headers to ensure requests are coming from your SPA.
- **How to implement**:
  - Example (in Express.js):
    ```javascript
    app.use((req, res, next) => {
      const referer = req.headers['referer'];
      const origin = req.headers['origin'];
      if (!referer?.startsWith('https://your-spa-domain.com') && !origin?.startsWith('https://your-spa-domain.com')) {
        return res.status(403).json({ error: 'Unauthorized' });
      }
      next();
    });
    ```
- **Limitations**: These headers can be spoofed, so this should not be your only line of defense.

---

### **7. Use Rate Limiting**
- **What it does**: Prevents abuse by limiting the number of requests from a single client.
- **How to implement**:
  - Use a library like `express-rate-limit` (for Node.js) or configure rate limiting in your API gateway.
  - Example (in Express.js):
    ```javascript
    const rateLimit = require('express-rate-limit');
    const limiter = rateLimit({
      windowMs: 15 * 60 * 1000, // 15 minutes
      max: 100 // limit each IP to 100 requests per windowMs
    });
    app.use(limiter);
    ```

---

### **8. Use Signed Requests**
- **What it does**: Sign requests with a secret key to ensure they originate from your SPA.
- **How to implement**:
  - Generate a signature on the client side using a secret key (e.g., HMAC).
  - Include the signature in the request headers.
  - Validate the signature on the server side.
- **Limitations**: The secret key must be embedded in the SPA, which can still be extracted.

---

### **9. Use a Web Application Firewall (WAF)**
- **What it does**: Protects your API from common attacks like SQL injection, XSS, and DDoS.
- **How to implement**:
  - Use a cloud-based WAF (e.g., AWS WAF, Cloudflare) or a self-hosted solution.

---

### **10. Monitor and Log Requests**
- **What it does**: Helps you detect and respond to suspicious activity.
- **How to implement**:
  - Log all incoming requests to your API.
  - Use tools like ELK Stack (Elasticsearch, Logstash, Kibana) or cloud-based monitoring services.

---

### **Best Practices Summary**
- Use **CORS** to restrict access to your SPA's domain.
- Use **OAuth 2.0 with PKCE** for secure authentication.
- Consider a **BFF pattern** to keep credentials secure.
- Always use **HTTPS** to encrypt data in transit.
- Implement **rate limiting** and **monitoring** to detect and prevent abuse.

By combining these strategies, you can significantly improve the security of your API and ensure it is only accessible by your SPA.


