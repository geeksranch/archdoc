# Network Architecture

## Overview

This document defines the network architecture for the enterprise hybrid cloud environment, including VPC design, connectivity, and traffic management.

## Network Topology

```mermaid
flowchart TB
    subgraph Internet["Internet"]
        EXT[External Users]
    end
    
    subgraph GCP["GCP Network (10.0.0.0/8)"]
        subgraph SharedVPC["Shared VPC Host Project"]
            subgraph ProdVPC["Production VPC"]
                subgraph ProdSub["Production Subnets"]
                    PS1[10.0.1.0/24 GKE Nodes]
                    PS2[10.0.2.0/24 GKE Pods]
                    PS3[10.0.3.0/24 GKE Services]
                    PS4[10.0.4.0/24 Compute]
                end
            end
            
            subgraph NonProdVPC["Non-Production VPC"]
                subgraph NPSub["Non-Prod Subnets"]
                    NPS1[10.1.1.0/24 Dev]
                    NPS2[10.1.2.0/24 Test]
                    NPS3[10.1.3.0/24 Staging]
                end
            end
        end
        
        subgraph EdgeVPC["Edge VPC"]
            LB[Cloud Load Balancer]
            ARMOR[Cloud Armor]
        end
    end
    
    subgraph OnPrem["On-Premises (172.16.0.0/12)"]
        subgraph DCNetwork["Data Center Network"]
            DC1[172.16.1.0/24 Production]
            DC2[172.16.2.0/24 Management]
            DC3[172.16.3.0/24 DMZ]
        end
    end
    
    subgraph Connectivity["Hybrid Connectivity"]
        IC[Cloud Interconnect]
        VPN[Cloud VPN]
    end
    
    EXT --> EdgeVPC
    EdgeVPC --> ProdVPC
    ProdVPC <--> Connectivity
    NonProdVPC <--> Connectivity
    Connectivity <--> OnPrem
```

## VPC Architecture

### VPC Design

| VPC | Purpose | CIDR | Region | Peering |
|-----|---------|------|--------|---------|
| **prod-vpc** | Production workloads | 10.0.0.0/16 | us-central1 | Shared VPC |
| **nonprod-vpc** | Dev/Test/Staging | 10.1.0.0/16 | us-central1 | Shared VPC |
| **shared-vpc** | Shared services | 10.2.0.0/16 | us-central1 | Host project |
| **edge-vpc** | External connectivity | 10.3.0.0/16 | Global | - |

### Subnet Design

| Subnet | VPC | CIDR | Purpose | Secondary Ranges |
|--------|-----|------|---------|------------------|
| prod-gke-nodes | prod-vpc | 10.0.1.0/24 | GKE node IPs | - |
| prod-gke-pods | prod-vpc | 10.0.2.0/16 | GKE pod IPs | Secondary |
| prod-gke-svc | prod-vpc | 10.0.3.0/20 | GKE service IPs | Secondary |
| prod-compute | prod-vpc | 10.0.4.0/24 | Compute Engine | - |
| prod-serverless | prod-vpc | 10.0.5.0/24 | Serverless VPC connector | - |

### IP Address Management

```mermaid
flowchart TB
    subgraph IPAM["IP Address Space"]
        subgraph GCP_Range["GCP (10.0.0.0/8)"]
            R1[10.0.0.0/16 Production]
            R2[10.1.0.0/16 Non-Production]
            R3[10.2.0.0/16 Shared Services]
            R4[10.3.0.0/16 Edge]
        end
        
        subgraph OnPrem_Range["On-Premises (172.16.0.0/12)"]
            R5[172.16.0.0/16 Data Center]
            R6[172.17.0.0/16 DMZ]
            R7[172.18.0.0/16 Management]
        end
        
        subgraph Reserved["Reserved"]
            R8[192.168.0.0/16 Future]
        end
    end
```

## Hybrid Connectivity

### Interconnect Architecture

```mermaid
flowchart LR
    subgraph GCP["GCP"]
        CR1[Cloud Router 1]
        CR2[Cloud Router 2]
        VLAN1[VLAN Attachment 1]
        VLAN2[VLAN Attachment 2]
    end
    
    subgraph Partner["Interconnect Partner"]
        PE1[Partner Edge 1]
        PE2[Partner Edge 2]
    end
    
    subgraph OnPrem["On-Premises"]
        R1[Router 1]
        R2[Router 2]
        FW[Firewall]
    end
    
    CR1 --> VLAN1
    CR2 --> VLAN2
    VLAN1 --> PE1
    VLAN2 --> PE2
    PE1 --> R1
    PE2 --> R2
    R1 --> FW
    R2 --> FW
```

### Connectivity Options

| Option | Bandwidth | Latency | Use Case | Cost |
|--------|-----------|---------|----------|------|
| **Dedicated Interconnect** | 10-100 Gbps | <5ms | High bandwidth, critical | High |
| **Partner Interconnect** | 50 Mbps - 10 Gbps | <10ms | Medium bandwidth | Medium |
| **Cloud VPN** | Up to 3 Gbps | Variable | Backup, dev/test | Low |
| **Cloud VPN HA** | Up to 3 Gbps | Variable | HA requirements | Low-Medium |

