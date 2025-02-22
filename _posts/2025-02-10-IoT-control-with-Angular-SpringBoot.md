---
title: Full-stack IoT example
author:
  name: 
  link: 
date: 2025-02-10 10:35:00 -0400
categories: [Architecture]
tags: [architecture, angular, springboot, sse, docker, real-time, postgreSQL, prometheus, grafana]
render_with_liquid: false
---

> Okay, the user wants a full functional implementation example for their IoT system using Angular, Spring Boot, and Docker, along with how to measure performance and capacity. Let me start by breaking down their needs.
First, they need a complete example that integrates all the mentioned technologies. They’re working on a home automation project controlling garden irrigation, pool pump, lights, etc. 
The example should cover real-time updates using SSE, which I previously explained. They also want to know how to test the system's performance and capacity. 
I should outline the architecture again briefly, then dive into the code. For the backend, Spring Boot will handle SSE, device management, and alerts. The frontend in Angular will display real-time data. Docker will containerize the backend and database.
Including steps for each component: Spring Boot setup with SSE endpoints, Angular service and component for consuming SSE, Docker setup with PostgreSQL. Then, performance measurement using tools like JMeter, monitoring with Spring Boot Actuator and Prometheus/Grafana.
I need to make sure the code examples are accurate and functional. For example, the SseEmitter in Spring Boot, Angular’s EventSource, and Docker Compose configuration. Also, explain how to use JMeter for load testing and interpret results like throughput and error rates.
They might be looking for practical steps they can follow without much hassle, so clarity is key. Including commands for Docker, dependencies in pom.xml and package.json, and sample configurations will help them set things up quickly.
For performance measurement, explaining tools and metrics clearly will help them understand how to assess their system’s limits. Mentioning scalability tips like horizontal scaling and connection pooling is important for capacity planning.
I should also remind them about security aspects, even though the example is simplified. Maybe a note about adding authentication and using HTTPS in production.
Finally, summarize the example and emphasize the importance of monitoring and scaling for a reliable system. Make sure the tone is encouraging and offers further help if needed.

Let’s build a **full-stack example** of a real-time IoT monitoring system using **Angular**, **Spring Boot**, and **Docker**, with **Server-Sent Events (SSE)** for updates. We’ll also cover **performance testing** and **capacity measurement**. Here’s the step-by-step implementation:

---

### **1. System Overview**
- **Backend**: Spring Boot (Java) with SSE for real-time updates.
- **Frontend**: Angular for real-time dashboard.
- **Database**: PostgreSQL (via Docker).
- **Monitoring**: Prometheus + Grafana (for performance metrics).
- **Deployment**: Docker Compose.

---

### **2. Backend Implementation (Spring Boot)**

#### **Step 1: Project Setup**
Create a Spring Boot project with these dependencies (`pom.xml`):
```xml
<dependencies>
    <!-- Core -->
    <dependency>
        <groupId>org.springframework.boot</groupId>
        <artifactId>spring-boot-starter-web</artifactId>
    </dependency>
    <!-- Database -->
    <dependency>
        <groupId>org.springframework.boot</groupId>
        <artifactId>spring-boot-starter-data-jpa</artifactId>
    </dependency>
    <dependency>
        <groupId>org.postgresql</groupId>
        <artifactId>postgresql</artifactId>
        <scope>runtime</scope>
    </dependency>
    <!-- Monitoring -->
    <dependency>
        <groupId>org.springframework.boot</groupId>
        <artifactId>spring-boot-starter-actuator</artifactId>
    </dependency>
    <dependency>
        <groupId>io.micrometer</groupId>
        <artifactId>micrometer-registry-prometheus</artifactId>
    </dependency>
</dependencies>
```

