# AI Assistant Guide

## Purpose

This document is the onboarding and operating guide for AI assistants and new
contributors working on the Azure Platform Framework.

It helps a new AI assistant or engineer become productive without relying on
prior chat history. It summarizes the project context, accepted decisions,
repository boundaries, and expected workflow, but it does not replace the
authoritative roadmap, ADRs, and engineering standards.

## Source of Truth

When sources conflict, use this authority order:

1. Accepted ADRs.
2. Engineering standards.
3. Roadmap and milestone documents.
4. Repository README and contributor guidance.
5. Current implementation.
6. Chat instructions.

Chat history is temporary. Git is authoritative. If a chat instruction appears
to conflict with an accepted ADR or standard, stop and identify the conflict
instead of silently changing the architecture.

## Project Goals

The Azure Platform Framework defines and implements an enterprise Azure Landing
Zone platform as a versioned platform product.

The platform goals are to:

- Build a reusable Azure Landing Zone foundation.
- Use Terraform as the authoritative infrastructure as code engine.
- Standardize reusable, versioned Terraform modules.
- Keep reusable modules separate from environment deployments.
- Support configuration-driven foundation, connectivity, subscription, and
  workload landing-zone deployment.
- Minimize hard-coded environment-specific configuration.
- Establish governance, security, observability, networking, DNS, identity, and
  subscription patterns before production deployment.
- Make platform changes testable, reviewable, recoverable, and operationally
  supportable.

## Current Project Status

Current release: `v0.1.0-alpha`

Current milestone: **M1 - Engineering Toolchain**

M0 Platform Architecture is complete. The architecture repository now contains
the accepted baseline ADRs and standards needed to begin engineering toolchain
implementation.

Current status:

- ADRs `0001` through `0007` are accepted.
- Terraform is the authoritative Infrastructure as Code engine.
- The four-repository model is accepted.
- Remote state, management group hierarchy, deployment identity, and enterprise
  networking decisions are accepted.
- Repository, Terraform module, naming, tagging, versioning, state management,
  and engineering validation standards exist.
- No production Azure platform resources should be deployed from this
  repository.
- M1 work should focus on local validation, CI validation, tooling, and
  repeatable engineering workflows.

## Repository Map

The accepted repository model uses four repositories.

### azure-platform-architecture

Responsibility:

- Roadmap.
- Architecture documents.
- ADRs.
- Engineering standards.
- Diagrams.
- Implementation guidance.
- Decision backlog and milestone definitions.

Prohibited content:

- Deployable Terraform root modules.
- Reusable Terraform module source.
- Terraform state files, state backups, or plan files.
- Provider lock files for deployable root modules.
- Azure tenant IDs, subscription IDs, secrets, private keys, or environment
  configuration.
- CI/CD workflows that apply Azure infrastructure.
- Generated deployment artifacts.

### azure-platform-modules

Responsibility:

- Reusable Terraform modules.
- Module examples.
- Module tests.
- Module documentation.
- Module release notes.
- Independent semantic module releases.
- Azure Verified Module wrappers or compositions where approved.

Prohibited content:

- Real environment root deployments.
- Environment-specific tenant IDs, subscription IDs, regions, CIDR ranges,
  resource names, or tag values.
- Remote backend configuration for real platform deployments.
- Terraform state files or plan files.
- Credentials, secrets, private keys, or pipeline secrets.
- Hard-coded enterprise IP allocations.
- Foundation or connectivity environment configuration.

### azure-platform-foundation

Responsibility:

- Management group root deployments.
- Platform subscription placement and enrollment configuration.
- Governance and Azure Policy assignments.
- Identity and RBAC assignments.
- Management and observability resources.
- Foundation environment configuration.
- Foundation state boundaries and deployment pipelines.

Prohibited content:

- Reusable module source that belongs in `azure-platform-modules`.
- Enterprise hub, firewall, routing, DNS, hybrid connectivity, or spoke
  onboarding deployments unless a documented dependency requires coordination.
- Terraform state files, state backups, or committed plan files.
- Secrets, private keys, client secrets, or local developer overrides.

### azure-platform-connectivity

Responsibility:

- Enterprise network hub root deployments.
- Routing, firewalls, firewall policy, DNS, private resolver, hybrid
  connectivity, and spoke onboarding.
- Connectivity environment configuration.
- Connectivity state boundaries and deployment pipelines.

Prohibited content:

- Reusable module source that belongs in `azure-platform-modules`.
- Management group hierarchy, global policy baseline, or platform identity
  deployments unless explicitly documented.
- Terraform state files, state backups, or committed plan files.
- Secrets, private keys, client secrets, or local developer overrides.

