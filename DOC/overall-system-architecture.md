# Overall System Architecture

```mermaid
graph TD
    subgraph Clients
        Web[Web Application]
        Mobile[Mobile App]
        ThirdParty[Third-Party Integrations]
    end

    subgraph Edge Layer
        CDN[CDN]
        LB[Load Balancer]
        WAF[Web Application Firewall]
    end

    subgraph API Layer
        APIGateway[API Gateway]
        RateLimiter[Rate Limiter]
        Auth[Authentication Service]
        AuthZ[Authorization Service]
    end

    subgraph Core Services
        UserService[User Service]
        OrderService[Order Service]
        PaymentService[Payment Service]
        NotificationService[Notification Service]
        SearchService[Search Service]
        InventoryService[Inventory Service]
    end

    subgraph Data Layer
        PostgresDB[(PostgreSQL - Primary)]
        ReadReplica[(PostgreSQL - Read Replica)]
        MongoDB[(MongoDB - Documents)]
        Redis[(Redis Cache)]
        Elasticsearch[(Elasticsearch)]
    end

    subgraph Messaging
        Kafka[Apache Kafka]
        DeadLetter[Dead Letter Queue]
    end

    subgraph External Services
        PaymentGateway[Payment Gateway]
        EmailProvider[Email Provider]
        SMSProvider[SMS Provider]
        CloudStorage[Cloud Storage]
    end

    subgraph Observability
        Prometheus[Prometheus]
        Grafana[Grafana]
        Jaeger[Jaeger Tracing]
        ELK[ELK Stack]
    end

    %% Client connections
    Web --> CDN
    Mobile --> CDN
    ThirdParty --> WAF
    CDN --> WAF
    WAF --> LB
    LB --> APIGateway

    %% API Layer flow
    APIGateway --> RateLimiter
    RateLimiter --> Auth
    Auth --> AuthZ
    AuthZ --> UserService
    AuthZ --> OrderService
    AuthZ --> PaymentService
    AuthZ --> SearchService
    AuthZ --> InventoryService

    %% Service to Database connections
    UserService --> PostgresDB
    UserService --> Redis
    OrderService --> PostgresDB
    OrderService --> ReadReplica
    PaymentService --> PostgresDB
    InventoryService --> PostgresDB
    InventoryService --> Redis
    SearchService --> Elasticsearch

    %% Event-driven connections
    OrderService --> Kafka
    PaymentService --> Kafka
    InventoryService --> Kafka
    Kafka --> NotificationService
    Kafka --> DeadLetter
    NotificationService --> MongoDB

    %% External service connections
    PaymentService --> PaymentGateway
    NotificationService --> EmailProvider
    NotificationService --> SMSProvider
    UserService --> CloudStorage

    %% Observability connections
    APIGateway -.-> Prometheus
    UserService -.-> Prometheus
    OrderService -.-> Prometheus
    PaymentService -.-> Prometheus
    Prometheus -.-> Grafana
    APIGateway -.-> Jaeger
    UserService -.-> Jaeger
    OrderService -.-> Jaeger
    Kafka -.-> ELK
```

## Architecture Components

### Edge Layer
- **CDN**: Caches static assets and reduces latency for global users
- **WAF**: Protects against common web exploits and DDoS attacks
- **Load Balancer**: Distributes traffic across multiple API Gateway instances

### API Layer
- **API Gateway**: Single entry point for all client requests, handles routing
- **Rate Limiter**: Prevents abuse and ensures fair resource allocation
- **Authentication Service**: Validates user identity (JWT, OAuth2)
- **Authorization Service**: Enforces access control policies (RBAC/ABAC)

### Core Services
- **User Service**: Manages user profiles, preferences, and account data
- **Order Service**: Handles order lifecycle and processing
- **Payment Service**: Processes transactions and manages payment methods
- **Notification Service**: Sends emails, SMS, and push notifications
- **Search Service**: Provides full-text search capabilities
- **Inventory Service**: Tracks stock levels and availability

### Data Layer
- **PostgreSQL Primary**: Main transactional database with ACID compliance
- **PostgreSQL Read Replica**: Offloads read queries for better performance
- **MongoDB**: Stores unstructured data like notifications and logs
- **Redis**: Caching layer for sessions, frequently accessed data
- **Elasticsearch**: Powers search functionality with full-text indexing

### Messaging
- **Apache Kafka**: Event streaming for async communication between services
- **Dead Letter Queue**: Captures failed messages for retry/analysis

### Observability
- **Prometheus + Grafana**: Metrics collection and visualization
- **Jaeger**: Distributed tracing for request flow analysis
- **ELK Stack**: Centralized logging and log analysis