#### **Step 2: SSE Controller**
```java
// SseController.java
import org.springframework.http.MediaType;
import org.springframework.web.bind.annotation.GetMapping;
import org.springframework.web.bind.annotation.RequestMapping;
import org.springframework.web.bind.annotation.RestController;
import org.springframework.web.servlet.mvc.method.annotation.SseEmitter;

import java.io.IOException;
import java.util.concurrent.CopyOnWriteArrayList;

@RestController
@RequestMapping("/api/sse")
public class SseController {

    private final CopyOnWriteArrayList<SseEmitter> emitters = new CopyOnWriteArrayList<>();

    @GetMapping(produces = MediaType.TEXT_EVENT_STREAM_VALUE)
    public SseEmitter subscribe() {
        SseEmitter emitter = new SseEmitter(Long.MAX_VALUE); // No timeout
        emitters.add(emitter);

        emitter.onCompletion(() -> emitters.remove(emitter));
        emitter.onTimeout(() -> emitters.remove(emitter));

        // Send initial message
        try {
            emitter.send(SseEmitter.event().data("Connected!"));
        } catch (IOException e) {
            emitter.completeWithError(e);
        }

        return emitter;
    }

    public void sendEvent(String data) {
        for (SseEmitter emitter : emitters) {
            try {
                emitter.send(SseEmitter.event().data(data));
            } catch (IOException e) {
                emitters.remove(emitter);
            }
        }
    }
}
```

#### **Step 3: Device Simulation Service**
Simulate IoT device data and send SSE updates:
```java
// DeviceSimulatorService.java
import org.springframework.scheduling.annotation.Scheduled;
import org.springframework.stereotype.Service;

@Service
public class DeviceSimulatorService {

    private final SseController sseController;

    public DeviceSimulatorService(SseController sseController) {
        this.sseController = sseController;
    }

    @Scheduled(fixedRate = 2000) // Send data every 2 seconds
    public void simulateDeviceData() {
        double temperature = Math.random() * 40; // Simulate temperature (0-40°C)
        String data = String.format("{\"temperature\": %.2f}", temperature);
        sseController.sendEvent(data);
    }
}
```

#### **Step 4: Configure PostgreSQL**
Add to `application.properties`:
```properties
spring.datasource.url=jdbc:postgresql://db:5432/iot
spring.datasource.username=postgres
spring.datasource.password=postgres
spring.jpa.hibernate.ddl-auto=update
```

---

### **3. Frontend Implementation (Angular)**

#### **Step 1: SSE Service**
```typescript
// sse.service.ts
import { Injectable } from '@angular/core';
import { Observable } from 'rxjs';

@Injectable({
  providedIn: 'root',
})
export class SseService {
  private eventSource: EventSource;

  constructor() {}

  getServerSentEvents(url: string): Observable<MessageEvent> {
    return new Observable((observer) => {
      this.eventSource = new EventSource(url);

      this.eventSource.onmessage = (event) => {
        observer.next(event);
      };

      this.eventSource.onerror = (error) => {
        observer.error(error);
      };
    });
  }

  closeConnection() {
    if (this.eventSource) {
      this.eventSource.close();
    }
  }
}
```

#### **Step 2: Dashboard Component**
```typescript
// dashboard.component.ts
import { Component, OnInit, OnDestroy } from '@angular/core';
import { SseService } from '../sse.service';
import { Subscription } from 'rxjs';

@Component({
  selector: 'app-dashboard',
  template: `
    <h1>Real-Time Temperature</h1>
    <div *ngIf="temperature !== null">
      Current Temperature: {{ temperature }}°C
    </div>
  `,
})
export class DashboardComponent implements OnInit, OnDestroy {
  temperature: number | null = null;
  private sseSubscription!: Subscription;

  constructor(private sseService: SseService) {}

  ngOnInit() {
    this.sseSubscription = this.sseService
      .getServerSentEvents('http://localhost:8080/api/sse')
      .subscribe({
        next: (event: MessageEvent) => {
          const data = JSON.parse(event.data);
          this.temperature = data.temperature;
        },
        error: (err) => console.error('SSE error:', err),
      });
  }

  ngOnDestroy() {
    this.sseSubscription.unsubscribe();
    this.sseService.closeConnection();
  }
}
```

