# Infrastructure Architecture

## Overview

This document defines the multi-cloud and on-premises infrastructure architecture for the enterprise hybrid cloud environment.

## Infrastructure Architecture Overview

```mermaid
flowchart TB
    subgraph GCP["Google Cloud Platform"]
        subgraph GKE["GKE Clusters"]
            GKE1[Production Cluster]
            GKE2[Non-Prod Cluster]
        end
        
        subgraph Compute["Compute Engine"]
            VM1[Legacy Apps]
            VM2[Management VMs]
        end
        
        subgraph AppEngine["App Engine"]
            AE1[Web Applications]
        end
        
        subgraph Serverless["Serverless"]
            CF[Cloud Functions]
            CR[Cloud Run]
        end
    end
    
    subgraph Hybrid["Hybrid Connectivity"]
        IC[Cloud Interconnect]
        VPN[Cloud VPN]
        ANTHOS[Anthos]
    end
    
    subgraph OnPrem["On-Premises"]
        subgraph VMware["VMware vSphere"]
            VMW1[Production VMs]
            VMW2[Dev/Test VMs]
        end
        
        subgraph BareMetal["Bare Metal"]
            BM1[Database Servers]
            BM2[Legacy Systems]
        end
        
        subgraph K8sOnPrem["Kubernetes"]
            K8S1[Anthos On-Prem]
        end
    end
    
    GCP <--> Hybrid
    Hybrid <--> OnPrem
```

## GCP Infrastructure

### Compute Services

| Service | Use Case | Configuration | Cost Model |
|---------|----------|---------------|------------|
| **GKE** | Container workloads | Regional clusters, Autopilot | Per node |
| **Compute Engine** | VM workloads | N2/E2 instances, preemptible | Per hour |
| **Cloud Run** | Stateless containers | Auto-scaling, min instances | Per request |
| **App Engine** | Web applications | Standard/Flexible | Per instance hour |
| **Cloud Functions** | Event-driven functions | Gen 2, concurrency | Per invocation |

### GKE Architecture

```mermaid
flowchart TB
    subgraph Production["Production Cluster (Regional)"]
        subgraph CP["Control Plane"]
            CP1[GKE Control Plane]
        end
        
        subgraph NP1["System Pool"]
            SN1[System Workloads]
            SN2[Monitoring Agents]
        end
        
        subgraph NP2["App Pool (Auto-scaling)"]
            AN1[Application Pods]
            AN2[Application Pods]
        end
        
        subgraph NP3["Data Pool"]
            DN1[Data Processing]
        end
    end
    
    subgraph Services["GKE Add-ons"]
        ASM[Anthos Service Mesh]
        GW[Gateway Controller]
        SEC[Config Connector]
    end
    
    CP --> NP1
    CP --> NP2
    CP --> NP3
    Production --> Services
```

### GKE Specifications

| Cluster | Environment | Region | Node Pools | Nodes | Machine Type |
|---------|-------------|--------|------------|-------|--------------|
| prod-cluster | Production | us-central1 | 3 | 6-30 | n2-standard-8 |
| nonprod-cluster | Dev/Test | us-central1 | 2 | 2-10 | e2-standard-4 |

### Compute Engine Architecture

| Instance Group | Purpose | Machine Type | Count | Zone Distribution |
|----------------|---------|--------------|-------|-------------------|
| web-servers | Legacy web apps | n2-standard-4 | 4 | Multi-zone |
| management | Bastion, jumpbox | e2-medium | 2 | Single zone |
| database | Legacy databases | n2-highmem-8 | 2 | Multi-zone |

## On-Premises Infrastructure

### VMware Environment

```mermaid
flowchart TB
    subgraph vCenter["vCenter Server"]
        VC[vCenter 8.0]
    end
    
    subgraph Cluster1["Production Cluster"]
        H1[ESXi Host 1]
        H2[ESXi Host 2]
        H3[ESXi Host 3]
    end
    
    subgraph Cluster2["Non-Prod Cluster"]
        H4[ESXi Host 4]
        H5[ESXi Host 5]
    end
    
    subgraph Storage["Storage"]
        SAN[NetApp SAN]
        NAS[NFS Storage]
    end
    
    vCenter --> Cluster1
    vCenter --> Cluster2
    Cluster1 --> Storage
    Cluster2 --> Storage
```

