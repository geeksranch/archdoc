# Security Architecture

## Overview

This document defines the security architecture for the enterprise hybrid cloud environment, covering identity management, network security, data protection, and security operations.

## Security Architecture Layers

```mermaid
flowchart TB
    subgraph Perimeter["Perimeter Security"]
        P1[Cloud Armor WAF]
        P2[DDoS Protection]
        P3[Bot Management]
    end
    
    subgraph Identity["Identity & Access"]
        I1[Cloud Identity]
        I2[Identity Platform]
        I3[Workload Identity]
        I4[Service Accounts]
    end
    
    subgraph Network["Network Security"]
        N1[VPC Firewall]
        N2[VPC Service Controls]
        N3[Private Service Connect]
        N4[Cloud NAT]
    end
    
    subgraph Data["Data Security"]
        D1[Cloud KMS]
        D2[Secret Manager]
        D3[Cloud DLP]
        D4[Data Catalog]
    end
    
    subgraph Workload["Workload Security"]
        W1[Binary Authorization]
        W2[Container Analysis]
        W3[Web Security Scanner]
        W4[Artifact Registry]
    end
    
    subgraph Operations["Security Operations"]
        O1[Security Command Center]
        O2[Chronicle SIEM]
        O3[Cloud Audit Logs]
        O4[Event Threat Detection]
    end
    
    Perimeter --> Identity
    Identity --> Network
    Network --> Data
    Data --> Workload
    Workload --> Operations
```

## Identity and Access Management

### IAM Architecture

```mermaid
flowchart TB
    subgraph IdP["Identity Providers"]
        CORP[Corporate AD]
        CLOUD[Cloud Identity]
        EXT[External IdP]
    end
    
    subgraph Federation["Federation"]
        SAML[SAML/OIDC]
        WIF[Workload Identity Federation]
    end
    
    subgraph IAM["GCP IAM"]
        ORG[Organization Policies]
        FOLDER[Folder IAM]
        PROJECT[Project IAM]
        RESOURCE[Resource IAM]
    end
    
    subgraph Principals["Principals"]
        USER[Users]
        GROUP[Groups]
        SA[Service Accounts]
        WI[Workload Identity]
    end
    
    IdP --> Federation
    Federation --> IAM
    Principals --> IAM
```

### Role Definitions

| Role | Scope | Permissions | Use Case |
|------|-------|-------------|----------|
| **Organization Admin** | Organization | Full admin | Platform team |
| **Folder Admin** | Folder | Folder management | Environment owners |
| **Project Owner** | Project | Full project access | Project leads |
| **GKE Admin** | Project | GKE cluster admin | Platform team |
| **Developer** | Project | Deploy, view logs | Dev teams |
| **Viewer** | Project | Read-only | Stakeholders |

### Service Account Strategy

| Service Account | Purpose | Key Rotation | Access |
|-----------------|---------|--------------|--------|
| terraform-sa | Infrastructure provisioning | 90 days | Org-level |
| gke-nodes-sa | GKE node identity | N/A (Workload Identity) | Project-level |
| app-sa | Application identity | N/A (Workload Identity) | Resource-level |
| cicd-sa | CI/CD pipeline | 30 days | Project-level |

## Network Security

### Defense in Depth

```mermaid
flowchart LR
    subgraph Layer1["Layer 1: Edge"]
        ARMOR[Cloud Armor]
        CDN[Cloud CDN]
    end
    
    subgraph Layer2["Layer 2: Perimeter"]
        LB[Load Balancer]
        SSL[SSL/TLS]
    end
    
    subgraph Layer3["Layer 3: Network"]
        FW[Firewall Rules]
        VPCSC[VPC-SC]
    end
    
    subgraph Layer4["Layer 4: Workload"]
        MESH[Service Mesh]
        mTLS[mTLS]
    end
    
    subgraph Layer5["Layer 5: Data"]
        ENC[Encryption]
        DLP[DLP]
    end
    
    Layer1 --> Layer2
    Layer2 --> Layer3
    Layer3 --> Layer4
    Layer4 --> Layer5
```

### Cloud Armor Policies

| Policy | Type | Action | Use Case |
|--------|------|--------|----------|
| rate-limit | Rate limiting | Throttle | DDoS protection |
| geo-block | Geographic | Block | Geo-restriction |
| owasp-crs | WAF | Block/Log | OWASP Top 10 |
| bot-protection | Bot mgmt | Challenge | Bot mitigation |
| custom-rules | L7 | Custom | Application-specific |

### VPC Service Controls

```mermaid
flowchart TB
    subgraph Perimeter["Secure Perimeter"]
        subgraph Services["Protected Services"]
            BQ[BigQuery]
            GCS[Cloud Storage]
            SQL[Cloud SQL]
            GKE[GKE]
        end
        
        subgraph AccessLevels["Access Levels"]
            CORP[Corporate Network]
            VPN[VPN Users]
            SA[Service Accounts]
        end
    end
    
    subgraph Ingress["Ingress Rules"]
        IN1[Allow from Corporate]
        IN2[Allow CI/CD]
    end
    
    subgraph Egress["Egress Rules"]
        EG1[Allow to backup]
        EG2[Allow monitoring]
    end
    
    AccessLevels --> Services
    Ingress --> Perimeter
    Perimeter --> Egress
```

