# Azure Platform Framework

This repository contains the architecture, standards, accepted decisions,
diagrams, and roadmap for the Ordicor Azure Platform Framework.

The framework is an opinionated Azure Landing Zone platform design. It favors
Terraform, reusable modules, clear repository boundaries, configuration-driven
deployment, least privilege, remote state isolation, and enterprise-first
architecture that can scale down for a lab without changing the model.

## Current Release

Current release: `v0.1.0-alpha`

This release establishes the baseline architecture decisions and standards used
by the implementation repositories. The architecture and engineering baseline
are complete enough for Foundation bootstrap preparation.

## Architecture Overview

The platform is designed around:

- Terraform as the authoritative Infrastructure as Code engine.
- Reusable, independently versioned Terraform modules.
- Separate root deployment repositories for foundation and connectivity.
- Native Terraform remote state using the `azurerm` backend and Azure Blob
  Storage.
- Microsoft Entra ID authentication and Azure RBAC authorization.
- A tenant hierarchy with Platform, Landing Zones, and Decommissioned
  management groups.
- Hub-and-spoke enterprise networking with platform-owned regional hubs.
- Centralized standards for repository structure, modules, naming, tagging,
  versioning, state, and engineering validation.

Current toolchain baseline:

- Terraform execution version: `1.15.8`
- Reusable module Terraform minimum: `>= 1.7.0`
- Reusable module AzureRM constraint: `>= 4.0.0, < 5.0.0`
- Root deployment AzureRM constraint pattern: `~> 4.80`
- First foundation root lock-file-selected AzureRM version: `4.81.0`
- AzAPI: excluded until a real platform capability requires it

Root constraints define the permitted provider range. Committed root lock files
record the exact selected provider version, and future provider updates require
explicit dependency-upgrade pull requests.

Accepted ADRs:

- [ADR 0001 - Infrastructure as Code Engine](docs/adr/0001-iac-engine.md)
- [ADR 0002 - Repository Separation](docs/adr/0002-repository-separation.md)
- [ADR 0003 - Terraform Toolchain Baseline](docs/adr/0003-terraform-toolchain-baseline.md)
- [ADR 0004 - Remote State Strategy](docs/adr/0004-remote-state-strategy.md)
- [ADR 0005 - Management Group Hierarchy](docs/adr/0005-management-group-hierarchy.md)
- [ADR 0006 - Deployment Identity Strategy](docs/adr/0006-deployment-identity-strategy.md)
- [ADR 0007 - Enterprise Networking Strategy](docs/adr/0007-enterprise-networking-strategy.md)
- [ADR 0008 - Root Deployment Repository Structure](docs/adr/0008-root-deployment-repository-structure.md)

## Repository Overview

The accepted repository model uses four repositories.

| Repository | Responsibility |
| --- | --- |
| `azure-platform-architecture` | Architecture, standards, ADRs, diagrams, roadmap, and implementation guidance |
| `azure-platform-modules` | Reusable and independently versioned Terraform modules |
| `azure-platform-foundation` | Management groups, platform subscriptions, governance, identity, management, and observability deployments |
| `azure-platform-connectivity` | Enterprise hubs, routing, firewalls, DNS, hybrid connectivity, and spoke onboarding deployments |

This repository is the source of truth for platform intent and accepted
architecture. Deployable Terraform root modules and reusable module source
belong in the implementation repositories, not here.

## Roadmap Summary

M0 Platform Architecture and the engineering baseline are complete. Foundation
bootstrap is the next milestone.

| Milestone | Status | Outcome |
| --- | --- | --- |
| M0 Architecture and Standards | Complete | Platform requirements, standards, and major architecture decisions are documented |
| M1 Engineering Toolchain | Complete | Local and CI validation expectations are documented and ready for implementation repository use |
| M2 Bootstrap and State | Next | Remote state and deployment identities are established |
| M3 Reusable Module Platform | Initial modules released | Initial reusable modules are tested and versioned |
| M4-M10 Platform Implementation | Not started | Resource organization, governance, observability, networking, landing zones, subscription vending, and production readiness mature over later milestones |

Full roadmap:

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

## Current Milestone

Next milestone: **M2 Bootstrap and State**

Current focus:

- Prepare `azure-platform-foundation` for bootstrap implementation.
- Consume immutable released modules from `azure-platform-modules`.
- Create and migrate the initial Azure Blob Storage remote state backend.
- Use Microsoft Entra ID and Azure RBAC for state access.
- Avoid storage account keys, SAS tokens, client secrets, and local state after
  migration.

Current released modules:

- `resource-group-v0.1.0`
- `storage-account-v0.1.0`
- `storage-container-v0.1.0`

## Getting Started

1. Read [docs/roadmap/00-overview.md](docs/roadmap/00-overview.md).
2. Read [docs/roadmap/01-architecture-and-standards.md](docs/roadmap/01-architecture-and-standards.md).
3. Review the accepted ADR index in [docs/adr](docs/adr).
4. Review standards in [docs/standards](docs/standards), especially the
   repository, Terraform module, versioning, and engineering validation
   standards.
5. For AI-assisted work, read
   [docs/ai/assistant-guide.md](docs/ai/assistant-guide.md) before making
   changes.
