# Vaultix Eureka Server - Architecture & Service Discovery

Technical guide detailing the service registration lifecycle, health tracking, client-side load balancing, and configuration guidelines for the **Vaultix Eureka Server**.

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
1. **Dynamic Service Registration**: Downstream microservices (`authorization`, `gateway`, `notification`, `demo`) automatically announce their network locations and metadata on boot.
2. **Dynamic Client Routing**: The API Gateway discovers live instances dynamically and load balances requests via `lb://<service-name>` without static IPs or DNS entries.
3. **Continuous Liveness & Heartbeats**: Tracks instance health with periodic heartbeat signals (evicting unhealthy nodes when heartbeats expire).
4. **Standalone Mode**: Configured with `register-with-eureka: false` and `fetch-registry: false` to operate purely as the discovery registry in local development.

---

## Registered Microservices Matrix

| Service Application Name | Port | Description | Downstream Route (via Gateway) |
| :--- | :--- | :--- | :--- |
| `gateway` | `8085` | API Gateway & Reverse Proxy | Entry Point (`/api/v1/...`) |
| `authorization` | `8082` | Auth, JWT, User & Session Management | `/api/v1/auth/**` |
| `demo` | `8083` | Sample Resource Server (RBAC & Revocation demo) | `/api/v1/demo/**` |
| `notification` | `8084` | Asynchronous Email & Notification Dispatch | `/api/v1/notifications/**` |

---

## Client Microservice Registration Guide

To register any Spring Boot microservice with this Eureka Server, add `spring-cloud-starter-netflix-eureka-client` to `pom.xml` and configure `application.yaml`:

```yaml
eureka:
  client:
    service-url:
      defaultZone: ${EUREKA_CLIENT_SERVICE_URL_DEFAULTZONE:http://localhost:8761/eureka/}
  instance:
    prefer-ip-address: true
    instance-id: ${spring.application.name}:${random.value}
```
