# 📦 **Assignment: Supporting Microservices for a Scalable Backend Architecture**

## 🎯 Overview

In this assignment, you will build **three new microservices** that support the core functionality of your backend system. These microservices are:

1. **Shipping-Service** – Handles order shipping records
2. **Catalog-Service** – Provides a public-facing product catalog
3. **Audit-Service** – Centralized logging of important business events

These services are built **independently** from your main services (Product, Inventory, Order, Notification), but they fit into the same architecture — clean, isolated, containerized, and collaborative.

---

## 🧠 Learning Goals

* Apply **microservice isolation and responsibility separation**
* Build services using **Spring Boot with validation, DTOs, MongoDB Atlas, and clean architecture**
* Structure a Spring project with real-world maintainability
* Use **multi-stage Docker builds and Compose**
* Understand how services **communicate and complement each other** in a distributed system

---

## 🔧 Tools You’ll Use

* Java 17+
* Spring Boot
* Spring Web
* Spring Data MongoDB (except Catalog)
* Validation (Jakarta)
* Lombok
* Docker + Docker Compose
* MongoDB Atlas

---

## 📐 Standard Project Structure

Every microservice must follow this package layout:

```
org.example.<service>
├── config             # Future beans/configs
├── controller         # REST API layer
├── dto                # request/response records
│   ├── request
│   └── response
├── model              # Domain entities (MongoDB @Document)
├── repository         # MongoRepository interfaces
└── service
    └── impl           # Business logic implementations
```

---

# 🛠 STEP-BY-STEP GUIDE (Apply to each microservice)

---

### ✅ 1. Spring Initializer Setup