## Current Decisions

Accepted decisions only:

- [ADR 0001 - Infrastructure as Code Engine](../adr/0001-iac-engine.md):
  Terraform is the authoritative and officially supported IaC engine.
- [ADR 0002 - Repository Separation](../adr/0002-repository-separation.md):
  the platform uses the four-repository model:
  `azure-platform-architecture`, `azure-platform-modules`,
  `azure-platform-foundation`, and `azure-platform-connectivity`.
- [ADR 0003 - Terraform Toolchain Baseline](../adr/0003-terraform-toolchain-baseline.md):
  Terraform `1.15.8` is the exact approved execution version, AzureRM `4.81.0`
  is the initial release-validation and lock-file-selected provider version,
  and AzAPI is excluded until a real capability requires it.
- [ADR 0004 - Remote State Strategy](../adr/0004-remote-state-strategy.md):
  the platform uses the native Terraform `azurerm` backend with Azure Blob
  Storage, Azure RBAC, and coarse initial state boundaries.
- [ADR 0005 - Management Group Hierarchy](../adr/0005-management-group-hierarchy.md):
  the hierarchy is Tenant, Platform, Landing Zones, and Decommissioned, with
  Platform containing Management, Identity, Connectivity, and Shared Services.
- [ADR 0006 - Deployment Identity Strategy](../adr/0006-deployment-identity-strategy.md):
  human identities are for bootstrap, development, troubleshooting, and
  emergency operations; routine deployments use automation identities with
  GitHub OIDC and Microsoft Entra Workload Identity Federation.
- [ADR 0007 - Enterprise Networking Strategy](../adr/0007-enterprise-networking-strategy.md):
  hub-and-spoke is the reference architecture, with platform-owned regional
  hubs, centralized routing, centralized DNS, and private endpoint first
  connectivity.

Additional accepted toolchain facts:

- Terraform `1.15.8` is the exact approved execution version for developer
  tooling, CI/CD, tests, plans, applies, release validation, state operations,
  troubleshooting, and runbooks.
- Reusable modules support Terraform `>= 1.7.0`, subject to validation before
  stable releases claim that compatibility.
- AzureRM release validation initially uses AzureRM `4.81.0`.
- Reusable modules support AzureRM `>= 4.0.0, < 5.0.0`, subject to validation
  before stable releases claim that compatibility.
- Root deployment root constraints use the approved pattern `~> 4.80`.
- Committed root dependency lock files record the exact selected provider
  version.
- The first foundation root lock file selected AzureRM `4.81.0`.
- Future provider updates require explicit dependency-upgrade pull requests.
- AzAPI is not included until a real platform capability requires it.
- Root repositories own remote state, provider configuration, backend
  configuration, dependency lock files, and environment values.
- Reusable child modules remain environment-neutral.

Do not invent decisions for unresolved architecture areas. If a decision is not
accepted in an ADR or standard, describe it as unresolved or deferred.

## Engineering Philosophy

The platform favors:

- Configuration-driven deployments.
- Explicit module and repository contracts.
- Composition over giant modules.
- No hidden environment behavior.
- Secure defaults where a module can safely choose a default.
- Predictable Terraform plans.
- Small blast radius through repository, state, identity, and pipeline
  boundaries.
- Immutable module references from deployment repositories.
- Standards before implementation when decisions affect multiple repositories.
- Implementation feedback that improves standards through deliberate updates.

Reusable modules are product artifacts. Root deployments are consumers of those
artifacts. Environment-specific values belong in deployment repositories, not in
reusable module source.

## Recommended Startup Workflow

1. Read this guide.
2. Read the relevant ADRs and standards for the requested change.
3. Inspect the actual repository and Git status.
4. Confirm the current release, active milestone, and accepted ADRs that apply.
5. Summarize the current state before modifying files when the task is broad or
   architecture-sensitive.
6. Make only the requested changes.
7. Do not broaden scope.
8. Report files changed, validation performed, blockers, and deferred work.
9. Never claim tests passed if tools were unavailable or were not run.
10. Do not commit or push unless explicitly asked.
11. Do not silently make architecture decisions.

If the requested change exposes an unresolved architecture decision, document
the decision point and ask for direction or propose an ADR. Do not hide the
decision in implementation.

## Repository-Specific Guardrails

Architecture repository guardrails:

- No deployable Terraform.
- No Terraform state.
- No provider lock files for deployable roots.
- No environment values.
- No secrets or generated deployment artifacts.

Modules repository guardrails:

- No backend configuration in child modules.
- No provider configuration in child modules.
- No `tfvars` for real environments.
- No real tenant, subscription, region, CIDR, name, credential, owner, support,
  or cost values.
