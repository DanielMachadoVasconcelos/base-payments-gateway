# 🚪 Base Payments Gateway

[![Spring Boot](https://img.shields.io/badge/Spring%20Boot-3.5.0-brightgreen.svg)](https://spring.io/projects/spring-boot)
[![Java](https://img.shields.io/badge/Java-21-orange.svg)](https://www.oracle.com/java/)
[![Spring Cloud](https://img.shields.io/badge/Spring%20Cloud-2025.0.0-blue.svg)](https://spring.io/projects/spring-cloud)
[![License](https://img.shields.io/badge/License-MIT-yellow.svg)](LICENSE)

> A Spring Cloud Gateway Server WebFlux application that routes API requests to the [Base Payments Service](https://github.com/DanielMachadoVasconcelos/base-payments).

---

## 📋 Table of Contents

- [Overview](#-overview)
- [Tech Stack](#-tech-stack)
- [Configuration](#-configuration)
- [Running Locally](#-running-locally)
- [Monitoring and Management](#-monitoring-and-management)
- [Service Integration](#-service-integration)
- [Logging](#-logging)

---

## 🔍 Overview

This gateway serves as an API gateway for the Base Payments microservice architecture. It handles routing, load balancing, and provides a single entry point for client applications to interact with the underlying services.

![API Gateway Pattern](https://microservices.io/i/apigateway.jpg)

> 💡 **API Gateway Pattern**: Acts as a single entry point for all clients, providing a unified interface to the microservices.

---

## 🛠️ Tech Stack

| Technology | Version | Purpose |
|------------|---------|---------|
| [Java](https://www.oracle.com/java/) | 21 | Programming Language |
| [Spring Boot](https://spring.io/projects/spring-boot) | 3.5.0 | Application Framework |
| [Spring Cloud](https://spring.io/projects/spring-cloud) | 2025.0.0 | Cloud Native Development |
| [Spring Cloud Gateway Server WebFlux](https://spring.io/projects/spring-cloud-gateway) | - | API Gateway |
| [Spring Cloud Netflix Eureka Client](https://spring.io/projects/spring-cloud-netflix) | - | Service Discovery |
| [Spring Cloud LoadBalancer](https://spring.io/guides/gs/spring-cloud-loadbalancer/) | - | Load Balancing |
| [Spring Boot Actuator](https://docs.spring.io/spring-boot/docs/current/reference/html/actuator.html) | - | Monitoring & Management |
| [Spring Boot WebFlux](https://docs.spring.io/spring-framework/reference/web/webflux.html) | - | Reactive Programming |
| [Redis](https://redis.io/) | - | Rate Limiting & Caching |
| [Docker Compose](https://docs.docker.com/compose/) | - | Container Orchestration |

---

## ⚙️ Configuration

### 🔌 Port
The application runs on port `9090`

### 🔍 Service Discovery
Uses [Eureka](https://github.com/Netflix/eureka) for service discovery

### 🛣️ Routes

| Path | Target Service | Additional Headers | Rate Limiting |
|------|---------------|-------------------|--------------|
| `/orders/**` | BASE-PAYMENTS | `version: 1.0.0` | 10 req/sec, burst: 20 |
| `/products/**` | BASE-PAYMENTS | `version: 1.0.0` | 10 req/sec, burst: 20 |

### 🚦 Rate Limiting

The application uses Redis-based rate limiting to protect the API from excessive requests:

- **Rate**: 10 requests per second
- **Burst Capacity**: 20 requests
- **Key Resolver**: Client IP address (each IP has its own rate limit)
- **Redis**: Required for storing rate limit counters
- **Response**: Returns HTTP 429 (Too Many Requests) status code when rate limit is exceeded
- **Empty Response**: Configured to return a response body with the status code

### 🐳 Docker Compose Integration

The project includes Docker Compose integration for easy setup of required services:

- **Redis**: Automatically starts a Redis container for rate limiting
- **Spring Boot Docker Compose**: Automatically manages container lifecycle
- **Volume Persistence**: Redis data is persisted across restarts

> 💡 **How it works**: When you run the application with `./gradlew bootRun`, Spring Boot automatically detects the `docker-compose.yml` file and starts the defined services before the application starts.

---

## 🚀 Running Locally

### Prerequisites

✅ Java 21 or higher  
✅ Gradle  
✅ Eureka Server running on port 8761  
✅ Docker and Docker Compose (optional, for automatic Redis setup)

> 💡 **Note**: If you don't have Docker installed, you'll need Redis Server running on port 6379 for rate limiting.

### Steps to Run

1️⃣ **Clone the repository**:
```bash
git clone [repository-url]
cd base-payments-gateway
```

2️⃣ **Build the application**:
```bash
./gradlew build
```

3️⃣ **Run the application**:
```bash
./gradlew bootRun
```

> 💡 **Alternatively**, you can run the JAR file directly:
> ```bash
> java -jar build/libs/base-payments-gateway-0.0.1-SNAPSHOT.jar
> ```

4️⃣ The application will be available at `http://localhost:9090`

---

## 📊 Monitoring and Management

The application exposes various [Spring Boot Actuator](https://docs.spring.io/spring-boot/docs/current/reference/html/actuator.html) endpoints for monitoring and management:

| Endpoint | URL | Description |
|----------|-----|-------------|
| Health Check | [http://localhost:9090/actuator/health](http://localhost:9090/actuator/health) | Shows application health information |
| Info | [http://localhost:9090/actuator/info](http://localhost:9090/actuator/info) | Displays application information |
| All Actuator Endpoints | [http://localhost:9090/actuator](http://localhost:9090/actuator) | Lists all available actuator endpoints |

---

## 🔄 Service Integration

This gateway routes requests to the [Base Payments Service](https://github.com/DanielMachadoVasconcelos/base-payments) using:

- ✅ **Eureka** for service discovery
- ✅ **Load Balancing** for distributing traffic

![Service Discovery Pattern](https://microservices.io/i/servicediscovery/discovery-problem.jpg)

---

## 📝 Logging

The application has detailed logging enabled for:

- 📋 Spring Cloud Gateway Server WebFlux
- 📋 Spring Web
- 📋 Netflix Discovery
- 📋 Spring Cloud Client Discovery

> ⚠️ **Note**: Detailed logging helps in debugging and monitoring the gateway's behavior, but may impact performance in production environments.

---

## 📚 Additional Resources

- [Spring Cloud Gateway Server WebFlux Documentation](https://docs.spring.io/spring-cloud-gateway/docs/current/reference/html/)
- [Spring Boot Documentation](https://docs.spring.io/spring-boot/docs/current/reference/html/)
- [Microservices.io - API Gateway Pattern](https://microservices.io/patterns/apigateway.html)
- [Microservices.io - Service Discovery Pattern](https://microservices.io/patterns/service-registry.html)

---

## 📄 License

This project is licensed under the MIT License - see the LICENSE file for details.
