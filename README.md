
# Spring-Reactive-Webflux-Api-Gateway

## Overview

This project is a **multi-microservice demonstration** of a **fully reactive API Gateway** built with **Spring Boot 3.x**, **Spring WebFlux**, and **Spring Cloud Gateway**. It showcases the power of the reactive stack (Project Reactor) for handling **high concurrency** with **non-blocking I/O**, low memory footprint, and backpressure support.

The gateway acts as a single entry point, routing requests reactively to downstream services while applying cross-cutting concerns like circuit breaking, rate limiting, and retries — all in a non-blocking manner.

## Real-World Scenario (Simulated)

In cloud API management platforms like **AWS API Gateway** or **Kong**:
- Millions of concurrent requests from mobile/web clients.
- Downstream services may have varying latency (DB calls, external APIs).
- Blocking threads would exhaust resources quickly.
- Need for resilience (circuit breakers), security (rate limiting), and observability.

We simulate a high-throughput e-commerce flow (order placement) where the gateway routes to reactive downstream services, handling thousands of concurrent requests efficiently using a small thread pool.

## Microservices Involved

| Service                  | Responsibility                                                                 | Port  |
|--------------------------|--------------------------------------------------------------------------------|-------|
| **eureka-server**        | Service discovery (Netflix Eureka)                                             | 8761  |
| **api-gateway**          | Reactive gateway: routing, filters, circuit breaker, rate limiting            | 8080  |
| **order-service**        | Reactive endpoint: orchestrates order creation (WebFlux)                       | 8081  |
| **inventory-service**    | Reactive: checks/reserves stock (simulated delay)                              | 8082  |
| **payment-service**      | Reactive: processes payment (simulated external call)                          | 8083  |

All services are built with **Spring WebFlux** (netty server, Mono/Flux).

## Tech Stack

- Spring Boot 3.x
- Spring WebFlux (functional + annotated controllers)
- Spring Cloud Gateway (reactive routes)
- Spring Cloud Circuit Breaker (Resilience4j reactive)
- Spring Cloud LoadBalancer (client-side LB with Eureka)
- Project Reactor (Mono/Flux)
- Spring Cloud Netflix Eureka
- Bucket4j or Resilience4j for rate limiting
- Micrometer + Actuator (reactive metrics)
- Lombok
- Maven (multi-module)
- Docker & Docker Compose

## Docker Containers

```yaml
services:
  eureka-server:
    build: ./eureka-server
    ports:
      - "8761:8761"

  api-gateway:
    build: ./api-gateway
    depends_on:
      - eureka-server
    ports:
      - "8080:8080"

  order-service:
    build: ./order-service
    depends_on:
      - eureka-server
    ports:
      - "8081:8081"

  inventory-service:
    build: ./inventory-service
    depends_on:
      - eureka-server
    ports:
      - "8082:8082"

  payment-service:
    build: ./payment-service
    depends_on:
      - eureka-server
    ports:
      - "8083:8083"
```

Run with: `docker-compose up --build`

## Reactive Gateway Features

| Feature                     | Implementation Details                                                  |
|-----------------------------|-------------------------------------------------------------------------|
| **Reactive Routing**        | RouteDefinition with `lb://service-name` (LoadBalancer)                 |
| **Circuit Breaker**         | Resilience4j reactive (`CircuitBreaker` filter on routes)              |
| **Rate Limiting**           | Custom filter with Bucket4j (token bucket) or Resilience4j RateLimiter  |
| **Retry**                   | Reactive `Retry` filter with backoff                                    |
| **Request/Response Modify**| Global filters for headers, logging, correlation ID                     |
| **Backpressure**            | Full Mono/Flux propagation — clients can push back                      |
| **WebClient**               | All inter-service calls via non-blocking WebClient                      |

## Key Features

- Fully non-blocking request processing
- High concurrency with small thread pool (Netty event loop)
- Resilience patterns: circuit breaker, retry, rate limit
- Service discovery integration with Eureka
- Custom global filters (correlation ID, logging)
- Reactive health checks and metrics
- Load testing ready (simulate 10k+ concurrent requests)

## Expected Endpoints

### API Gateway (`http://localhost:8080`)

| Method | Endpoint                        | Description                                      |
|--------|---------------------------------|--------------------------------------------------|
| POST   | `/orders`                       | Place order → routed to order-service            |
| GET    | `/inventory/{itemId}`           | Proxy to inventory-service                       |
| POST   | `/payments`                     | Proxy to payment-service                         |

### Downstream Services
- Order Service: orchestrates calls to inventory & payment via WebClient
- Simulated delays to show non-blocking behavior

### Actuator
- `/actuator/gateway/routes` — list active routes
- `/actuator/metrics` — reactive metrics (e.g., `reactor.netty`)

## Architecture Overview

```
Clients (high concurrency)
   ↓ (non-blocking)
API Gateway (Spring Cloud Gateway + Netty)
   ↓ (reactive routes + filters)
Eureka Discovery
   ↓
┌─────────────────────┬──────────────────────┐
│ inventory-service   │ payment-service      │
│ (WebFlux + Netty)   │ (WebFlux + Netty)    │
└─────────────────────┴──────────────────────┘
   ↓
Order Service (orchestrates via WebClient)
```

**Reactive Flow**:
1. Request → Gateway → Route with filters
2. LoadBalancer → WebClient (non-blocking) → downstream
3. Response streamed back (Mono/Flux)
4. Backpressure propagates end-to-end

## How to Run

1. Clone repository
2. Start Docker: `docker-compose up --build`
3. Access Eureka: `http://localhost:8761`
4. Place order via gateway: POST `/orders`
5. Observe logs → no thread blocking
6. Load test with tools like k6 or Gatling → see high throughput

## Testing Reactiveness

1. Normal request → fast
2. Add delay in payment-service (5s) → single request slow, but concurrent requests unaffected
3. Hammer with 1000 concurrent requests → low memory, high throughput
4. Trigger circuit breaker → fallback response
5. Exceed rate limit → 429 response

## Skills Demonstrated

- Full reactive stack: WebFlux, Gateway, WebClient
- Non-blocking inter-service communication
- Reactive resilience patterns (circuit breaker, retry)
- High-throughput API gateway design
- Backpressure and resource efficiency
- Reactive metrics and observability
- Functional routing in Gateway

## Future Extensions

- Reactive Redis for rate limiting/state
- Reactive database (R2DBC + PostgreSQL)
- WebSocket support through gateway
- OpenTelemetry tracing in reactive stack
- Kubernetes deployment with horizontal scaling
- Global request timeout and bulkhead
