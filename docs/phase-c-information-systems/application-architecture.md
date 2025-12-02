# Application Architecture

## Overview

This document defines the application architecture for the enterprise hybrid cloud environment, including application patterns, layers, and integration approaches.

## Application Architecture Layers

```mermaid
block-beta
    columns 1
    
    block:presentation:1
        columns 3
        P1["Web Applications"]
        P2["Mobile Apps"]
        P3["Partner Portals"]
    end
    
    block:api:1
        columns 2
        A1["API Gateway (Apigee)"]
        A2["Identity & Access Management"]
    end
    
    block:services:1
        columns 4
        S1["Customer Service"]
        S2["Product Service"]
        S3["Order Service"]
        S4["Analytics Service"]
    end
    
    block:integration:1
        columns 3
        I1["Event Bus (Pub/Sub)"]
        I2["Service Mesh (Anthos)"]
        I3["Integration Platform"]
    end
    
    block:data:1
        columns 4
        D1["Cloud SQL"]
        D2["BigQuery"]
        D3["Cloud Storage"]
        D4["Firestore"]
    end
    
    presentation --> api
    api --> services
    services --> integration
    integration --> data
```

## Application Portfolio

### Application Classification

| Category | Description | Examples | Target Platform |
|----------|-------------|----------|-----------------|
| **Customer-Facing** | External user applications | Web portal, Mobile app | GKE, Cloud Run |
| **Internal Tools** | Employee applications | Admin dashboards, CRM | GKE, App Engine |
| **Integration** | System connectors | APIs, ETL jobs | Cloud Functions, Dataflow |
| **Analytics** | BI and reporting | Dashboards, Reports | Looker, BigQuery |
| **Legacy** | Existing on-prem systems | ERP, Mainframe | Retain on-premises |

### Application Inventory

| Application | Type | Current State | Target State | Migration Strategy |
|-------------|------|--------------|--------------|-------------------|
| Customer Portal | Web App | On-Prem VMs | GKE | Replatform |
| Mobile API | API Service | Monolith | Microservices | Refactor |
| Admin Dashboard | Internal | Legacy | Cloud Run | Repurchase |
| Data Warehouse | Analytics | On-Prem | BigQuery | Replatform |
| ERP System | Enterprise | On-Prem | Hybrid | Retain |

## Microservices Architecture

### Service Design

```mermaid
flowchart TB
    subgraph Presentation["Presentation Layer"]
        WEB[Web App]
        MOBILE[Mobile App]
    end
    
    subgraph Gateway["API Gateway Layer"]
        APIGEE[Apigee API Gateway]
        AUTH[Cloud Identity]
    end
    
    subgraph Services["Microservices Layer"]
        direction LR
        CS[Customer Service]
        PS[Product Service]
        OS[Order Service]
        NS[Notification Service]
        AS[Analytics Service]
    end
    
    subgraph Mesh["Service Mesh"]
        ISTIO[Istio/Anthos Service Mesh]
    end
    
    subgraph Data["Data Layer"]
        direction LR
        CSDB[(Customer DB)]
        PSDB[(Product DB)]
        OSDB[(Order DB)]
        BQ[(BigQuery)]
    end
    
    WEB --> APIGEE
    MOBILE --> APIGEE
    APIGEE --> AUTH
    AUTH --> Services
    Services --> Mesh
    
    CS --> CSDB
    PS --> PSDB
    OS --> OSDB
    AS --> BQ
```

### Service Specifications

| Service | Responsibility | API Type | Data Store | Scale |
|---------|---------------|----------|------------|-------|
| **Customer Service** | Customer profiles, preferences | REST/gRPC | Cloud SQL | High |
| **Product Service** | Product catalog, inventory | REST | Firestore | High |
| **Order Service** | Order management, payments | REST/gRPC | Cloud Spanner | High |
| **Notification Service** | Email, SMS, push notifications | Async | Pub/Sub | Medium |
| **Analytics Service** | Metrics, reporting | REST | BigQuery | Low |

## API Architecture

### API Design Patterns

```mermaid
flowchart LR
    subgraph Clients["API Consumers"]
        C1[Web App]
        C2[Mobile App]
        C3[Partners]
        C4[Internal Services]
    end
    
    subgraph Gateway["Apigee API Gateway"]
        G1[Rate Limiting]
        G2[Authentication]
        G3[Transformation]
        G4[Analytics]
    end
    
    subgraph Products["API Products"]
        P1[Customer API v1]
        P2[Product API v1]
        P3[Order API v1]
        P4[Internal API v1]
    end
    
    subgraph Backend["Backend Services"]
        B1[Customer Service]
        B2[Product Service]
        B3[Order Service]
        B4[Legacy Adapter]
    end
    
    Clients --> Gateway
    Gateway --> Products
    Products --> Backend
```