---

### **4. Docker Setup**

#### **docker-compose.yml**
```yaml
version: '3.8'

services:
  backend:
    build: ./backend
    ports:
      - "8080:8080"
    environment:
      SPRING_DATASOURCE_URL: jdbc:postgresql://db:5432/iot
    depends_on:
      - db

  db:
    image: postgres:13
    environment:
      POSTGRES_USER: postgres
      POSTGRES_PASSWORD: postgres
      POSTGRES_DB: iot
    ports:
      - "5432:5432"
    volumes:
      - pgdata:/var/lib/postgresql/data

  prometheus:
    image: prom/prometheus
    ports:
      - "9090:9090"
    volumes:
      - ./prometheus.yml:/etc/prometheus/prometheus.yml

  grafana:
    image: grafana/grafana
    ports:
      - "3000:3000"

volumes:
  pgdata:
```

#### **Dockerfile (Backend)**
```dockerfile
FROM openjdk:17-jdk-slim
WORKDIR /app
COPY target/*.jar app.jar
ENTRYPOINT ["java", "-jar", "app.jar"]
```

---

### **5. Performance Testing**

#### **Tools**:
- **JMeter**: For load testing SSE connections.
- **Prometheus + Grafana**: For monitoring system metrics (CPU, memory, connections).
- **Spring Boot Actuator**: Exposes metrics endpoints.

#### **Step 1: Enable Actuator in Spring Boot**
Add to `application.properties`:
```properties
management.endpoints.web.exposure.include=health,metrics,prometheus
```

#### **Step 2: Configure Prometheus**
Create `prometheus.yml`:
```yaml
global:
  scrape_interval: 15s

scrape_configs:
  - job_name: 'spring-boot'
    metrics_path: '/actuator/prometheus'
    static_configs:
      - targets: ['backend:8080']
```

#### **Step 3: Run JMeter Test**
1. **Test Plan**:
   - Simulate 100 concurrent users connecting to `/api/sse`.
   - Measure **throughput**, **response time**, and **error rate**.

2. **Results**:
   - **Throughput**: Number of requests per second the system can handle.
   - **Latency**: Average response time for SSE connections.
   - **Error Rate**: Percentage of failed connections.

---

### **6. Capacity Measurement**
- **Connection Limit**: Test how many SSE connections your backend can handle (e.g., 10k connections).
- **Resource Usage**: Monitor CPU/memory usage under load (use `docker stats` or Grafana).
- **Database Throughput**: Measure PostgreSQL queries per second.

#### **Scaling Tips**:
- **Horizontal Scaling**: Deploy multiple backend instances behind a load balancer.
- **Connection Pooling**: Optimize PostgreSQL connection pooling with HikariCP.
- **Asynchronous Processing**: Use reactive programming (Spring WebFlux) for higher concurrency.

---

### **7. Run the System**
1. Build the Spring Boot app:
   ```bash
   cd backend && mvn clean package
   ```
2. Start the stack:
   ```bash
   docker-compose up --build
   ```
3. Access:
   - Angular: `http://localhost:4200`
   - Prometheus: `http://localhost:9090`
   - Grafana: `http://localhost:3000`

---

### **8. Final Notes**
- **Security**: Add JWT authentication for SSE endpoints (not shown here for simplicity).
- **Optimization**: Use `@Async` in Spring Boot to handle SSE emitters non-blockingly.
- **Alerting**: Configure Grafana alerts for high CPU/memory usage or connection drops.

This example gives you a foundation to build upon. Let me know if you need help with specific scenarios! 😊

## Learn More

For more knowledge about Jekyll posts, visit the [Jekyll Docs: Posts](https://jekyllrb.com/docs/posts/).