Go to [https://start.spring.io](https://start.spring.io) and configure the following:

* Project: **Maven**
* Language: **Java**
* Group: `org.example.<service-name>`
* Artifact: `shipping-service`, `catalog-service`, or `audit-service`
* Dependencies:

  * Spring Web
  * Spring Data MongoDB (skip for Catalog)
  * Lombok
  * Validation
  * Spring Boot DevTools *(optional)*

Click **Generate**, extract it, and import into your IDE.

---

### ✅ 2. Create the Package Structure

Manually create the following folders under `src/main/java/org.example.<your-service>`:

* `config`
* `controller`
* `dto/request`, `dto/response`
* `model`
* `repository`
* `service/impl`

Each class will be documented below with **starter code comments** (no logic).

---

# 🚚 Microservice 1: Shipping-Service

## 📚 Description

This service is called **after an order is placed** and stores shipping information (orderId + address). It logs the shipping action and stores the record in **MongoDB Atlas**.

---

## 🔖 Starter Classes

### `model/Shipping.java`

```java
/**
 * Represents a shipping record.
 */
@Document("shippings")
public class Shipping {

    @Id
    private String id;

    // Unique ID of the related order
    private String orderId;

    // Shipping address
    private String address;

    // Date and time the shipping was created
    private LocalDateTime shippedAt;

    // TODO: Add constructors, getters/setters, or use Lombok
}
```

---

### `dto/request/ShippingRequest.java`

```java
/**
 * Request to create a new shipping record.
 */
public record ShippingRequest(
    @NotBlank String orderId,
    @NotBlank String address
) {}
```

---

### `controller/ShippingController.java`

```java
/**
 * Controller exposing POST /api/shipping
 */
@RestController
@RequestMapping("/api/shipping")
@RequiredArgsConstructor
public class ShippingController {

    // TODO: Inject ShippingServiceImpl

    /**
     * Accepts a shipping request and saves it.
     */
    @PostMapping
    public void create(@Valid @RequestBody ShippingRequest request) {
        // TODO: Call shippingService.createShipping(request)
    }
}
```

---

### `service/impl/ShippingServiceImpl.java`

```java
/**
 * Business logic to handle shipping record creation.
 */
@Service
@RequiredArgsConstructor
public class ShippingServiceImpl {

    // TODO: Inject repository

    public void createShipping(ShippingRequest request) {
        // TODO: Map request to Shipping model
        // TODO: Set shippedAt to LocalDateTime.now()
        // TODO: Save to repository
        // TODO: Log to console
    }
}
```

---

# 🗂️ Microservice 2: Catalog-Service

## 📚 Description

This service exposes a frontend-friendly **read-only product catalog**. It fetches product info from **Product-Service** and optionally stock status from **Inventory-Service**.

---

### `dto/response/CatalogItemResponse.java`

```java
/**
 * A single catalog product with stock status.
 */
public record CatalogItemResponse(
    Long id,
    String name,
    String description,
    double price,
    boolean inStock
) {}
```

---

### `controller/CatalogController.java`

```java
/**
 * Controller exposing GET /api/catalog
 */
@RestController
@RequestMapping("/api/catalog")
@RequiredArgsConstructor
public class CatalogController {

    // TODO: Inject CatalogServiceImpl

    @GetMapping
    public List<CatalogItemResponse> getCatalog() {
        // TODO: Call service to fetch and return catalog data
        return null;
    }
}
```

---

### `service/impl/CatalogServiceImpl.java`

```java
/**
 * Business logic for fetching catalog data.
 */
@Service
@RequiredArgsConstructor
public class CatalogServiceImpl {

    // TODO: Use RestTemplate to call product-service and inventory-service

    public List<CatalogItemResponse> getCatalog() {
        // TODO: Call product-service to get product list
        // TODO: For each product, call inventory-service to get stock
        // TODO: Return a list of CatalogItemResponse
        return null;
    }
}
```

---

# 🧾 Microservice 3: Audit-Service

## 📚 Description

This service logs key system events (like order placed, shipping created) into **MongoDB Atlas**. It exposes both `POST /api/audit` and `GET /api/audit/logs`.

---

### `model/AuditLog.java`

```java
/**
 * A single audit log entry.
 */
@Document("logs")
public class AuditLog {

    @Id
    private String id;

    private String type;           // e.g. ORDER_PLACED
    private String sourceService;  // e.g. order-service
    private String message;

    private LocalDateTime timestamp;

    // TODO: Lombok or full boilerplate
}
```

---

### `dto/request/AuditRequest.java`

```java
/**
 * Request payload for creating a log entry.
 */
public record AuditRequest(
    @NotBlank String type,
    @NotBlank String sourceService,
    @NotBlank String message
) {}
```

---

### `controller/AuditController.java`

```java
/**
 * Exposes audit log endpoints.
 */
@RestController
@RequestMapping("/api/audit")
@RequiredArgsConstructor
public class AuditController {

    // TODO: Inject AuditServiceImpl

    @PostMapping
    public void createLog(@Valid @RequestBody AuditRequest request) {
        // TODO: Call auditService.logEvent(request)
    }

    @GetMapping("/logs")
    public List<AuditLog> getLogs() {
        // TODO: Call auditService.getAllLogs()
        return null;
    }
}
```

---

### `service/impl/AuditServiceImpl.java`

```java
/**
 * Handles persistence of audit logs.
 */
@Service
@RequiredArgsConstructor
public class AuditServiceImpl {

    // TODO: Inject AuditRepository

    public void logEvent(AuditRequest request) {
        // TODO: Map to AuditLog
        // TODO: Set timestamp
        // TODO: Save to MongoDB
    }

    public List<AuditLog> getAllLogs() {
        // TODO: Return all logs from repository
        return null;
    }
}
```

---

# 🐳 Dockerization Steps for All Microservices

---

### ✅ `.env` File

```
MONGO_URI=mongodb+srv://<user>:<pass>@cluster.mongodb.net/<db>?retryWrites=true&w=majority
```

---

### ✅ `Dockerfile` (Multi-stage)

```Dockerfile
# Build stage
FROM maven:3.9.4-eclipse-temurin-17 AS build
WORKDIR /app
COPY . .
RUN mvn clean package -DskipTests

# Runtime stage
FROM eclipse-temurin:17-jdk-alpine
WORKDIR /app
COPY --from=build /app/target/*.jar app.jar
ENTRYPOINT ["java", "-jar", "/app/app.jar"]
```

---

### ✅ `docker-compose.yml`

```yaml
version: '3.8'

services:
  <your-service-name>:
    build: .
    ports:
      - "PORT:PORT"
    environment:
      - SPRING_DATA_MONGODB_URI=${MONGO_URI}
      - SERVER_PORT=PORT
    networks:
      - micro-net

networks:
  micro-net:
    external: true
```

Replace `PORT` and `<your-service-name>` as needed.

---

## ✅ Submission Instructions

Each student must:

* Clone the starter structure
* Implement logic for all three microservices
* Use Postman or curl to test endpoints
* Run all services via Docker Compose
* Submit working code with clear structure


