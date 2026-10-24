# Vaultix Eureka Server

Centralized **Service Registry & Discovery** server for the **Vaultix** microservices platform, powered by **Spring Cloud Netflix Eureka** on **Java 27 Virtual Threads** (Project Loom).

---

## Service Overview

| Attribute | Value |
| :--- | :--- |
| **Port** | `8761` (Standard Eureka server port) |
| **Dashboard URL** | [http://localhost:8761](http://localhost:8761) |
| **Framework** | Spring Boot 4.1.1 + Spring Cloud 2025.1.3 (`spring-cloud-starter-netflix-eureka-server`) |
| **Runtime** | Java 27 (Amazon Corretto) with Virtual Threads enabled |
| **Role** | Standalone discovery hub (self-preservation & client-side load balancing) |

---

## Documentation Index

Detailed service discovery architecture, load balancing mechanics, and client registration guides are maintained in the [`docs/`](docs/) directory:

| Document | Description |
| :--- | :--- |
| 📖 [**Architecture & Service Discovery**](docs/ARCHITECTURE.md) | In-depth breakdown of dynamic service registration, client-side load balancing (`lb://`), liveness tracking & heartbeats, registered microservices matrix, and client registration guidelines. |

---

## Architectural Data Flow

```
                     ┌─────────────────────────┐
                     │   Eureka Server (8761)  │
                     │    Service Registry     │
                     └───────▲─────────▲───────┘
                             │         │
               Register &    │         │  Register &
               Heartbeat     │         │  Heartbeat
                             │         │
               ┌─────────────┴──┐   ┌──┴─────────────┐
               │    Gateway     │   │ Authorization  │
               │     (8085)     │──►│     (8082)     │
               └────────────────┘   └────────────────┘
                     Routes via lb://authorization
```

---

## Directory Structure

```
eureka-server/
├── pom.xml
├── README.md
├── docs/
│   └── ARCHITECTURE.md
└── src/main/
    ├── java/com/vaultix/eureka_server/
    │   └── EurekaServerApplication.java
    └── resources/
        └── application.yaml
```

---

## Quick Start

### 1. Build
```powershell
.\mvnw.cmd clean package
```

### 2. Run
```powershell
.\mvnw.cmd spring-boot:run
```

Once started, open the web console at [http://localhost:8761](http://localhost:8761) to monitor registered microservice instances.
