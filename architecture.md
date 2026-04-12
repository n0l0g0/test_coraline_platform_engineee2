# Architecture Design

## Overview

The system is designed as a microservices-based architecture deployed on AWS.

It consists of:
- Frontend (Next.js)
- Backend APIs
- Workflow orchestration (Apache Airflow)
- Data storage and caching layers

---

## Environment Separation

| Environment | AWS Account |
|------------|------------|
| Development | Account A |
| Staging | Account B |
| Production | Account C |

Each environment is isolated to ensure security and reduce risk.

---

## Core Components

### Frontend
- CloudFront (CDN + static assets)
- Next.js application running on EKS

### Backend
- API services (Blue/Green deployment)
- Apache Airflow (Scheduler + Workers)

### Infrastructure
- EKS (private subnet)
- Application Load Balancer (ALB)
- ECR (container registry)

### Data Layer
- RDS PostgreSQL (private access only)
- ElastiCache Redis
- S3 (object storage)

---

## External Integration

- On-premise database via Site-to-Site VPN
- Third-party APIs via NAT Gateway

---

## Security

- EKS deployed in private subnet
- RDS not publicly accessible
- Secrets managed via AWS Secrets Manager
- IRSA used for pod-level IAM access

---

## Observability

### Monitoring
- Prometheus + Grafana

### Logging
- CloudWatch Logs (via Fluent Bit)

### Alerting
- SNS → Slack / PagerDuty

---

## Reliability

- Health checks via ALB and Kubernetes probes
- Multi-AZ deployment for high availability