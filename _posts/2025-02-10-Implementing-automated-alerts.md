---
title: Implementing automated alerts 
author:
  name: 
  link: 
date: 2025-02-10 11:35:00 -0400
categories: [Alerts]
tags: [telgram, alert, springboot, email]
render_with_liquid: false
---
Implementing **automated alerts** for your IoT system (e.g., for garden irrigation, pool pump, or heater) is a great way to stay informed about system status, anomalies, or failures. Below, I’ll guide you through setting up **Telegram** and **email alerts** using **Spring Boot** as the backend.

---

### **1. Telegram Alerts**
Telegram is a popular choice for real-time notifications because of its simplicity and free API.

#### **Step 1: Create a Telegram Bot**
1. Open Telegram and search for the **BotFather**.
2. Use the `/newbot` command to create a new bot.
3. Follow the instructions to name your bot and get the **API token** (e.g., `123456789:ABCdefGhIJKlmNoPQRstuVWXyz`).

#### **Step 2: Get Your Chat ID**
1. Start a chat with your bot on Telegram.
2. Send a message (e.g., "Hello").
3. Use the following URL to get your chat ID:
   ```
   https://api.telegram.org/bot<YOUR_API_TOKEN>/getUpdates
   ```
   Replace `<YOUR_API_TOKEN>` with your bot’s token.
4. Look for the `chat.id` field in the JSON response.

#### **Step 3: Send Alerts from Spring Boot**
Use the Telegram Bot API to send messages from your Spring Boot backend.

##### **Dependency**:
Add the following to your `pom.xml`:
```xml
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-web</artifactId>
</dependency>
```

##### **Service Class**:
Create a `TelegramService` to send messages:
```java
import org.springframework.stereotype.Service;
import org.springframework.web.client.RestTemplate;

@Service
public class TelegramService {

    private static final String TELEGRAM_API_URL = "https://api.telegram.org/bot<YOUR_API_TOKEN>/sendMessage";
    private static final String CHAT_ID = "<YOUR_CHAT_ID>";

    private final RestTemplate restTemplate = new RestTemplate();

    public void sendAlert(String message) {
        String url = TELEGRAM_API_URL + "?chat_id=" + CHAT_ID + "&text=" + message;
        restTemplate.getForObject(url, String.class);
    }
}
```

##### **Usage**:
Call the `sendAlert` method whenever you need to send a notification:
```java
@Autowired
private TelegramService telegramService;

public void checkSoilMoisture() {
    if (soilMoisture < threshold) {
        telegramService.sendAlert("⚠️ Low soil moisture detected! Triggering irrigation.");
    }
}
```

---

### **2. Email Alerts**
Email alerts are useful for less urgent notifications or when Telegram is not an option.

#### **Step 1: Configure SMTP in Spring Boot**
Spring Boot makes it easy to send emails using JavaMail.

##### **Dependency**:
Add the following to your `pom.xml`:
```xml
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-mail</artifactId>
</dependency>
```

##### **Configuration**:
Add your SMTP server details to `application.properties`:
```properties
spring.mail.host=smtp.gmail.com
spring.mail.port=587
spring.mail.username=your-email@gmail.com
spring.mail.password=your-email-password
spring.mail.properties.mail.smtp.auth=true
spring.mail.properties.mail.smtp.starttls.enable=true
```

##### **Service Class**:
Create an `EmailService` to send emails:
```java
import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.mail.SimpleMailMessage;
import org.springframework.mail.javamail.JavaMailSender;
import org.springframework.stereotype.Service;

@Service
public class EmailService {

    @Autowired
    private JavaMailSender mailSender;

    public void sendAlert(String to, String subject, String text) {
        SimpleMailMessage message = new SimpleMailMessage();
        message.setTo(to);
        message.setSubject(subject);
        message.setText(text);
        mailSender.send(message);
    }
}
```

##### **Usage**:
Call the `sendAlert` method to send an email:
```java
@Autowired
private EmailService emailService;

public void checkPoolPump() {
    if (poolPumpStatus.equals("OFF")) {
        emailService.sendAlert(
            "your-email@gmail.com",
            "Pool Pump Alert",
            "⚠️ Pool pump is offline! Please check the system."
        );
    }
}
```

---

### **3. Combining Alerts**
You can combine Telegram and email alerts for redundancy or different levels of urgency. For example:
- Use **Telegram** for real-time, high-priority alerts.
- Use **Email** for daily summaries or less urgent notifications.

##### **Example**:
```java
public void handleAlert(String message, boolean isUrgent) {
    if (isUrgent) {
        telegramService.sendAlert(message);
    }
    emailService.sendAlert("your-email@gmail.com", "System Alert", message);
}
```

---

### **4. Advanced Features**
#### **a. Templated Emails**
Use **Thymeleaf** to send HTML emails with dynamic content.

##### **Dependency**:
```xml
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-thymeleaf</artifactId>
</dependency>
```

##### **Template**:
Create a `templates/alert-email.html` file:
```html
<!DOCTYPE html>
<html xmlns:th="http://www.thymeleaf.org">
<head>
    <title>Alert</title>
</head>
<body>
    <h1>⚠️ Alert</h1>
    <p th:text="${message}"></p>
</body>
</html>
```

##### **Service Class**:
```java
import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.mail.javamail.JavaMailSender;
import org.springframework.mail.javamail.MimeMessageHelper;
import org.springframework.stereotype.Service;
import org.thymeleaf.TemplateEngine;
import org.thymeleaf.context.Context;

import javax.mail.MessagingException;
import javax.mail.internet.MimeMessage;

@Service
public class EmailService {

    @Autowired
    private JavaMailSender mailSender;

    @Autowired
    private TemplateEngine templateEngine;

    public void sendHtmlAlert(String to, String subject, String message) throws MessagingException {
        MimeMessage mimeMessage = mailSender.createMimeMessage();
        MimeMessageHelper helper = new MimeMessageHelper(mimeMessage, true);

        Context context = new Context();
        context.setVariable("message", message);
        String htmlContent = templateEngine.process("alert-email", context);

        helper.setTo(to);
        helper.setSubject(subject);
        helper.setText(htmlContent, true);

        mailSender.send(mimeMessage);
    }
}
```

---

### **5. Testing**
- Test Telegram alerts by triggering a low soil moisture condition.
- Test email alerts by simulating a pool pump failure.

---

### **6. Final Notes**
- **Rate Limiting**: Avoid spamming users with too many alerts.
- **Error Handling**: Handle exceptions (e.g., network issues, invalid credentials).
- **Logging**: Log all alerts for auditing purposes.

Let me know if you need help with specific configurations or additional features! 😊


## Learn More

For more knowledge about Jekyll posts, visit the [Jekyll Docs: Posts](https://jekyllrb.com/docs/posts/).
