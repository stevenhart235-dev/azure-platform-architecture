# ADR 0008 - Root Deployment Repository Structure

## Status

Accepted

## Decision Date

2026-07-16

## Owners

Ordicor Platform

## Context

The Azure Platform Framework separates architecture, reusable modules, and
deployment repositories.

ADR 0001 establishes Terraform as the authoritative Infrastructure as Code
engine. ADR 0002 establishes separate repositories for architecture, reusable
modules, foundation deployments, and connectivity deployments. ADR 0003
establishes the Terraform toolchain baseline and confirms that root deployment
repositories own provider configuration, backend configuration, dependency lock
files, and environment values. ADR 0004 establishes the remote state strategy.
ADR 0005 establishes the management group hierarchy. ADR 0006 establishes the
deployment identity strategy. ADR 0007 establishes the enterprise networking
strategy and the connectivity repository boundary.

Deployment repositories require a consistent structure that:

- Scales from a single lab to enterprise environments.
- Minimizes duplicated Terraform code.
- Clearly separates reusable deployment logic from environment configuration.
- Supports remote state and CI/CD.
- Aligns with accepted repository separation and remote state strategies.

The repository standard intentionally deferred the final deployment repository
directory structure until remote state, deployment identity, and deployment
architecture decisions were clear enough to avoid inventing patterns in
implementation repositories.

This ADR defines the standard structure for Azure Platform deployment
repositories, including `azure-platform-foundation`,
`azure-platform-connectivity`, and future landing-zone deployment repositories.
It does not create Terraform code, backend configuration, workflows, Azure
resources, or implementation repository files.

## Decision Drivers

- Deployment repositories must remain consistent across foundation,
  connectivity, and future landing-zone scopes.
- Environment-specific configuration must be separated from reusable
  deployment logic.
- Terraform code should not be copied per environment when configuration can
  drive differences.
- Root deployments must support remote state boundaries, plan/apply workflows,
  and controlled CI/CD.
- Repository layout must keep ownership and review scope obvious.
- The lab must be able to start small without creating a lab-only repository
  pattern.
- Future enterprise environments must be able to add nonproduction and
  production configuration without restructuring the repository.
- Reusable child module source must remain in `azure-platform-modules`.

## Decision

Adopt the following standard deployment repository layout:

```text
repository/
|
|-- platform/
|   |-- bootstrap/
|   |-- management-groups/
|   |-- governance/
|   |-- identity/
|   |-- management/
|   `-- ...
|
|-- environments/
|   |-- lab/
|   |-- nonprod/
|   `-- prod/
|
|-- docs/
|
`-- README.md
```

The `platform/` directory contains reusable root-deployment components for the
repository responsibility. These are root-deployment building blocks, not
reusable child modules. They may configure providers and backends only where
they are actual Terraform root modules for an approved state boundary.

The `environments/` directory contains environment configuration, environment
selection, and state-boundary inputs for lab, nonproduction, production, or
future approved environments. Environments must not be represented through
long-lived Git branches.

The `docs/` directory contains repository-local deployment documentation,
runbooks, operational notes, and AI assistant guidance. Authoritative ADRs and
standards remain in `azure-platform-architecture`.

The repository root contains common repository files such as `README.md`,
`CONTRIBUTING.md`, `SECURITY.md`, `CODEOWNERS`, `.editorconfig`, `.gitignore`,
and pull request templates according to the repository standard.

## Directory Semantics

### `platform/`

`platform/` contains deployment implementation units for the repository's
platform responsibility.

For `azure-platform-foundation`, expected subdirectories include:

- `bootstrap/`
- `management-groups/`
- `governance/`
- `identity/`
- `management/`

Additional foundation subdirectories may be added when they map to an accepted
foundation responsibility, such as observability, platform subscription
placement, or other approved foundation state boundaries.

For `azure-platform-connectivity`, `platform/` should contain connectivity
implementation units such as regional hubs, routing, firewall policy, DNS,
hybrid connectivity, private resolver, or spoke onboarding when those
implementation details are deliberately defined.

For future landing-zone deployment repositories, `platform/` should contain the
deployment units required to create or enroll landing-zone scopes according to
the accepted landing-zone and subscription vending design.

### `environments/`

`environments/` contains configuration for environment instances.

Initial standard environment names are:

- `lab`
- `nonprod`
- `prod`

Repositories may start with only `lab` when implementation is limited to the
Ordicor Platform Lab. Empty environment directories should not be created only
for symmetry if they do not contain useful configuration or documentation.

Environment directories provide configuration to deployment units under
`platform/`. They must not contain copied Terraform implementations for each
environment when the same deployment logic can be reused with different
configuration.

Environment configuration may include values such as environment label,
region, subscription placement, feature enablement, policy scope, address
allocation, and state boundary inputs when those values are appropriate for the
deployment repository and authorized by accepted ADRs and standards.

Environment configuration must not contain secrets, credentials, local
developer overrides, Terraform state, committed plan files, or unapproved
sensitive values.

### `docs/`

`docs/` contains local operational and contributor documentation for the
deployment repository.

Examples include:

- Deployment notes.
- Runbooks.
- State boundary documentation.
- Bootstrap procedure documentation.
- Recovery notes.
- Repository-local AI assistant guidance.

Architecture decisions and reusable standards remain in
`azure-platform-architecture`. Repository-local documentation must reference
authoritative standards rather than redefining them.

## Root Deployment Rules

Root deployment repositories own execution reproducibility.

Required rules:

- Configure providers only in root modules.
- Configure backends only in root modules.
- Commit `.terraform.lock.hcl` for real root deployments.
- Use Terraform as the authoritative execution engine.
- Use the approved Terraform execution version for CI, plans, applies, release
  validation, and state operations.
- Consume reusable modules through immutable tags or approved commit SHAs.
- Do not consume mutable module branches such as `main` for production
  deployment references.
- Do not copy reusable child module logic from `azure-platform-modules`.
- Keep environment-specific values in deployment repositories, not reusable
  module source.
- Do not commit Terraform state, state backups, plan files, secrets,
  credentials, local overrides, or generated sensitive artifacts.

## State Alignment

The repository structure must support the remote state strategy in ADR 0004.

State boundaries are represented by root deployments and backend configuration,
not by duplicating Terraform code per environment. The initial state model is
coarse and may be split later based on lifecycle, blast radius, ownership,
deployment cadence, and recovery requirements.

The `bootstrap/` deployment unit is special because bootstrap may begin with
local state before backend resources exist. After backend creation, bootstrap
state must be migrated to Azure Storage and normal operations must not remain
dependent on local state.

Use of `terraform_remote_state` is not the default integration pattern. Any use
must be documented as an intentional cross-state dependency according to
ADR 0004.

## Identity And CI/CD Alignment

The repository structure must support the deployment identity strategy in
ADR 0006.

Each deployment repository owns its deployment identity model. Plan and apply
automation must be separable by workflow, trust condition, and RBAC where
practical.

CI/CD workflows, when implemented, should be able to target specific platform
deployment units and environments without relying on long-lived environment
branches. Pull requests that affect deployable infrastructure must identify
the target environment, state boundary, and expected blast radius.

This ADR does not define workflow file layout, exact GitHub environment names,
federated credential subjects, backend configuration files, or command
wrappers. Those details remain implementation work and must align with the
accepted ADRs and standards.

## Rationale

The layout separates two concerns that must remain distinct:

- `platform/` contains reusable deployment logic and state-boundary root
  modules for the repository responsibility.
- `environments/` contains configuration for lab, nonproduction, production,
  or future approved environment instances.

This separation lets the lab start with a small configuration footprint while
preserving the enterprise target model. It avoids a directory-per-environment
copy of Terraform implementation, which would make drift likely and make
changes harder to review.

The layout also aligns with the four-repository model. Foundation,
connectivity, and future landing-zone deployment repositories can share a
recognizable shape while still owning different platform responsibilities,
state boundaries, deployment identities, CODEOWNERS, and CI/CD approval paths.

Keeping `docs/` in every deployment repository provides a local home for
runbooks and operational notes without moving authoritative ADRs and standards
out of the architecture repository.

## Consequences

- Deployment repositories have a shared top-level structure.
- Implementation work can start with `lab` and expand to `nonprod` and `prod`
  without changing the repository model.
- Terraform implementation should be reused across environments through
  configuration instead of copied per environment.
- Root deployment components have a clear home under `platform/`.
- Environment-specific values have a clear home under `environments/`.
- Local operational documentation has a clear home under `docs/`.
- Future foundation, connectivity, and landing-zone CI/CD can target
  environment and state-boundary paths consistently.
- Repository-local documentation must continue to reference authoritative ADRs
  and standards rather than fork them.

## Risks

- Contributors may treat `platform/` as a reusable child module library unless
  repository documentation is clear.
- Environment directories may accumulate copied Terraform if reviews do not
  enforce configuration-driven reuse.
- The initial coarse state layout may need splitting as deployment cadence and
  blast radius become clearer.
- `bootstrap/` can become a one-off manual process unless repeatability and
  state migration are enforced during M2.
- CI/CD path filters may become brittle if deployment units and environments
  are not documented consistently.
- Future landing-zone repositories may need additional structure after
  subscription vending and landing-zone archetype decisions are accepted.

## Validation Criteria

This decision is valid when:

- Deployment repositories use `platform/`, `environments/`, and `docs/` as the
  standard top-level implementation structure.
- `platform/` contains root-deployment implementation units for the
  repository responsibility.
- `environments/` contains environment configuration rather than copied
  Terraform implementations.
- Initial environment naming supports `lab`, `nonprod`, and `prod`.
- Environment branches are not used as the environment model.
- Reusable child module source remains in `azure-platform-modules`.
- Root deployments own provider configuration, backend configuration, and
  `.terraform.lock.hcl`.
- Deployment repositories continue to reject state files, plan files, secrets,
  credentials, and local overrides.
- Bootstrap implementation includes a documented migration path from local
  state to remote state when implemented.
- CI/CD can identify target environment, state boundary, and blast radius for
  deployable infrastructure changes.

## Related ADRs

- [ADR 0001 - Infrastructure as Code Engine](0001-iac-engine.md)
- [ADR 0002 - Repository Separation](0002-repository-separation.md)
- [ADR 0003 - Terraform Toolchain Baseline](0003-terraform-toolchain-baseline.md)
- [ADR 0004 - Remote State Strategy](0004-remote-state-strategy.md)
- [ADR 0005 - Management Group Hierarchy](0005-management-group-hierarchy.md)
- [ADR 0006 - Deployment Identity Strategy](0006-deployment-identity-strategy.md)
- [ADR 0007 - Enterprise Networking Strategy](0007-enterprise-networking-strategy.md)

## Revisit Conditions

Revisit this decision if:

- Deployment repositories require materially different structure to support
  production operations.
- State boundaries become too large, too small, or too awkward for safe plans,
  applies, rollback, or recovery.
- CI/CD implementation shows that the standard layout makes path targeting,
  environment approvals, or plan/apply separation unreliable.
- Future landing-zone or subscription vending design requires additional
  top-level structure.
- Enterprise environment, regulatory, administrative, or network boundaries
  require separate deployment repositories or a different environment model.
- Production incidents or recovery exercises show that repository structure
  obscures ownership, state boundaries, or operational procedures.

