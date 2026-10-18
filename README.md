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

## Architecture & Ecosystem Role

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

### Key Responsibilities:
1. **Dynamic Service Registration**: Downstream microservices (`authorization`, `gateway`, etc.) automatically announce their network locations and metadata on boot.
2. **Dynamic Client Routing**: The API Gateway discovers live instances dynamically and load balances requests via `lb://<service-name>` without static IPs or DNS entries.
3. **Continuous Liveness & Heartbeats**: Tracks instance health with periodic heartbeat signals (evicting unhealthy nodes when heartbeats expire).
4. **Standalone Mode**: Configured with `register-with-eureka: false` and `fetch-registry: false` to operate purely as the discovery registry in local development.

---

## Registered Microservices

| Service Application Name | Port | Description | Downstream Route (via Gateway) |
| :--- | :--- | :--- | :--- |
| `gateway` | `8085` | API Gateway & Reverse Proxy | Entry Point (`/api/v1/...`) |
| `authorization` | `8082` | Auth, JWT, User & Session Management | `/api/v1/auth/**` |

---

## Client Microservice Configuration

To register a microservice with this Eureka Server, configure its `application.yaml`:

```yaml
eureka:
  client:
    service-url:
      defaultZone: ${EUREKA_CLIENT_SERVICE_URL_DEFAULTZONE:http://localhost:8761/eureka/}
  instance:
    prefer-ip-address: true
    instance-id: ${spring.application.name}:${random.value}
```

---

## Directory Structure

```
eureka-server/
├── pom.xml
├── README.md
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
