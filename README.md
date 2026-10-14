# Vaultix Eureka Server

Service Registry & Discovery microservice for the **Vaultix** platform using Spring Cloud Netflix Eureka.

---

## Service Details

- **Group ID**: `com.vaultix`
- **Artifact ID**: `eureka-server`
- **Version**: `0.0.1`
- **Port**: `8761` (default Eureka port)
- **Dashboard URL**: [http://localhost:8761](http://localhost:8761)

---

## Purpose

The Eureka Server acts as the centralized phonebook and service registry for all Vaultix microservices:
- Microservices (`authorization`, `api-gateway`, etc.) register their network locations upon startup.
- Services discover each other dynamically without hardcoded hostnames or IPs.
- Continuous heartbeat monitoring checks service health and deregisters unhealthy instances.

---

## Running Locally

### Build
```powershell
.\mvnw.cmd clean package
```

### Run
```powershell
.\mvnw.cmd spring-boot:run
```
Once started, open the Eureka dashboard at [http://localhost:8761](http://localhost:8761).
