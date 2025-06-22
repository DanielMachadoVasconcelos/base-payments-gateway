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
- [Testing](#-testing)
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

## 🧪 Testing

> ⚠️ **Note**: The automated tests for this project may encounter issues due to the complexity of testing Spring Cloud Gateway with Redis rate limiting in a test environment. Some tests are commented out because they require Redis to be running. To run these tests, uncomment them and ensure Redis is running. The manual testing procedures below provide a reliable way to verify the functionality.

### Automated Testing

The project includes automated tests organized by feature to verify the functionality of the gateway:

1. **Orders Feature Tests**:
   - **OrdersRouteTest**: Verifies that the orders route is correctly configured with the proper path predicate
   - **OrdersVersionHeaderTest**: Ensures that the version header is correctly added to requests to the orders endpoint
   - **OrdersRateLimitTest**: Tests the rate limiting functionality for the orders endpoint, including verification of the 429 status code when the rate limit is exceeded

2. **Products Feature Tests**:
   - **ProductsRouteTest**: Verifies that the products route is correctly configured with the proper path predicate
   - **ProductsVersionHeaderTest**: Ensures that the version header is correctly added to requests to the products endpoint
   - **ProductsRateLimitTest**: Tests the rate limiting functionality for the products endpoint, including verification of the 429 status code when the rate limit is exceeded

The tests use WireMock to simulate the backend services, allowing for comprehensive testing of the gateway's routing, header manipulation, and rate limiting capabilities without requiring external services.

### Manual Testing of Routes

You can also test the gateway routes manually using tools like cURL, Postman, or any HTTP client:

1. **Start the Gateway and Dependencies**:
   - Start Redis (automatically handled by Docker Compose)
   - Start Eureka Server (if available)
   - Start the Base Payments Service (or a mock service on the same port)
   - Start the Gateway application

2. **Test the Orders Route**:
   ```bash
   # Send a request to the orders endpoint
   curl -v http://localhost:9090/orders

   # Check that the request is routed to the BASE-PAYMENTS service
   # Verify that the 'version: 1.0.0' header is added
   ```

3. **Test the Products Route**:
   ```bash
   # Send a request to the products endpoint
   curl -v http://localhost:9090/products

   # Check that the request is routed to the BASE-PAYMENTS service
   # Verify that the 'version: 1.0.0' header is added
   ```

### Manual Testing of Rate Limiting

You can test the rate limiting functionality by sending multiple requests in quick succession:

1. **Test Rate Limiting**:
   ```bash
   # Send multiple requests to the orders endpoint
   for i in {1..25}; do
     curl -v http://localhost:9090/orders
     echo "Request $i completed"
     # No delay between requests to trigger rate limiting
   done
   ```

2. **Expected Behavior**:
   - The first 20 requests should succeed (due to burst capacity of 20)
   - Subsequent requests should receive a 429 Too Many Requests response
   - After waiting a few seconds, new requests should succeed again

3. **Verify Rate Limit Reset**:
   ```bash
   # Wait for rate limit to reset (about 2 seconds)
   sleep 2

   # Send another request, which should now succeed
   curl -v http://localhost:9090/orders
   ```

### Using Postman for Testing

Postman provides a more user-friendly interface for testing:

1. **Create a Collection** for gateway tests
2. **Add Requests** for each endpoint (/orders, /products)
3. **Use the Runner** to send multiple requests to test rate limiting
4. **Check Response Headers** to verify routing and rate limiting behavior

### Monitoring Rate Limiting

#### Using Application Logs

You can monitor rate limiting through the application logs:

```bash
# Tail the logs to see rate limiting in action
tail -f logs/application.log

# Look for entries like:
# "Requesting tokens from Redis rate limiter: 1"
# "Received token response: [true/false]"
```

> 💡 **Tip**: Set the logging level to DEBUG or TRACE in application.yml for more detailed rate limiting logs.

#### Using Actuator Endpoints

Spring Boot Actuator provides endpoints to monitor the application, including rate limiting:

1. **Gateway Routes**: View all configured routes
   ```
   http://localhost:9090/actuator/gateway/routes
   ```

2. **Gateway Global Filters**: View all global filters, including rate limiting
   ```
   http://localhost:9090/actuator/gateway/globalfilters
   ```

3. **Gateway Route Filters**: View filters for specific routes
   ```
   http://localhost:9090/actuator/gateway/routefilters
   ```

4. **Redis Health**: Check Redis connection status
   ```
   http://localhost:9090/actuator/health/redis
   ```

5. **Metrics**: Monitor rate limiter metrics
   ```
   http://localhost:9090/actuator/metrics/spring.cloud.gateway.requests
   ```

> 💡 **Tip**: You can use these endpoints to verify that the rate limiter is properly configured and functioning.

### Troubleshooting

If you encounter issues when testing:

#### Routes Not Working
- Verify that Eureka Server is running and the BASE-PAYMENTS service is registered
- Check the gateway logs for routing errors
- Ensure the BASE-PAYMENTS service is accessible directly
- Verify that the route configuration in application.yml is correct

#### Rate Limiting Not Working
- Verify that Redis is running (`docker ps` should show the Redis container)
- Check Redis connection in the logs
- Try restarting Redis (`docker-compose restart redis`)
- Ensure the rate limiter configuration in application.yml is correct
- Try with a different client IP to reset the rate limit counter

#### Common Errors
- **404 Not Found**: The route may be incorrect or the downstream service is not available
- **503 Service Unavailable**: The downstream service might be registered with Eureka but not actually running
- **429 Too Many Requests**: Rate limit exceeded (this is expected when testing rate limiting)
- **Connection Refused**: Redis or Eureka might not be running

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
