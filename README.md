# Vaultix Eureka Server

[![Java 27](https://img.shields.io/badge/Java-27-ED8B00?style=flat&logo=openjdk&logoColor=white)](https://openjdk.org/)
[![Spring Boot 4.1.2](https://img.shields.io/badge/Spring_Boot-4.1.2-6DB33F?style=flat&logo=springboot&logoColor=white)](https://spring.io/projects/spring-boot)
[![Spring Cloud 2025.1.4](https://img.shields.io/badge/Spring_Cloud-2025.1.4-6DB33F?style=flat&logo=spring&logoColor=white)](https://spring.io/projects/spring-cloud)
[![Virtual Threads](https://img.shields.io/badge/Virtual_Threads-Enabled-blue?style=flat)](https://openjdk.org/jeps/444)
[![License: Apache 2.0](https://img.shields.io/badge/License-Apache_2.0-blue.svg)](https://opensource.org/licenses/Apache-2.0)

Centralized **Service Registry & Dynamic Discovery Hub** for the **Vaultix** distributed microservices platform, powered by **Spring Cloud Netflix Eureka Server** on **Java 27 Virtual Threads** (Project Loom).

---

## Service Specifications

| Attribute | Specification |
| :--- | :--- |
| **Port** | `8761` (Standard Eureka service port) |
| **Web Dashboard** | [http://localhost:8761](http://localhost:8761) |
| **REST Endpoints** | [http://localhost:8761/eureka/apps](http://localhost:8761/eureka/apps) |
| **Framework** | Spring Boot `4.1.2` / Spring Cloud `2025.1.4` (`spring-cloud-starter-netflix-eureka-server`) |
| **Runtime** | Java 27 (OpenJDK / Amazon Corretto 27) with Virtual Threads enabled |
| **Concurrency Engine** | High-throughput non-blocking Virtual Threads (`spring.threads.virtual.enabled: true`) |
| **Deployment Mode** | Standalone registry (self-registration disabled; peer replication ready) |

---

## Architectural Data Flow

The Eureka Server acts as the centralized phonebook of the platform. Downstream microservices register on boot, periodically refresh their leases with heartbeats, and allow the API Gateway to route requests dynamically:

```mermaid
flowchart TD
    subgraph Registry ["Eureka Server (Port 8761)"]
        REG["Dynamic Registry Table"]
        LIVENESS["Liveness Tracker & Eviction Timer"]
        CONSOLE["Web Dashboard UI"]
    end

    subgraph Edge ["Edge Layer"]
        GW["Vaultix Gateway (Port 8085)"]
    end

    subgraph Services ["Core Services"]
        AUTH["Vaultix Authorization (Port 8082)"]
        DEMO["Vaultix Demo (Port 8083)"]
        NOTIF["Vaultix Notification (Port 8084)"]
    end

    AUTH -->|Register & Heartbeat (30s)| Registry
    DEMO -->|Register & Heartbeat (30s)| Registry
    NOTIF -->|Register & Heartbeat (30s)| Registry
    GW -->|Fetch Registry & Heartbeat| Registry

    GW -.->|Route via lb://authorization| AUTH
    GW -.->|Route via lb://demo| DEMO
    GW -.->|Route via lb://notification| NOTIF
```

---

## Directory Structure

```text
eureka-server/
├── pom.xml                                    /* Spring Boot 4.1.2 & Spring Cloud 2025.1.4 BOM */
├── README.md                                  /* Service overview and operational runbook */
├── docs/
│   └── ARCHITECTURE.md                        /* Deep-dive architecture and discovery manual */
└── src/main/
    ├── java/com/vaultix/eureka_server/
    │   └── EurekaServerApplication.java       /* @EnableEurekaServer bootstrap entry point */
    └── resources/
        └── application.yaml                   /* Virtual threads, standalone registry settings */
```

---

## Configuration Reference (`application.yaml`)

```yaml
server:
  port: 8761

spring:
  application:
    name: eureka-server
  threads:
    virtual:
      enabled: true

eureka:
  client:
    register-with-eureka: false
    fetch-registry: false
```

### Key Configuration Rationale:
1. **`spring.threads.virtual.enabled: true`**: Harnesses Java 27 lightweight virtual threads to serve high-frequency client heartbeats (`/eureka/apps/*`) with near-zero memory footprint and no thread pool exhaustion.
2. **`eureka.client.register-with-eureka: false`**: Prevents the standalone registry from registering itself as a discoverable application node.
3. **`eureka.client.fetch-registry: false`**: Disables local registry caching on the server itself, as it maintains the authoritative state in memory.

---

## Registered Microservices Matrix

| Service Application Name | Default Port | Primary Responsibilities | Gateway Route Pattern |
| :--- | :---: | :--- | :--- |
| **`gateway`** | `8085` | API Gateway, TLS termination, edge rate limiting, context injection | Entry Point (`/api/v1/...`) |
| **`authorization`** | `8082` | Identity management, OAuth2 resource server, WebAuthn, JWT issuance, Sudo sessions | `/api/v1/auth/**` |
| **`demo`** | `8083` | Demonstration resource server, RBAC verification, token revocation testing | `/api/v1/demo/**` |
| **`notification`** | `8084` | Asynchronous email, SMS, and template dispatching via RabbitMQ | `/api/v1/notifications/**` |

---

## Quickstart & Local Execution

### 1. Build
```powershell
.\mvnw.cmd clean package
```

### 2. Run
```powershell
.\mvnw.cmd spring-boot:run
```

Once running, access the web console at [http://localhost:8761](http://localhost:8761) to inspect all currently registered instances, lease renewal statuses, and uptime metrics.

---

## In-Depth Documentation

For complete technical specifications, lifecycle flows, self-preservation tuning, and REST API documentation, see:

👉 [**Architecture & Service Discovery Deep-Dive (docs/ARCHITECTURE.md)**](docs/ARCHITECTURE.md)
