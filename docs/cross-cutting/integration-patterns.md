# Integration Patterns

## Overview

This document defines the integration patterns for the enterprise hybrid cloud architecture, including event-driven architecture, API architecture, and system integration approaches.

## Event-Driven Architecture

```mermaid
flowchart LR
    subgraph Producers["Event Producers"]
        P1[Order Service]
        P2[Customer Service]
        P3[Inventory Service]
        P4[IoT Devices]
    end
    
    subgraph EventBus["Event Bus (Pub/Sub)"]
        T1[orders-topic]
        T2[customers-topic]
        T3[inventory-topic]
        T4[iot-events-topic]
    end
    
    subgraph Consumers["Event Consumers"]
        C1[Analytics Service]
        C2[Notification Service]
        C3[Warehouse Service]
        C4[ML Pipeline]
    end
    
    subgraph Storage["Event Storage"]
        S1[(BigQuery)]
        S2[(Cloud Storage)]
    end
    
    P1 --> T1
    P2 --> T2
    P3 --> T3
    P4 --> T4
    
    T1 --> C1
    T1 --> C2
    T2 --> C1
    T2 --> C2
    T3 --> C3
    T3 --> C1
    T4 --> C4
    T4 --> C1
    
    C1 --> S1
    C4 --> S2
```

## API Architecture

```mermaid
flowchart TB
    subgraph Clients["API Clients"]
        CL1[Web Application]
        CL2[Mobile App]
        CL3[Partner Systems]
        CL4[Internal Services]
    end
    
    subgraph Gateway["API Gateway (Apigee)"]
        GW1[Authentication]
        GW2[Rate Limiting]
        GW3[Analytics]
        GW4[Transformation]
    end
    
    subgraph Products["API Products"]
        AP1[Customer API v1]
        AP2[Order API v1]
        AP3[Product API v1]
        AP4[Analytics API v1]
    end
    
    subgraph Backend["Backend Services"]
        BE1[Customer Service]
        BE2[Order Service]
        BE3[Product Service]
        BE4[Analytics Service]
    end
    
    Clients --> Gateway
    Gateway --> Products
    Products --> Backend
```

## Integration Patterns

### Pattern Catalog

| Pattern | Use Case | Technology |
|---------|----------|------------|
| **Request-Response** | Synchronous operations | REST, gRPC |
| **Publish-Subscribe** | Event distribution | Pub/Sub |
| **Event Sourcing** | Audit, replay | Pub/Sub, BigQuery |
| **Saga** | Distributed transactions | Pub/Sub, Workflows |
| **CQRS** | Read/write separation | Cloud SQL, BigQuery |
| **API Gateway** | External access | Apigee |
| **Service Mesh** | Service-to-service | Anthos Service Mesh |

### Request-Response Pattern

```mermaid
sequenceDiagram
    participant Client
    participant Gateway as API Gateway
    participant Service as Microservice
    participant DB as Database
    
    Client->>Gateway: POST /api/orders
    Gateway->>Gateway: Authenticate
    Gateway->>Service: Forward Request
    Service->>DB: Insert Order
    DB-->>Service: Confirmation
    Service-->>Gateway: Response
    Gateway-->>Client: 201 Created
```

### Publish-Subscribe Pattern

```mermaid
sequenceDiagram
    participant Producer
    participant Topic as Pub/Sub Topic
    participant Sub1 as Subscription 1
    participant Sub2 as Subscription 2
    participant Consumer1
    participant Consumer2
    
    Producer->>Topic: Publish Message
    Topic->>Sub1: Deliver Message
    Topic->>Sub2: Deliver Message
    Sub1->>Consumer1: Push Message
    Sub2->>Consumer2: Push Message
    Consumer1-->>Sub1: Acknowledge
    Consumer2-->>Sub2: Acknowledge
```

### Saga Pattern

```mermaid
flowchart LR
    subgraph Saga["Order Saga"]
        S1[Create Order]
        S2[Reserve Inventory]
        S3[Process Payment]
        S4[Ship Order]
    end
    
    subgraph Compensation["Compensation"]
        C1[Cancel Order]
        C2[Release Inventory]
        C3[Refund Payment]
    end
    
    S1 --> S2
    S2 --> S3
    S3 --> S4
    
    S2 -.->|Failure| C1
    S3 -.->|Failure| C2
    S4 -.->|Failure| C3
    C3 --> C2
    C2 --> C1
```

## Event Schema Standards

### Event Envelope

```json
{
  "eventId": "uuid",
  "eventType": "order.created",
  "eventTime": "2024-01-15T10:30:00Z",
  "source": "order-service",
  "dataContentType": "application/json",
  "data": {
    "orderId": "12345",
    "customerId": "C001",
    "items": []
  },
  "metadata": {
    "correlationId": "uuid",
    "causationId": "uuid"
  }
}
```

### Event Types

| Domain | Event Type | Description |
|--------|------------|-------------|
| **Order** | order.created | New order placed |
| **Order** | order.updated | Order modified |
| **Order** | order.completed | Order fulfilled |
| **Customer** | customer.registered | New registration |
| **Customer** | customer.updated | Profile updated |
| **Inventory** | inventory.reserved | Stock reserved |
| **Inventory** | inventory.released | Stock released |
| **Payment** | payment.processed | Payment completed |
| **Payment** | payment.failed | Payment failed |

## API Standards

### REST API Guidelines

| Guideline | Example |
|-----------|---------|
| Resource naming | `/api/v1/customers/{id}` |
| HTTP methods | GET, POST, PUT, PATCH, DELETE |
| Status codes | 200, 201, 400, 401, 404, 500 |
| Versioning | URL path (`/v1/`) |
| Pagination | `?page=1&size=20` |
| Filtering | `?status=active&type=premium` |

### API Security

| Layer | Implementation |
|-------|----------------|
| **Authentication** | OAuth 2.0, API Keys |
| **Authorization** | JWT claims, Scopes |
| **Rate Limiting** | Apigee policies |
| **Input Validation** | OpenAPI schema |
| **Transport** | TLS 1.3 |

## Service Mesh Configuration

### Anthos Service Mesh

```mermaid
flowchart TB
    subgraph Mesh["Service Mesh"]
        subgraph Service1["Service A"]
            A_POD[Pod]
            A_PROXY[Envoy Proxy]
        end
        
        subgraph Service2["Service B"]
            B_POD[Pod]
            B_PROXY[Envoy Proxy]
        end
        
        subgraph Service3["Service C"]
            C_POD[Pod]
            C_PROXY[Envoy Proxy]
        end
        
        CP[Control Plane]
    end
    
    A_PROXY <-->|mTLS| B_PROXY
    B_PROXY <-->|mTLS| C_PROXY
    A_PROXY <-->|mTLS| C_PROXY
    
    CP --> A_PROXY
    CP --> B_PROXY
    CP --> C_PROXY
```

### Traffic Management

| Feature | Use Case |
|---------|----------|
| **Traffic Splitting** | Canary deployments |
| **Retries** | Transient failures |
| **Timeouts** | Failure isolation |
| **Circuit Breaking** | Cascade prevention |
| **Fault Injection** | Chaos testing |

## Integration Monitoring

### Key Metrics

| Metric | Target | Alert Threshold |
|--------|--------|-----------------|
| API Latency (p95) | < 200ms | > 500ms |
| API Error Rate | < 1% | > 5% |
| Event Processing Lag | < 1s | > 30s |
| Message Throughput | N/A | -20% from baseline |
| Dead Letter Queue | 0 | > 100 |

---

[← Back to Main Documentation](../../README.md)