## Data Security

### Encryption Architecture

| Layer | Method | Key Management |
|-------|--------|----------------|
| **At Rest** | AES-256 | CMEK or Google-managed |
| **In Transit** | TLS 1.3 | Managed certificates |
| **In Use** | Confidential Computing | AMD SEV |

### Key Management

```mermaid
flowchart TB
    subgraph KMS["Cloud KMS"]
        KR1[KeyRing: production]
        KR2[KeyRing: non-production]
        
        KR1 --> K1[Key: data-encryption]
        KR1 --> K2[Key: secrets-encryption]
        KR2 --> K3[Key: dev-encryption]
    end
    
    subgraph Usage["Key Usage"]
        GCS[Cloud Storage]
        BQ[BigQuery]
        SQL[Cloud SQL]
        GKE[GKE Secrets]
    end
    
    K1 --> GCS
    K1 --> BQ
    K1 --> SQL
    K2 --> GKE
```

### Data Classification

| Classification | Examples | Encryption | Access Control | Retention |
|----------------|----------|------------|----------------|-----------|
| **Public** | Marketing content | Google-managed | Open | Indefinite |
| **Internal** | Business docs | Google-managed | Authenticated | 7 years |
| **Confidential** | Financial data | CMEK | Need-to-know | 7 years |
| **Restricted** | PII, PHI | CMEK + VPC-SC | Strict controls | As required |

## Workload Security

### Container Security

```mermaid
flowchart LR
    subgraph Build["Build Phase"]
        SRC[Source Code]
        SCAN1[SAST Scan]
        BUILD[Container Build]
        SCAN2[Image Scan]
    end
    
    subgraph Deploy["Deploy Phase"]
        REG[Artifact Registry]
        BA[Binary Authorization]
        GKE[GKE Cluster]
    end
    
    subgraph Runtime["Runtime Phase"]
        POD[Running Pods]
        WI[Workload Identity]
        MON[Security Monitoring]
    end
    
    SRC --> SCAN1
    SCAN1 --> BUILD
    BUILD --> SCAN2
    SCAN2 --> REG
    REG --> BA
    BA --> GKE
    GKE --> POD
    POD --> WI
    POD --> MON
```

### Binary Authorization

| Attestor | Description | Required |
|----------|-------------|----------|
| vulnerability-scan | No critical CVEs | Yes |
| code-review | PR approved | Yes |
| qa-approval | QA sign-off | Production only |
| security-approval | Security review | Critical systems |

## Security Operations

### Security Monitoring

```mermaid
flowchart TB
    subgraph Sources["Log Sources"]
        AUDIT[Cloud Audit Logs]
        VPC[VPC Flow Logs]
        DNS[DNS Logs]
        FW[Firewall Logs]
        APP[Application Logs]
    end
    
    subgraph Collection["Log Collection"]
        SINK[Log Sink]
        PUBSUB[Pub/Sub]
    end
    
    subgraph Analysis["Analysis"]
        SCC[Security Command Center]
        CHRONICLE[Chronicle SIEM]
        BQ[BigQuery]
    end
    
    subgraph Response["Response"]
        ALERT[Alerting]
        SOAR[SOAR Playbooks]
        TICKET[Ticketing]
    end
    
    Sources --> Collection
    Collection --> Analysis
    Analysis --> Response
```

### Security Command Center

| Finding Type | Severity | Action |
|--------------|----------|--------|
| Public IP exposure | High | Auto-remediate |
| Unencrypted storage | High | Alert + remediate |
| IAM anomaly | Medium | Alert + investigate |
| Vulnerability | Variable | Prioritized patching |
| Compliance violation | Variable | Alert + document |

### Incident Response

```mermaid
stateDiagram-v2
    [*] --> Detection
    Detection --> Triage
    Triage --> Analysis
    Analysis --> Containment
    Containment --> Eradication
    Eradication --> Recovery
    Recovery --> PostIncident
    PostIncident --> [*]
    
    Analysis --> Escalation: Critical
    Escalation --> Containment
```

## Compliance

### Compliance Frameworks

| Framework | Scope | Status | Review Cycle |
|-----------|-------|--------|--------------|
| SOC 2 Type II | All systems | Compliant | Annual |
| ISO 27001 | All systems | Compliant | Annual |
| GDPR | EU data | Compliant | Ongoing |
| PCI DSS | Payment systems | Level 1 | Quarterly |
| HIPAA | Healthcare data | Compliant | Annual |

### Audit Controls

| Control Area | Implementation | Evidence |
|--------------|----------------|----------|
| Access Control | IAM, VPC-SC | Access logs, IAM policies |
| Encryption | CMEK, TLS | KMS logs, certificates |
| Monitoring | SCC, Chronicle | SIEM dashboards, alerts |
| Change Management | CAB, GitOps | Change tickets, git history |
| Backup | Scheduled snapshots | Backup logs, test restores |

---

[← Back to Network Architecture](network-architecture.md) | [Back to Phase D](README.md)
