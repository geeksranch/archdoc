# CI/CD Pipeline Architecture

## Overview

This document defines the CI/CD pipeline architecture for the enterprise hybrid cloud environment, including build, test, security scanning, and deployment processes.

## CI/CD Pipeline Architecture

```mermaid
flowchart LR
    subgraph Development["Development"]
        DEV[Developer]
        IDE[IDE/Local]
        GIT[Git Repository]
    end
    
    subgraph Build["Build & Test"]
        TRIGGER[Cloud Build Trigger]
        BUILD[Build Container]
        UNIT[Unit Tests]
        LINT[Linting]
    end
    
    subgraph Security["Security Scans"]
        SAST[SAST Scan]
        SCA[Dependency Scan]
        IMG[Container Scan]
        SEC_GATE[Security Gate]
    end
    
    subgraph Staging["Staging"]
        STG_DEPLOY[Deploy to Staging]
        INT_TEST[Integration Tests]
        PERF[Performance Tests]
    end
    
    subgraph Approval["Approval"]
        APPROVE{Manual Approval}
    end
    
    subgraph Production["Production"]
        PROD_DEPLOY[Deploy to Production]
        SMOKE[Smoke Tests]
        MONITOR[Monitoring]
    end
    
    DEV --> IDE
    IDE --> GIT
    GIT --> TRIGGER
    TRIGGER --> BUILD
    BUILD --> UNIT
    UNIT --> LINT
    LINT --> SAST
    SAST --> SCA
    SCA --> IMG
    IMG --> SEC_GATE
    SEC_GATE --> STG_DEPLOY
    STG_DEPLOY --> INT_TEST
    INT_TEST --> PERF
    PERF --> APPROVE
    APPROVE -->|Approved| PROD_DEPLOY
    APPROVE -->|Rejected| DEV
    PROD_DEPLOY --> SMOKE
    SMOKE --> MONITOR
```

## Pipeline Components

### Source Control

| Component | Technology | Purpose |
|-----------|------------|---------|
| Repository | GitHub / Cloud Source Repositories | Code storage |
| Branching | GitFlow | Branch strategy |
| Protection | Branch rules | Enforce reviews |
| Triggers | Cloud Build Triggers | Automated builds |

### Build Stage

```yaml
# cloudbuild.yaml example
steps:
  # Build container image
  - name: 'gcr.io/cloud-builders/docker'
    args: ['build', '-t', 'gcr.io/$PROJECT_ID/$_SERVICE_NAME:$COMMIT_SHA', '.']
  
  # Run unit tests
  - name: 'gcr.io/$PROJECT_ID/$_SERVICE_NAME:$COMMIT_SHA'
    entrypoint: 'sh'
    args: ['-c', 'npm test']
  
  # Run linting
  - name: 'gcr.io/$PROJECT_ID/$_SERVICE_NAME:$COMMIT_SHA'
    entrypoint: 'sh'
    args: ['-c', 'npm run lint']
```

### Security Scanning

```mermaid
flowchart TB
    subgraph SAST["Static Analysis"]
        S1[SonarQube]
        S2[Semgrep]
    end
    
    subgraph SCA["Dependency Analysis"]
        D1[Snyk]
        D2[OWASP Dependency Check]
    end
    
    subgraph Container["Container Analysis"]
        C1[Container Analysis API]
        C2[Artifact Registry Scanning]
    end
    
    subgraph Policy["Policy Enforcement"]
        P1[Binary Authorization]
        P2[Security Gate]
    end
    
    SAST --> Policy
    SCA --> Policy
    Container --> Policy
    
    P2 -->|Pass| DEPLOY[Deploy]
    P2 -->|Fail| BLOCK[Block]
```

### Security Gates

| Gate | Criteria | Action on Failure |
|------|----------|-------------------|
| SAST | No critical/high findings | Block deployment |
| SCA | No critical CVEs | Block deployment |
| Container Scan | No critical vulnerabilities | Block deployment |
| Binary Authorization | Attestations present | Block deployment |
| Secret Detection | No secrets in code | Block merge |

## Deployment Strategies

### Blue-Green Deployment

```mermaid
flowchart TB
    subgraph BlueGreen["Blue-Green Deployment"]
        LB[Load Balancer]
        
        subgraph Blue["Blue (Current)"]
            B1[Pod 1]
            B2[Pod 2]
        end
        
        subgraph Green["Green (New)"]
            G1[Pod 1]
            G2[Pod 2]
        end
        
        LB -->|100%| Blue
        LB -.->|0%| Green
    end
    
    subgraph Cutover["After Validation"]
        LB2[Load Balancer]
        
        subgraph Blue2["Blue (Previous)"]
            B3[Pod 1]
            B4[Pod 2]
        end
        
        subgraph Green2["Green (Current)"]
            G3[Pod 1]
            G4[Pod 2]
        end
        
        LB2 -.->|0%| Blue2
        LB2 -->|100%| Green2
    end
```

