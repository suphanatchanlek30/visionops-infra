# VisionOps Infra

Infrastructure, automation, cloud architecture, observability, security, reliability engineering, and technical documentation for **VisionOps Reliability Platform**.

## Overview

VisionOps Reliability Platform is a production-oriented DevOps, Cloud, Platform Engineering, and SRE learning portfolio project.

This repository contains the infrastructure design, architecture documentation, security baseline, cost controls, operational checklists, and future Infrastructure as Code for the VisionOps platform.

The project uses a **Synthetic Vision Event Processing Platform** as the workload for learning and validating cloud-native operations.

Core workload components include:

- Event API
- Worker
- Queue
- Database
- Object Storage
- Dashboard
- Event Generator
- Observability Stack
- CI/CD and GitOps workflow

This repository does **not** contain real customer data, real CCTV data, credentials, secrets, or production infrastructure state.

---

## Repository Role

VisionOps is organized into three repositories:

| Repository | Responsibility |
|---|---|
| `visionops-app` | Application source code for API, Worker, Dashboard, Event Generator, and tests |
| `visionops-infra` | Infrastructure, architecture, security, cost, reliability, and operations documentation |
| `visionops-gitops` | Kubernetes desired state, Helm charts, Argo CD applications, and environment configuration |

This repository is the main home for:

- AWS architecture
- Terraform planning
- Security design
- Threat modeling
- Cost control
- Teardown checklist
- SRE documentation
- Operational evidence
- Architecture Decision Records

---

## Current Status

| Item | Status |
|---|---|
| Current phase | M0 — Safety & Design |
| M0 status | Completed |
| Cloud resources created in M0 | No |
| AWS budget configured | Yes |
| Root MFA baseline | Completed |
| Root access keys | 0 |
| Sensitive details committed | No |
| Primary cloud | AWS |
| Primary region | ap-southeast-1 |
| Budget hard cap | 100 USD |
| Operating target | 90 USD |

---

## M0 — Safety & Design Baseline

M0 establishes the safety and design foundation before creating real cloud infrastructure.

Completed M0 work includes:

| Issue | Purpose |
|---|---|
| `[M0] Bootstrap GitHub repositories and project governance` | Create repositories, project board, fields, and governance structure |
| `[M0] Approve VisionOps project charter and scope boundaries` | Define project scope, goals, constraints, roadmap, and guardrails |
| `[M0] Secure AWS root account and verify no root access keys` | Confirm root MFA and no root access keys |
| `[M0] Create AWS budget and cost alert thresholds` | Set AWS cost alerts before starting cloud work |
| `[M0] Record tool and platform version matrix` | Record local tool versions for reproducibility |
| `[M0] Define functional and non-functional requirements` | Define system behavior and quality requirements |
| `[M0] Create system architecture and network diagrams` | Create system, runtime, delivery, observability, and network architecture |
| `[M0] Write ADR-001: EKS versus ECS/Fargate` | Record why EKS is selected for the learning lab |
| `[M0] Create initial threat model and data classification` | Define assets, data classification, trust boundaries, and threats |
| `[M0] Define AWS cost envelope and teardown checklist` | Define cost limits, expensive resource watchlist, and teardown checklist |

---

## Documentation Map

| Document | Purpose |
|---|---|
| [`docs/project-charter.md`](docs/project-charter.md) | Project charter, scope, roadmap, constraints, and success criteria |
| [`docs/version-matrix.md`](docs/version-matrix.md) | Local development and platform tool versions |
| [`docs/requirements/functional-requirements.md`](docs/requirements/functional-requirements.md) | Functional requirements for the VisionOps workload |
| [`docs/requirements/non-functional-requirements.md`](docs/requirements/non-functional-requirements.md) | Security, reliability, observability, performance, and cost requirements |
| [`docs/architecture/system-architecture.md`](docs/architecture/system-architecture.md) | System context, runtime, CI/CD, GitOps, infrastructure, and observability architecture |
| [`docs/architecture/network-architecture.md`](docs/architecture/network-architecture.md) | Network flow, request path, worker flow, failure flow, and trust boundaries |
| [`docs/adrs/ADR-001-compute-platform.md`](docs/adrs/ADR-001-compute-platform.md) | Architecture decision record for EKS versus ECS/Fargate |
| [`docs/security/threat-model.md`](docs/security/threat-model.md) | Threat model, data classification, assets, and security controls |
| [`docs/cost/aws-cost-envelope.md`](docs/cost/aws-cost-envelope.md) | AWS budget envelope and cost control rules |
| [`docs/cost/teardown-checklist.md`](docs/cost/teardown-checklist.md) | AWS teardown and orphan resource audit checklist |

---

## Safety Rules

This repository is public, so the following data must never be committed:

- AWS access keys
- AWS secret access keys
- AWS session tokens
- Root credentials
- MFA QR codes or MFA secrets
- `.env` files
- Private keys such as `.pem` or `.key`
- kubeconfig files
- Terraform state files
- Terraform plan files containing sensitive values
- Database passwords
- Billing details
- Payment information
- Real CCTV data
- Real personal image data
- Company or internship confidential data

Only sanitized evidence should be committed.

---

## Cost Control Rules

VisionOps uses AWS with strict cost discipline.

| Rule | Value |
|---|---|
| Hard cap | 100 USD |
| Operating target | 90 USD |
| Safety reserve | 10 USD |
| Strategy | Local-first |
| Cloud usage | Short-lived sessions |
| EKS usage | Create → Validate → Collect evidence → Destroy |

AWS Budget is treated as a **notification guardrail**, not a spending hard stop.

Before every cloud session, the teardown checklist must be reviewed.

---

## Planned Technical Direction

Future phases will introduce:

- Linux and network operations lab
- Event API and Worker implementation
- Docker and container hardening
- Terraform-based AWS infrastructure
- Local Kubernetes using kind
- Kubernetes manifests and Helm
- GitHub Actions CI/CD
- GitOps with Argo CD
- Short-lived AWS EKS validation
- Observability with Prometheus, Grafana, logs, traces, and alerts
- SRE GameDay and recovery evidence
- Optional AI/CV inference worker extension

---

## Roadmap

| Phase | Focus |
|---|---|
| M0 | Safety, design, requirements, architecture, security, cost baseline |
| M1 | Linux and Network Operations Foundation Lab |
| M2 | Application and Container Baseline |
| M3 | Terraform and AWS Foundation |
| M4 | Kubernetes Platform Baseline |
| M5 | CI/CD and GitOps |
| M6 | AWS EKS Integration |
| M7 | Observability and SRE |
| M8 | GameDay, Backup, Restore, and Recovery |
| M9 | Portfolio Release |
| M10 | Optional AI/CV Operations Extension |

---

## Non-Goals

This project is not intended to be:

- A production service for real users
- A real CCTV analytics platform
- A real customer data platform
- A payment system
- A full enterprise security compliance system
- A multi-region production architecture
- A commercial SaaS product

The goal is to build a realistic learning platform with strong engineering evidence.

---

## License

This repository is licensed under the Apache License 2.0.