### BGP Configuration

| Peer | ASN | Advertised Routes | Received Routes |
|------|-----|-------------------|-----------------|
| GCP Cloud Router | 16550 | 10.0.0.0/8 | 172.16.0.0/12 |
| On-Prem Router 1 | 65001 | 172.16.0.0/12 | 10.0.0.0/8 |
| On-Prem Router 2 | 65001 | 172.16.0.0/12 | 10.0.0.0/8 |

## Load Balancing

### Load Balancer Architecture

```mermaid
flowchart TB
    subgraph External["External Traffic"]
        USERS[Users]
    end
    
    subgraph GlobalLB["Global Load Balancing"]
        GCLB[Global HTTPS LB]
        ARMOR[Cloud Armor]
        CDN[Cloud CDN]
    end
    
    subgraph RegionalLB["Regional Load Balancing"]
        RLBI[Regional Internal LB]
        RLBE[Regional External LB]
    end
    
    subgraph Backends["Backends"]
        GKE[GKE Services]
        CR[Cloud Run]
        VM[Compute Engine]
    end
    
    USERS --> GCLB
    GCLB --> ARMOR
    ARMOR --> CDN
    CDN --> GKE
    
    RLBI --> GKE
    RLBI --> VM
    RLBE --> CR
```

### Load Balancer Types

| Type | Scope | Use Case | Features |
|------|-------|----------|----------|
| **Global HTTPS** | Global | Web applications | SSL, CDN, Armor |
| **Global SSL Proxy** | Global | SSL traffic | SSL offload |
| **Global TCP Proxy** | Global | TCP traffic | Proxy protocol |
| **Regional External** | Regional | Regional apps | Network LB |
| **Regional Internal** | Regional | Internal services | Private IPs |

## Firewall Architecture

### Firewall Rules

| Priority | Name | Direction | Source | Target | Ports | Action |
|----------|------|-----------|--------|--------|-------|--------|
| 1000 | allow-iap | Ingress | 35.235.240.0/20 | All | 22, 3389 | Allow |
| 1100 | allow-health-check | Ingress | Health check ranges | Tagged | 80, 443 | Allow |
| 1200 | allow-internal | Ingress | 10.0.0.0/8 | 10.0.0.0/8 | All | Allow |
| 1300 | allow-onprem | Ingress | 172.16.0.0/12 | 10.0.0.0/8 | Defined | Allow |
| 65534 | deny-all-ingress | Ingress | 0.0.0.0/0 | All | All | Deny |

### Network Security

```mermaid
flowchart TB
    subgraph Perimeter["Perimeter Security"]
        ARMOR[Cloud Armor WAF]
        DDoS[DDoS Protection]
    end
    
    subgraph Network["Network Security"]
        FW[Firewall Rules]
        VPC_SC[VPC Service Controls]
        PSC[Private Service Connect]
    end
    
    subgraph Service["Service Security"]
        ASM[Anthos Service Mesh]
        mTLS[Mutual TLS]
    end
    
    Perimeter --> Network
    Network --> Service
```

## DNS Architecture

### Cloud DNS Configuration

| Zone Type | Domain | Purpose |
|-----------|--------|---------|
| **Public** | example.com | Public DNS resolution |
| **Private** | internal.example.com | Internal GCP resolution |
| **Forwarding** | onprem.example.com | On-premises resolution |
| **Peering** | shared.example.com | Cross-project resolution |

### DNS Resolution Flow

```mermaid
flowchart LR
    subgraph GCP["GCP"]
        VM[VM Instance]
        DNS[Cloud DNS]
        META[Metadata Server]
    end
    
    subgraph OnPrem["On-Premises"]
        OPDNS[On-Prem DNS]
    end
    
    subgraph Public["Public"]
        PUBDNS[Public DNS]
    end
    
    VM --> META
    META --> DNS
    DNS -->|Forward| OPDNS
    DNS -->|Forward| PUBDNS
    OPDNS -->|Forward| DNS
```

## Service Connectivity

### Private Google Access

| Service | Access Method | Endpoint |
|---------|---------------|----------|
| Cloud Storage | Private Google Access | storage.googleapis.com |
| BigQuery | Private Google Access | bigquery.googleapis.com |
| Cloud SQL | Private IP | Private service connection |
| GKE | Private cluster | Private endpoint |

### VPC Service Controls

```mermaid
flowchart TB
    subgraph Perimeter["VPC Service Control Perimeter"]
        subgraph Protected["Protected Services"]
            BQ[BigQuery]
            GCS[Cloud Storage]
            SQL[Cloud SQL]
        end
        
        subgraph Projects["Allowed Projects"]
            P1[prod-apps]
            P2[prod-data]
        end
    end
    
    subgraph External["Outside Perimeter"]
        DEV[dev-project]
        EXT[External Users]
    end
    
    Protected <--> Projects
    External -.->|Blocked| Protected
```

---

[← Back to Infrastructure](infrastructure.md) | [Next: Security Architecture →](security-architecture.md)