### API Standards

| Standard | Description |
|----------|-------------|
| **REST** | Primary API style for external APIs |
| **gRPC** | Service-to-service communication |
| **GraphQL** | Complex query requirements |
| **OpenAPI 3.0** | API specification format |
| **OAuth 2.0** | Authentication standard |
| **JSON** | Data format for REST APIs |

## Container Architecture

### GKE Cluster Design

```mermaid
flowchart TB
    subgraph GKE["GKE Cluster"]
        subgraph NodePool1["System Node Pool"]
            N1[System Workloads]
        end
        
        subgraph NodePool2["Application Node Pool"]
            N2[Customer Service]
            N3[Product Service]
            N4[Order Service]
        end
        
        subgraph NodePool3["Data Node Pool"]
            N5[Analytics Service]
            N6[ETL Jobs]
        end
    end
    
    subgraph Services["Kubernetes Services"]
        LB[Cloud Load Balancer]
        ING[Ingress Controller]
        SVC[ClusterIP Services]
    end
    
    LB --> ING
    ING --> NodePool2
    NodePool2 --> SVC
    SVC --> NodePool3
```

### Container Standards

| Standard | Specification |
|----------|--------------|
| **Base Images** | Google Distroless images |
| **Registry** | Artifact Registry |
| **Secrets** | Secret Manager |
| **Config** | ConfigMaps, Workload Identity |
| **Logging** | Cloud Logging agent |
| **Monitoring** | Cloud Monitoring, Prometheus |

## Frontend Architecture

### Web Application Architecture

```mermaid
flowchart LR
    subgraph CDN["Content Delivery"]
        C1[Cloud CDN]
        C2[Firebase Hosting]
    end
    
    subgraph Frontend["Frontend Layer"]
        F1[React/Angular SPA]
        F2[Server-Side Rendering]
    end
    
    subgraph BFF["Backend for Frontend"]
        B1[Web BFF]
        B2[Mobile BFF]
    end
    
    subgraph API["API Layer"]
        A1[Apigee Gateway]
    end
    
    CDN --> Frontend
    Frontend --> BFF
    BFF --> API
```

## Integration Patterns

### Synchronous Integration

```mermaid
sequenceDiagram
    participant Client
    participant Gateway as API Gateway
    participant Service as Microservice
    participant DB as Database
    
    Client->>Gateway: HTTP Request
    Gateway->>Gateway: Auth/Rate Limit
    Gateway->>Service: Forward Request
    Service->>DB: Query
    DB-->>Service: Result
    Service-->>Gateway: Response
    Gateway-->>Client: HTTP Response
```

### Asynchronous Integration

```mermaid
sequenceDiagram
    participant Producer
    participant PubSub as Pub/Sub
    participant Consumer1 as Consumer 1
    participant Consumer2 as Consumer 2
    participant Storage as Cloud Storage
    
    Producer->>PubSub: Publish Event
    PubSub->>Consumer1: Deliver Event
    PubSub->>Consumer2: Deliver Event
    Consumer1->>Storage: Store Result
    Consumer2->>Consumer2: Process Event
```

## Application Security

### Security Layers

| Layer | Controls |
|-------|----------|
| **Edge** | Cloud Armor WAF, DDoS protection |
| **Gateway** | API key validation, OAuth 2.0, rate limiting |
| **Service** | Service mesh mTLS, RBAC |
| **Data** | Encryption at rest, VPC Service Controls |
| **Runtime** | Binary Authorization, Workload Identity |

### Security Implementation

```mermaid
flowchart TB
    subgraph Edge["Edge Security"]
        E1[Cloud Armor]
        E2[Cloud Load Balancer]
    end
    
    subgraph Gateway["Gateway Security"]
        G1[Apigee API Gateway]
        G2[Identity Platform]
    end
    
    subgraph Service["Service Security"]
        S1[Service Mesh mTLS]
        S2[Workload Identity]
    end
    
    subgraph Data["Data Security"]
        D1[VPC Service Controls]
        D2[CMEK Encryption]
    end
    
    Edge --> Gateway
    Gateway --> Service
    Service --> Data
```

---

[← Back to Data Architecture](data-architecture.md) | [Back to Phase C](README.md)
