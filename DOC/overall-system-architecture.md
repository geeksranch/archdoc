# Overall System Architecture

This document provides two views of the system architecture: a simplified overview for quick understanding and a detailed diagram showing all components and connections.

## Architecture Overview

The following diagram provides a high-level view of the main architectural layers and their relationships:

```mermaid
graph TD
    Clients[Clients]
    Edge[Edge Layer]
    API[API Layer]
    Core[Core Services]
    Data[Data Layer]
    Messaging[Messaging]
    External[External Services]
    Observability[Observability]

    Clients --> Edge
    Edge --> API
    API --> Core
    Core --> Data
    Core --> Messaging
    Core --> External
    Messaging --> Core
    
    Core -.-> Observability
    API -.-> Observability
    Messaging -.-> Observability

    %% Layer color definitions
    classDef clients fill:#E3F2FD,stroke:#1565C0,stroke-width:2px,color:#0D47A1
    classDef edge fill:#FFF3E0,stroke:#EF6C00,stroke-width:2px,color:#E65100
    classDef api fill:#F3E5F5,stroke:#7B1FA2,stroke-width:2px,color:#4A148C
    classDef core fill:#E8F5E9,stroke:#2E7D32,stroke-width:2px,color:#1B5E20
    classDef data fill:#FBE9E7,stroke:#D84315,stroke-width:2px,color:#BF360C
    classDef messaging fill:#E0F7FA,stroke:#00838F,stroke-width:2px,color:#006064
    classDef external fill:#FFF8E1,stroke:#F9A825,stroke-width:2px,color:#F57F17
    classDef observability fill:#F5F5F5,stroke:#616161,stroke-width:2px,color:#212121

    class Clients clients
    class Edge edge
    class API api
    class Core core
    class Data data
    class Messaging messaging
    class External external
    class Observability observability
```

## Detailed Architecture

The following diagram shows all components, their connections, and the detailed structure of each layer:

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

    %% Component color definitions
    classDef clients fill:#E3F2FD,stroke:#1565C0,stroke-width:2px,color:#0D47A1
    classDef edge fill:#FFF3E0,stroke:#EF6C00,stroke-width:2px,color:#E65100
    classDef api fill:#F3E5F5,stroke:#7B1FA2,stroke-width:2px,color:#4A148C
    classDef core fill:#E8F5E9,stroke:#2E7D32,stroke-width:2px,color:#1B5E20
    classDef data fill:#FBE9E7,stroke:#D84315,stroke-width:2px,color:#BF360C
    classDef messaging fill:#E0F7FA,stroke:#00838F,stroke-width:2px,color:#006064
    classDef external fill:#FFF8E1,stroke:#F9A825,stroke-width:2px,color:#F57F17
    classDef observability fill:#F5F5F5,stroke:#616161,stroke-width:2px,color:#212121

    %% Apply styles to Clients components
    class Web,Mobile,ThirdParty clients

    %% Apply styles to Edge Layer components
    class CDN,LB,WAF edge

    %% Apply styles to API Layer components
    class APIGateway,RateLimiter,Auth,AuthZ api

    %% Apply styles to Core Services components
    class UserService,OrderService,PaymentService,NotificationService,SearchService,InventoryService core

    %% Apply styles to Data Layer components
    class PostgresDB,ReadReplica,MongoDB,Redis,Elasticsearch data

    %% Apply styles to Messaging components
    class Kafka,DeadLetter messaging

    %% Apply styles to External Services components
    class PaymentGateway,EmailProvider,SMSProvider,CloudStorage external

    %% Apply styles to Observability components
    class Prometheus,Grafana,Jaeger,ELK observability
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