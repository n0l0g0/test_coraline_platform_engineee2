# CI/CD Pipeline Design

## Overview

The CI/CD pipeline is implemented using GitHub Actions.

It supports:
- Automated build and deployment
- Environment promotion
- Safe production release using Canary deployment

---

## Pipeline Flow

Feature branch → Dev → Merge PR → Staging → Tag release → Production

---

## Development Environment

### Trigger
- Push to feature branch

### Steps
1. Build Docker image
2. Push image to ECR
3. Deploy to EKS (dev namespace)

---

## Staging Environment

### Trigger
- Push to main branch

### Steps
1. Build Docker image
2. Run integration tests
3. Deploy to EKS (staging namespace)

---

## Production Environment

### Trigger
- Git tag (v*.*.*)

### Steps
1. Manual approval
2. Build and push image to ECR
3. Deploy Green version to EKS
4. Start Canary deployment

---

## Canary Deployment Strategy

### Concept

- Blue = current stable version
- Green = new version

---

### Traffic Shifting

Step 1:
- 90% → Blue
- 10% → Green

Step 2:
- 50% → Blue
- 50% → Green

Step 3:
- 0% → Blue
- 100% → Green

---

### Implementation

- ALB weighted routing
- Two target groups:
  - tg-blue
  - tg-green

---

### Monitoring During Canary

- Error rate
- Latency
- CPU / Memory
- API success rate

---

### Rollback Strategy

If issues occur:
- Switch ALB weight back to Blue immediately
- No redeploy required

---

### Safety Checks

- ALB health check
- Kubernetes readiness/liveness probes
- No critical alerts from monitoring system

---

## Promotion Strategy

- Dev → Staging: Merge Pull Request
- Staging → Production: Release tag (v*.*.*)

---

## Benefits

- Zero downtime deployment
- Reduced risk
- Fast rollback
- Production stability