# Disaster Recovery Architecture

## Overview

This document defines the disaster recovery (DR) architecture for the enterprise hybrid cloud environment, including recovery strategies, RTO/RPO targets, and failover procedures.

## Disaster Recovery Architecture

```mermaid
flowchart TB
    subgraph Primary["Primary Region (us-central1)"]
        subgraph ProdGKE["Production GKE"]
            P_APP[Applications]
            P_SVC[Services]
        end
        
        subgraph ProdData["Production Data"]
            P_SQL[(Cloud SQL)]
            P_BQ[(BigQuery)]
            P_GCS[(Cloud Storage)]
        end
        
        P_LB[Load Balancer]
    end
    
    subgraph DR["DR Region (us-east1)"]
        subgraph DRGKE["DR GKE"]
            D_APP[Applications]
            D_SVC[Services]
        end
        
        subgraph DRData["DR Data"]
            D_SQL[(Cloud SQL Replica)]
            D_BQ[(BigQuery)]
            D_GCS[(Cloud Storage)]
        end
        
        D_LB[Load Balancer]
    end
    
    subgraph OnPrem["On-Premises DR"]
        subgraph OnPremInfra["DR Infrastructure"]
            O_APP[Critical Apps]
            O_DB[(Database Replica)]
        end
        
        O_VPN[VPN Connection]
    end
    
    subgraph Replication["Replication"]
        R1[Sync Replication]
        R2[Async Replication]
    end
    
    P_SQL -->|Sync| D_SQL
    P_GCS -->|Async| D_GCS
    P_BQ -->|Cross-Region| D_BQ
    P_SQL -->|Async| O_DB
    
    Primary <-->|Cloud Interconnect| OnPrem
    DR <-->|VPN| OnPrem
```

## Recovery Objectives

### RTO/RPO Targets

| Tier | Description | RTO | RPO | Examples |
|------|-------------|-----|-----|----------|
| **Tier 1** | Mission Critical | 15 min | 0 min | Payment Processing, Core APIs |
| **Tier 2** | Business Critical | 1 hour | 15 min | Customer Portal, Order Management |
| **Tier 3** | Business Important | 4 hours | 1 hour | Analytics, Reporting |
| **Tier 4** | Business Support | 24 hours | 4 hours | Development, Testing |

### Recovery Strategies

| Strategy | RTO | Cost | Use Case |
|----------|-----|------|----------|
| **Hot Standby** | Minutes | High | Tier 1 systems |
| **Warm Standby** | 1-4 hours | Medium | Tier 2-3 systems |
| **Cold Standby** | 24+ hours | Low | Tier 4 systems |
| **Backup/Restore** | Days | Lowest | Archives |

## DR Components

### Compute DR

```mermaid
flowchart LR
    subgraph Primary["Primary (Active)"]
        P_GKE[GKE Cluster]
        P_CR[Cloud Run]
        P_CE[Compute Engine]
    end
    
    subgraph DR["DR (Standby)"]
        D_GKE[GKE Cluster]
        D_CR[Cloud Run]
        D_CE[Compute Engine]
    end
    
    P_GKE -.->|Config Sync| D_GKE
    P_CR -.->|Same Image| D_CR
    P_CE -.->|Snapshot| D_CE
```

### Data DR

| Service | Replication Method | RPO | DR Location |
|---------|-------------------|-----|-------------|
| **Cloud SQL** | Synchronous replica | 0 | us-east1 |
| **Cloud Spanner** | Multi-region | 0 | Global |
| **BigQuery** | Cross-region copy | 1 hour | us-east1 |
| **Cloud Storage** | Dual-region | 0 | us-central1/us-east1 |
| **Firestore** | Multi-region | 0 | nam5 |

### Network DR

