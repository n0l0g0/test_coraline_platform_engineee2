# Platform Architecture & CI/CD Design

This repository contains the architecture design and CI/CD pipeline for a web application platform.

## Overview

The system is designed using a microservices architecture running on AWS, supporting multiple environments:
- Development
- Staging
- Production

Each environment is isolated in a separate AWS account to reduce blast radius.

## Key Technologies

- Kubernetes (EKS)
- Docker
- GitHub Actions (CI/CD)
- AWS Services (ALB, CloudFront, RDS, S3, Redis, ECR)
- Apache Airflow

## Features

- Multi-environment deployment (dev/staging/prod)
- Blue/Green + Canary deployment (production)
- Secure secret management using AWS Secrets Manager + IRSA
- Observability (monitoring, logging, alerting)

## Files

- architecture.md → system architecture explanation
- cicd-flow.md → CI/CD pipeline explanation
- diagrams/architecture.png → architecture diagram

## Assumptions

- All workloads run on EKS
- AWS managed services are preferred to reduce operational overhead
- GitHub Actions is used for CI/CD

## Trade-offs

- EKS is chosen over ECS for flexibility and extensibility
- Blue/Green deployment is used instead of rolling update for safer production releases

## Scaling Strategy

- Horizontal Pod Autoscaler (HPA)
- Cluster Autoscaler

## Failure Handling

- Instant rollback using ALB weight switch
- Airflow retry mechanism for failed jobs