### Canary Deployment

```mermaid
flowchart LR
    subgraph Phase1["Phase 1: 5% Traffic"]
        P1_STABLE[Stable: 95%]
        P1_CANARY[Canary: 5%]
    end
    
    subgraph Phase2["Phase 2: 25% Traffic"]
        P2_STABLE[Stable: 75%]
        P2_CANARY[Canary: 25%]
    end
    
    subgraph Phase3["Phase 3: 100% Traffic"]
        P3_NEW[New Version: 100%]
    end
    
    Phase1 -->|Metrics OK| Phase2
    Phase2 -->|Metrics OK| Phase3
```

### Rolling Update

```mermaid
flowchart TB
    subgraph Rolling["Rolling Update"]
        subgraph Step1["Step 1"]
            S1_V1[v1]
            S1_V2[v1]
            S1_V3[v2]
        end
        
        subgraph Step2["Step 2"]
            S2_V1[v1]
            S2_V2[v2]
            S2_V3[v2]
        end
        
        subgraph Step3["Step 3"]
            S3_V1[v2]
            S3_V2[v2]
            S3_V3[v2]
        end
    end
    
    Step1 --> Step2
    Step2 --> Step3
```

## Environment Management

### Environment Configuration

| Environment | Purpose | Deployment | Approval |
|-------------|---------|------------|----------|
| **Development** | Developer testing | Automatic | None |
| **Integration** | Integration testing | Automatic | None |
| **Staging** | Pre-production validation | Automatic | None |
| **Production** | Live traffic | Manual | Required |

### Environment Promotion

```mermaid
flowchart LR
    DEV[Development] -->|Automated| INT[Integration]
    INT -->|Automated| STG[Staging]
    STG -->|Manual Approval| PROD[Production]
```

## GitOps Workflow

### Configuration Management

```mermaid
flowchart TB
    subgraph Repositories
        APP[Application Repo]
        CONFIG[Config Repo]
    end
    
    subgraph CI["CI Pipeline"]
        BUILD[Build]
        TEST[Test]
        PUSH[Push Image]
    end
    
    subgraph CD["CD Pipeline"]
        SYNC[Config Sync]
        DEPLOY[Deploy]
    end
    
    subgraph Cluster["GKE Cluster"]
        NS[Namespace]
        POD[Pods]
    end
    
    APP --> BUILD
    BUILD --> TEST
    TEST --> PUSH
    PUSH --> CONFIG
    CONFIG --> SYNC
    SYNC --> DEPLOY
    DEPLOY --> NS
    NS --> POD
```

### Anthos Config Management

```yaml
# config-sync configuration
apiVersion: configsync.gke.io/v1
kind: ConfigSync
metadata:
  name: config-management
spec:
  sourceType: git
  git:
    repo: https://github.com/org/config-repo
    branch: main
    dir: environments/production
    auth: gcpserviceaccount
  sourceFormat: unstructured
  policyDir: policies
```

## Pipeline Metrics

### Performance Metrics

| Metric | Target | Current | Trend |
|--------|--------|---------|-------|
| Build Time | < 10 min | 8 min | ↓ |
| Test Time | < 15 min | 12 min | ↓ |
| Deploy Time | < 5 min | 4 min | → |
| Lead Time | < 1 day | 4 hours | ↓ |
| Deployment Frequency | Daily | 2/day | ↑ |

### Quality Metrics

| Metric | Target | Current | Trend |
|--------|--------|---------|-------|
| Deployment Success Rate | > 99% | 98.5% | ↑ |
| Rollback Rate | < 2% | 1.5% | ↓ |
| Change Failure Rate | < 5% | 3% | ↓ |
| MTTR | < 1 hour | 45 min | ↓ |

## Pipeline Security

### Secret Management

```mermaid
flowchart TB
    subgraph Secrets["Secret Sources"]
        SM[Secret Manager]
        KMS[Cloud KMS]
    end
    
    subgraph Pipeline["Pipeline"]
        BUILD[Build Step]
        DEPLOY[Deploy Step]
    end
    
    subgraph Workload["Workload"]
        POD[Pod]
        WI[Workload Identity]
    end
    
    SM -->|Build-time| BUILD
    KMS -->|Encryption| SM
    SM -->|Runtime| POD
    POD --> WI
    WI --> SM
```

### Access Control

| Role | Repository | Build | Deploy Dev | Deploy Prod |
|------|------------|-------|------------|-------------|
| Developer | Read/Write | View | Execute | Request |
| Lead Developer | Read/Write | Execute | Execute | Approve |
| Platform Team | Read/Write | Admin | Admin | Execute |
| Security | Read | View | View | Approve |

---

[← Back to Governance Structure](governance-structure.md) | [Back to Phase G](README.md)