```mermaid
flowchart TB
    subgraph DNS["Global DNS"]
        GDNS[Cloud DNS]
    end
    
    subgraph Primary["Primary Region"]
        P_GLB[Global LB]
        P_VPC[VPC]
    end
    
    subgraph DR["DR Region"]
        D_GLB[Global LB]
        D_VPC[VPC]
    end
    
    GDNS --> P_GLB
    GDNS -.->|Failover| D_GLB
    P_VPC <-->|VPC Peering| D_VPC
```

## Failover Procedures

### Automatic Failover

```mermaid
flowchart TB
    A[Health Check Failure] --> B{Consecutive Failures?}
    B -->|Yes| C[Mark Unhealthy]
    B -->|No| A
    C --> D[Remove from LB]
    D --> E[Route to DR]
    E --> F[Alert Operations]
    F --> G[Start Investigation]
```

### Manual Failover

```mermaid
flowchart TB
    A[Disaster Declared] --> B[Notify Teams]
    B --> C[Activate DR Runbook]
    C --> D{DR Type}
    
    D -->|Regional| E[Activate DR Region]
    D -->|Global| F[Activate All DR Sites]
    
    E --> G[Update DNS]
    F --> G
    G --> H[Verify Connectivity]
    H --> I[Validate Applications]
    I --> J[Confirm Failover]
    J --> K[Notify Stakeholders]
```

## DR Runbooks

### Regional Failover Runbook

| Step | Action | Owner | Time |
|------|--------|-------|------|
| 1 | Declare DR event | Incident Commander | 0 min |
| 2 | Notify DR team | On-Call Engineer | 5 min |
| 3 | Assess damage | Platform Team | 10 min |
| 4 | Promote DB replicas | DBA | 15 min |
| 5 | Update DNS records | Network Team | 20 min |
| 6 | Verify application health | App Teams | 30 min |
| 7 | Validate data integrity | Data Team | 45 min |
| 8 | Confirm DR complete | Incident Commander | 60 min |

### Failback Runbook

| Step | Action | Owner | Time |
|------|--------|-------|------|
| 1 | Confirm primary recovered | Platform Team | 0 min |
| 2 | Sync data to primary | DBA | 30 min |
| 3 | Test primary systems | App Teams | 60 min |
| 4 | Schedule failback window | Change Management | 24 hr |
| 5 | Execute failback | Platform Team | 24+ hr |
| 6 | Verify primary active | All Teams | 24+ hr |
| 7 | Restore DR standby | Platform Team | 48 hr |

## DR Testing

### Testing Schedule

| Test Type | Frequency | Duration | Scope |
|-----------|-----------|----------|-------|
| **Tabletop** | Monthly | 2 hours | Process review |
| **Component** | Quarterly | 4 hours | Individual systems |
| **Partial** | Semi-annually | 8 hours | Tier 1-2 systems |
| **Full** | Annually | 24 hours | All systems |

### Test Scenarios

| Scenario | Objective | Systems |
|----------|-----------|---------|
| Regional outage | Validate regional failover | All Tier 1-2 |
| Database failure | Test DB failover | Cloud SQL, Spanner |
| Network partition | Test hybrid connectivity | Interconnect, VPN |
| Ransomware | Test backup restore | All data systems |
| Cascading failure | Test circuit breakers | Microservices |

## Monitoring and Alerting

### DR Health Checks

| Check | Frequency | Alert Threshold |
|-------|-----------|-----------------|
| Replication lag | 1 min | > 5 minutes |
| Backup completion | Daily | Missed backup |
| DR site connectivity | 5 min | 3 failures |
| Replica health | 1 min | Unhealthy |
| DNS propagation | 5 min | Mismatch |

### DR Dashboard

| Metric | Target | Current |
|--------|--------|---------|
| Replication Status | Healthy | ✓ Healthy |
| Last Backup | < 24h | 2h ago |
| DR Site Status | Ready | ✓ Ready |
| Last DR Test | < 90 days | 45 days ago |
| Recovery Confidence | > 95% | 98% |

---

[← Back to Main Documentation](../../README.md)