### On-Premises Specifications

| Component | Specification | Purpose |
|-----------|---------------|---------|
| ESXi Hosts | Dell PowerEdge R750, 64 cores, 512GB RAM | Virtualization |
| Storage | NetApp AFF A400, 100TB | Primary storage |
| Backup | NetApp StorageGRID | Backup and archive |
| Network | Cisco Nexus 9000 | Data center switching |

## Hybrid Connectivity

### Anthos Multi-Cloud

```mermaid
flowchart LR
    subgraph GCP["Google Cloud"]
        GKE[GKE Cluster]
        ACM[Anthos Config Mgmt]
        ASM[Anthos Service Mesh]
    end
    
    subgraph OnPrem["On-Premises"]
        AON[Anthos On-Prem]
        VMW[VMware vSphere]
    end
    
    subgraph Management["Management"]
        HUB[Fleet Management]
        POL[Policy Controller]
    end
    
    GKE --> ACM
    AON --> ACM
    ACM --> HUB
    ASM --> GKE
    ASM --> AON
    HUB --> POL
```

### Interconnect Architecture

| Connection | Type | Bandwidth | Redundancy |
|------------|------|-----------|------------|
| Primary | Dedicated Interconnect | 10 Gbps | LACP |
| Secondary | Partner Interconnect | 10 Gbps | Diverse path |
| Backup | Cloud VPN | 3 Gbps | HA VPN |

## Storage Architecture

### Cloud Storage Classes

| Class | Use Case | Availability | Cost |
|-------|----------|--------------|------|
| **Standard** | Frequently accessed | 99.99% | $0.020/GB |
| **Nearline** | Monthly access | 99.9% | $0.010/GB |
| **Coldline** | Quarterly access | 99.9% | $0.004/GB |
| **Archive** | Yearly access | 99.9% | $0.001/GB |

### Persistent Storage

| Type | Use Case | Performance | Redundancy |
|------|----------|-------------|------------|
| **pd-ssd** | Database, high IOPS | 30,000 IOPS | Regional |
| **pd-balanced** | General workloads | 6,000 IOPS | Regional |
| **pd-standard** | Batch, logs | 3,000 IOPS | Zonal |
| **Filestore** | Shared file system | NFS v3 | Regional |

## Resource Management

### Organization Structure

```mermaid
flowchart TB
    ORG[Organization]
    
    ORG --> F1[Production Folder]
    ORG --> F2[Non-Production Folder]
    ORG --> F3[Shared Services Folder]
    
    F1 --> P1[Project: prod-apps]
    F1 --> P2[Project: prod-data]
    
    F2 --> P3[Project: dev]
    F2 --> P4[Project: test]
    F2 --> P5[Project: staging]
    
    F3 --> P6[Project: shared-vpc-host]
    F3 --> P7[Project: security]
    F3 --> P8[Project: logging]
```

### Quotas and Limits

| Resource | Production | Non-Production | Shared |
|----------|------------|----------------|--------|
| CPUs | 500 | 200 | 50 |
| Memory (GB) | 2,000 | 500 | 100 |
| GPUs | 8 | 2 | 0 |
| IPs | 100 | 50 | 20 |
| Storage (TB) | 100 | 20 | 10 |

## Infrastructure as Code

### Terraform Structure

```
terraform/
├── modules/
│   ├── gke-cluster/
│   ├── vpc-network/
│   ├── compute-instance/
│   └── storage-bucket/
├── environments/
│   ├── production/
│   ├── staging/
│   └── development/
└── shared/
    ├── iam/
    └── networking/
```

### Deployment Pipeline

```mermaid
flowchart LR
    A[Git Push] --> B[Cloud Build Trigger]
    B --> C[Terraform Plan]
    C --> D{Review}
    D -->|Approved| E[Terraform Apply]
    D -->|Rejected| F[Fix & Retry]
    E --> G[Update State]
    G --> H[Notify]
```

---

[← Back to Phase D](README.md) | [Next: Network Architecture →](network-architecture.md)