- Examples may act as root modules for validation and documentation.
- Released modules use immutable semantic versions.

Foundation and connectivity repository guardrails:

- Root modules may configure providers and backends.
- Environment configuration belongs in these repositories.
- State boundaries must follow approved ADRs and standards.
- Do not copy reusable module logic into root deployments.
- Consume reusable modules through immutable tags or commit SHAs, not mutable
  branches.

## Current Implementation Status

This is a project snapshot and must be updated when milestones change:

- `v0.1.0-alpha` has been released.
- M0 Platform Architecture is complete.
- M1 Engineering Toolchain is active.
- ADRs `0001` through `0007` are accepted.
- Repository, Terraform module, naming, tagging, versioning, state management,
  and engineering validation standards exist.
- Engineering Validation Standard has been added.
- The architecture repository remains documentation-only and must not contain
  deployable Terraform, state, plan files, or environment-specific values.
- Implementation repositories are expected to consume this architecture
  baseline as M1 and later milestones proceed.
- Foundation and connectivity repositories are not yet implemented.

## Current Engineering Priorities

The immediate priority is M1 Engineering Toolchain.

Current priorities:

- Translate the Engineering Validation Standard into repeatable local and
  GitHub Actions validation workflows.
- Standardize `terraform fmt`, `terraform validate`, `terraform test`, TFLint,
  and Trivy checks.
- Align CI behavior with the approved Terraform execution version and provider
  strategy.
- Define pull request and release validation evidence for reusable modules.
- Keep workflow implementation details in the appropriate implementation
  repositories.
- Preserve the M0 architecture baseline unless a deliberate ADR or standard
  update changes it.

Do not expand implementation scope before the M1 validation model is clear and
reviewable.

## Unresolved Decisions

The following items remain unresolved unless a newer accepted ADR or standard
states otherwise:

- Subscription model.
- IPAM.
- DNS ownership.
- Governance and policy rollout.
- CI/CD implementation.
- TFLint and security scanner configuration.
- Example lock-file policy.
- Tag key casing and controlled values.
- Final naming convention.

Do not decide these implicitly while writing Terraform, documentation, tests,
or examples.

## Change Discipline

Use the smallest durable change that fits the request.

Create or update an ADR when:

- A decision changes repository boundaries, state ownership, deployment
  identity, provider strategy, management group structure, network topology,
  DNS ownership, policy rollout, or another architecture-level contract.
- Multiple repositories must follow the decision.
- The decision needs consequences, tradeoffs, and revisit conditions.

Update a standard when:

- A reusable engineering rule changes.
- Module, repository, naming, tagging, versioning, testing, pipeline, or state
  behavior changes.
- A checklist or definition of done is no longer accurate.

Make an implementation change when:

- The relevant ADRs and standards already authorize the pattern.
- The requested work is within the responsibility of the current repository.
- Required inputs, ownership, and validation expectations are clear.

Update roadmap or backlog documents when:

- A milestone status changes.
- A new dependency, risk, or unresolved decision is discovered.
- Work moves between milestones or requires follow-up.

## New Session Starter

Copy this prompt into a new Codex session:

```text
Read docs/ai/assistant-guide.md first. Then read the relevant ADRs and
standards for the task, inspect the repository and Git status, and make no
changes yet. Summarize the current project state, the authoritative decisions
that apply, any unresolved decisions, and the next recommended step.
```

## Review Checklist

- [ ] The assistant read this guide before acting.
- [ ] Relevant ADRs and standards were read.
- [ ] Git status and repository contents were inspected.
- [ ] The current state was summarized before file edits.
- [ ] The change stayed within the requested scope.
- [ ] No prohibited content was added to the repository.
- [ ] No architecture decision was made silently.
- [ ] Terraform was not created in the architecture repository.
- [ ] Environment-specific values, secrets, state, plan files, and provider
      lock files were not added where prohibited.
- [ ] Files changed were reported.
- [ ] Validation performed was reported accurately.
- [ ] Blockers and deferred work were identified.
- [ ] No commit or push was performed unless explicitly requested.

## Definition of Done

A new assistant or contributor is correctly grounded when they can:

- Identify the source-of-truth hierarchy.
- Explain the four-repository model and repository guardrails.
- Summarize accepted ADR decisions without inventing unresolved decisions.
- Distinguish reusable module responsibilities from root deployment
  responsibilities.
- Apply Terraform, AzureRM, AzAPI, naming, tagging, and versioning standards
  consistently.
- Inspect the repository before changing it.
- Keep changes scoped to the request.
- Report validation honestly.
- Recognize when a change requires an ADR, standard update, roadmap update, or
  implementation change.
