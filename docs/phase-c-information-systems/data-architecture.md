# Data Architecture

## Overview

This document defines the data architecture for the enterprise hybrid cloud environment, covering data sources, storage, processing, and governance.

## Data Architecture Flow

```mermaid
flowchart LR
    subgraph Sources["Data Sources"]
        direction TB
        S1[(Transactional DBs)]
        S2[(Legacy Systems)]
        S3[APIs]
        S4[IoT/Devices]
        S5[SaaS Apps]
    end
    
    subgraph Ingestion["Ingestion Layer"]
        direction TB
        I1[Pub/Sub]
        I2[Dataflow]
        I3[Cloud Functions]
        I4[Transfer Service]
    end
    
    subgraph Storage["Storage Layer"]
        direction TB
        subgraph OnPrem["On-Premises"]
            OP1[(Data Warehouse)]
            OP2[(Archive Storage)]
        end
        subgraph GCP["Google Cloud"]
            G1[(Cloud Storage)]
            G2[(BigQuery)]
            G3[(Cloud SQL)]
            G4[(Firestore)]
        end
    end
    
    subgraph Processing["Processing Layer"]
        direction TB
        P1[Dataproc]
        P2[Dataflow]
        P3[Cloud Composer]
        P4[BigQuery ML]
    end
    
    subgraph Consumption["Consumption Layer"]
        direction TB
        C1[Looker]
        C2[Data Studio]
        C3[APIs]
        C4[ML Models]
    end
    
    Sources --> Ingestion
    Ingestion --> Storage
    Storage --> Processing
    Processing --> Consumption
```

## Data Domains

### Domain Classification

| Domain | Description | Owner | Storage Location |
|--------|-------------|-------|-----------------|
| **Customer** | Customer profiles, preferences, interactions | CRM Team | Cloud SQL, BigQuery |
| **Product** | Product catalog, inventory, pricing | Product Team | Firestore, BigQuery |
| **Transaction** | Orders, payments, invoices | Finance Team | Cloud Spanner, BigQuery |
| **Analytics** | Aggregated metrics, KPIs, reports | Analytics Team | BigQuery |
| **Operational** | Logs, metrics, traces | Platform Team | Cloud Logging, BigQuery |

### Data Classification

| Classification | Description | Storage Requirements | Access Control |
|----------------|-------------|---------------------|----------------|
| **Public** | Publicly available information | Any | Open |
| **Internal** | Internal business information | Encrypted at rest | Role-based |
| **Confidential** | Sensitive business data | Encrypted, audited | Need-to-know |
| **Restricted** | Regulated/PII data | On-prem or VPC-SC | Strict controls |

## Storage Architecture

### Cloud Storage Strategy

```mermaid
flowchart TB
    subgraph HotData["Hot Data (Frequent Access)"]
        H1[(Cloud SQL)]
        H2[(Firestore)]
        H3[(Memorystore)]
    end
    
    subgraph WarmData["Warm Data (Periodic Access)"]
        W1[(BigQuery)]
        W2[(Cloud Storage Standard)]
    end
    
    subgraph ColdData["Cold Data (Archival)"]
        C1[(Cloud Storage Nearline)]
        C2[(Cloud Storage Archive)]
        C3[(On-Prem Archive)]
    end
    
    HotData -->|Age > 30 days| WarmData
    WarmData -->|Age > 1 year| ColdData
```

### Storage Services Matrix

| Service | Use Case | Performance | Cost |
|---------|----------|-------------|------|
| **Cloud SQL** | Transactional workloads | High | Medium |
| **Cloud Spanner** | Global transactions | Very High | High |
| **BigQuery** | Analytics, Data Warehouse | High | Low-Medium |
| **Cloud Storage** | Object storage, data lake | Medium | Low |
| **Firestore** | Document database | High | Medium |
| **Memorystore** | Caching, sessions | Very High | High |

## Data Flow Patterns

### Real-Time Streaming

```mermaid
flowchart LR
    A[Event Source] --> B[Pub/Sub Topic]
    B --> C[Dataflow Streaming]
    C --> D[(BigQuery)]
    C --> E[Cloud Functions]
    E --> F[Notifications]
    D --> G[Real-time Dashboard]
```

### Batch Processing

```mermaid
flowchart LR
    A[(Source Systems)] --> B[Cloud Storage]
    B --> C[Cloud Composer]
    C --> D[Dataflow Batch]
    D --> E[(BigQuery)]
    E --> F[Scheduled Reports]
```

### Change Data Capture (CDC)

```mermaid
flowchart LR
    A[(On-Prem DB)] --> B[Datastream]
    B --> C[Cloud Storage]
    C --> D[BigQuery]
    D --> E[Analytics]
```

## Data Governance

### Governance Framework

```mermaid
flowchart TB
    subgraph Governance["Data Governance"]
        G1[Data Stewardship]
        G2[Data Quality]
        G3[Data Security]
        G4[Data Lifecycle]
        G5[Compliance]
    end
    
    subgraph Implementation["Implementation"]
        I1[Data Catalog]
        I2[DLP]
        I3[IAM Policies]
        I4[Retention Policies]
        I5[Audit Logging]
    end
    
    G1 --> I1
    G2 --> I1
    G3 --> I2
    G3 --> I3
    G4 --> I4
    G5 --> I5
```

### Data Quality Rules

| Dimension | Description | Implementation |
|-----------|-------------|----------------|
| **Accuracy** | Data correctly represents reality | Validation rules, checksums |
| **Completeness** | All required data is present | Not-null constraints, alerts |
| **Consistency** | Data is consistent across systems | Master data management |
| **Timeliness** | Data is available when needed | SLA monitoring, alerts |
| **Uniqueness** | No unintended duplicates | Deduplication, primary keys |

### Data Catalog

| Attribute | Description |
|-----------|-------------|
| **Dataset Name** | Unique identifier for the dataset |
| **Description** | Business description and purpose |
| **Owner** | Data steward responsible |
| **Classification** | Security classification level |
| **Schema** | Structure and data types |
| **Lineage** | Source systems and transformations |
| **Quality Metrics** | Current quality scores |
| **Access Policy** | Who can access and how |

## Master Data Management

### MDM Architecture

```mermaid
flowchart TB
    subgraph Sources["Source Systems"]
        S1[CRM]
        S2[ERP]
        S3[E-commerce]
    end
    
    subgraph MDM["MDM Hub"]
        M1[Customer Master]
        M2[Product Master]
        M3[Vendor Master]
    end
    
    subgraph Consumers["Consumer Systems"]
        C1[Analytics]
        C2[Marketing]
        C3[Operations]
    end
    
    S1 --> MDM
    S2 --> MDM
    S3 --> MDM
    MDM --> C1
    MDM --> C2
    MDM --> C3
```

## Compliance & Privacy

### Data Residency

| Data Type | Residency Requirement | Storage Location |
|-----------|----------------------|------------------|
| EU Customer PII | EU only | europe-west1 |
| Financial Data | Country-specific | Regional buckets |
| Analytics Data | No restriction | us-central1 |
| Backup Data | Same as source | Multi-regional |

### Privacy Controls

| Control | Implementation |
|---------|----------------|
| **Data Masking** | DLP API for sensitive fields |
| **Encryption** | CMEK for regulated data |
| **Access Logging** | Cloud Audit Logs |
| **Retention** | Lifecycle policies |
| **Right to Deletion** | Automated workflows |

---

[← Back to Phase C](README.md) | [Next: Application Architecture →](application-architecture.md)
