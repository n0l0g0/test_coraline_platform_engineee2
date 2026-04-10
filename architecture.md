# Architecture Diagram
## Internal Web Application — Multi-Service System

```mermaid
graph TB
    subgraph Users
        U[Browser / Internal Users]
    end

    subgraph AWS Cloud
        subgraph Public Layer
            CF[CloudFront\nCDN + Static Assets]
            ALB[Application Load Balancer]
        end

        subgraph EKS Cluster - Private Subnet
            WEB[Web Frontend\nNext.js]
            API[API Service\nFastAPI]
            subgraph Airflow
                SCH[Airflow Scheduler]
                WKR[Airflow Workers\nCeleryExecutor]
            end
        end

        subgraph Data Layer - Private Subnet
            RDS[(RDS PostgreSQL)]
            REDIS[(ElastiCache Redis\nCache + Celery Broker)]
            S3[(S3\nDAGs / Files / Artifacts)]
        end

        subgraph Observability
            CW[CloudWatch Logs\nFluent Bit DaemonSet]
            GRAF[Prometheus + Grafana]
            SNS[SNS\nSlack / PagerDuty]
        end

        ECR[ECR\nContainer Registry]
        SM[AWS Secrets Manager]
    end

    subgraph External Systems
        EXTDB[(On-premise Database\nvia Site-to-Site VPN)]
        EXTAPI[3rd Party APIs\nvia NAT Gateway]
    end

    U -->|HTTPS| CF -->|HTTP| ALB -->|port 3000| WEB
    U -->|HTTPS /api| ALB -->|port 8000| API
    API --> RDS
    API --> REDIS
    API -->|IRSA| SM
    API --> EXTAPI
    SCH --> WKR
    WKR --> RDS
    WKR --> S3
    WKR -->|IRSA| SM
    WKR --> EXTDB
    EKS Cluster - Private Subnet -->|logs| CW
    EKS Cluster - Private Subnet -->|metrics| GRAF
    GRAF -->|alert| SNS
    ECR -->|pull image| EKS Cluster - Private Subnet
```

---

## Security Layers

```
Internet
    │
    ▼
CloudFront (WAF + DDoS protection)
    │
    ▼
ALB Security Group  ← allow 80/443 from 0.0.0.0/0
    │
    ▼
EKS Node Security Group  ← allow traffic from ALB only
    │
    ├── API Pod  ←─────────────────────────────────────────┐
    │     │                                                │
    │     ▼                                               IAM / IRSA
    │   RDS Security Group  ← allow 5432 from EKS only    │
    │   Redis Security Group ← allow 6379 from EKS only   │
    │                                                      │
    └── Airflow Worker Pod ────────────────────────────────┘
          │
          ▼
        VPN Tunnel → On-premise DB
```

## Traffic Flow

| Path | Route | Note |
|------|-------|------|
| User → Web | CloudFront → ALB → Web Pod | Static assets cached ที่ CloudFront |
| User → API | ALB → API Pod | JWT auth ตรวจที่ API |
| API → DB | API Pod → RDS (private) | ไม่ผ่าน internet |
| Airflow → On-premise | Worker Pod → VPN → On-premise DB | Encrypted tunnel |
| Secrets | Pod → Secrets Manager via IRSA | ไม่มี hardcode ที่ไหน |
