# Architecture Decision Records

This directory contains the accepted Architecture Decision Records (ADRs) for
the Azure Platform Framework.

ADRs are authoritative architecture decisions. Standards and roadmap documents
must align with accepted ADRs. Empty placeholder ADR files are not decision
records and should not be treated as accepted architecture.

## Accepted ADR Index

| Number | Title | Status | Purpose |
| --- | --- | --- | --- |
| [0001](0001-iac-engine.md) | Infrastructure as Code Engine | Accepted | Selects Terraform as the authoritative and officially supported Infrastructure as Code engine. |
| [0002](0002-repository-separation.md) | Repository Separation | Accepted | Defines the four-repository model and ownership boundaries for architecture, modules, foundation, and connectivity. |
| [0003](0003-terraform-toolchain-baseline.md) | Terraform Toolchain Baseline | Accepted | Establishes the approved Terraform execution version, reusable module compatibility floor, AzureRM baseline, root constraint pattern, and AzAPI posture. |
| [0004](0004-remote-state-strategy.md) | Remote State Strategy | Accepted | Selects Azure Blob Storage with the native Terraform `azurerm` backend, Entra ID authentication, Azure RBAC, bootstrap migration, and initial state boundaries. |
| [0005](0005-management-group-hierarchy.md) | Management Group Hierarchy | Accepted | Defines the target tenant, platform, landing-zone, and decommissioned management group hierarchy. |
| [0006](0006-deployment-identity-strategy.md) | Deployment Identity Strategy | Accepted | Defines human bootstrap access, repository-owned automation identities, GitHub OIDC, workload identity federation, plan/apply separation, and break-glass expectations. |
| [0007](0007-enterprise-networking-strategy.md) | Enterprise Networking Strategy | Accepted | Defines hub-and-spoke as the reference networking architecture, with platform-owned regional hubs, centralized routing, DNS, and private endpoint first connectivity. |
| [0008](0008-root-deployment-repository-structure.md) | Root Deployment Repository Structure | Accepted | Defines the standard `platform/`, `environments/`, and `docs/` layout for deployment repositories, including Foundation bootstrap placement. |

## Removed Placeholders

The following empty duplicate-number placeholders were removed because they
contained no accepted decisions:

- `0003-module-composition.md`
- `0004-azure-verified-modules.md`
- `0006-subscription-model.md`

No accepted ADR content was removed.

## Deferred Decisions

The absence of an ADR for a topic does not imply an accepted decision. Deferred
topics must remain governed by accepted ADRs, standards, or explicit future
architecture work before implementation depends on them.

Known deferred topics include:

- Final subscription creation and vending model.
- Final IPAM operating model.
- Detailed DNS implementation and ownership lifecycle.
- Governance and policy rollout details.
- Final pipeline implementation details.
- Final naming convention and tag value catalogs.
