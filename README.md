# Azure Platform Engineering

This repository contains the architecture, standards, decisions, and
implementation roadmap for the organization's Azure Landing Zone platform.

## Objectives

- Build a reusable Azure Landing Zone platform.
- Standardize Terraform and OpenTofu module development.
- Establish enterprise governance and networking.
- Support repeatable subscription and workload landing-zone deployment.
- Minimize hard-coded environment-specific configuration.
- Treat platform infrastructure as a versioned product.

## Repositories

| Repository | Responsibility |
|---|---|
| azure-platform-architecture | Architecture, standards, ADRs, and roadmap |
| azure-platform-modules | Reusable Terraform/OpenTofu modules |
| azure-platform-foundation | Tenant and platform foundation |
| azure-platform-connectivity | Enterprise network platform |

## Roadmap

1. [Roadmap overview](docs/roadmap/00-overview.md)
2. [Architecture and standards](docs/roadmap/01-architecture-and-standards.md)
3. [Engineering toolchain](docs/roadmap/02-engineering-toolchain.md)
4. [Bootstrap and state](docs/roadmap/03-bootstrap-and-state.md)
5. [Reusable modules](docs/roadmap/04-reusable-modules.md)
6. [Resource organization](docs/roadmap/05-resource-organization.md)
7. [Governance and security](docs/roadmap/06-governance-security.md)
8. [Management and observability](docs/roadmap/07-management-observability.md)
9. [Enterprise networking](docs/roadmap/08-enterprise-networking.md)
10. [Application landing zones](docs/roadmap/09-application-landing-zones.md)
11. [Subscription vending](docs/roadmap/10-subscription-vending.md)
12. [Production readiness](docs/roadmap/11-production-readiness.md)

## Current milestone

Milestone 0: Architecture and standards definition.