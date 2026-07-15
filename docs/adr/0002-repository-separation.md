# ADR 0002 - Repository Separation

## Status

Accepted

## Context

The Azure Landing Zone roadmap defines a platform that will be delivered through
architecture documentation, reusable Terraform modules, platform foundation
deployments, and enterprise connectivity deployments. The roadmap explicitly
prioritizes configuration-driven deployments, reusable modules, clear separation
between reusable modules and environment deployments, Azure Verified Module
adoption where appropriate, automated validation and deployment pipelines, state
isolation by lifecycle and blast radius, and recoverable infrastructure changes.

ADR 0001 establishes Terraform as the authoritative and officially supported
Infrastructure as Code engine for this platform. Terraform is required for
validation, planning, applying, state management, CI/CD workflows, and release
acceptance.

Repository separation must support those goals. It must make ownership,
release, state, access control, and pipeline boundaries clear before
implementation repositories are created.

This ADR does not create repositories, Terraform code, or pipelines. It defines
the repository model and contracts that later implementation work must follow.

## Decision Drivers

- Architecture decisions and implementation code must have different lifecycle
  and review patterns.
- Reusable modules must be independently versioned and consumed immutably by
  deployment repositories.
- Environment deployments must provide configuration without modifying module
  source.
- Foundation and connectivity deployments have different blast radius,
  permissions, state boundaries, and release cadence.
- CI/CD pipelines must be isolated by repository responsibility and deployment
  scope.
- CODEOWNERS and access controls must reflect real platform ownership.
- Shared standards must be referenced from one source of truth without copying
  them into every implementation repository.
- The architecture repository must not become a hidden deployment repository.

## Options Considered

1. Monorepo containing architecture docs, reusable modules, foundation
   deployments, and connectivity deployments.
2. Two-repository model:
   - architecture/docs
   - all Terraform implementation
3. Three-repository model:
   - architecture/docs
   - reusable modules
   - combined platform deployments
4. Four-repository model:
   - `azure-platform-architecture`
   - `azure-platform-modules`
   - `azure-platform-foundation`
   - `azure-platform-connectivity`

## Option 1 - Monorepo

### Description

Use one repository for architecture documentation, ADRs, standards, reusable
Terraform modules, foundation root deployments, connectivity root deployments,
examples, tests, and pipeline definitions.

### Advantages

- Simple repository discovery.
- Easy local cross-reference between architecture, modules, and deployments.
- One pull request can update docs, modules, and deployments together.
- Fewer cross-repository dependency management mechanics.
- Lower initial setup effort.

### Disadvantages

- Blurs architecture, module, and deployment ownership.
- Makes CODEOWNERS rules more complex and easier to bypass accidentally.
- Increases risk that documentation-only changes trigger implementation
  pipelines or implementation changes avoid proper deployment review.
- Couples module releases to environment deployment changes.
- Makes state and blast-radius boundaries less obvious to contributors.
- Encourages environment-specific configuration to drift into reusable module
  code.
- Requires careful pipeline filtering to avoid accidental plans or applies.

### State and Blast-Radius Implications

All state-consuming root deployments live beside modules and documentation.
This does not directly combine Terraform state files, but it makes operational
boundaries less visible. A single repository permission or branch protection
mistake can affect foundation and connectivity workflows at the same time.

### Release and Versioning Implications

Module versioning becomes harder to enforce because modules and consuming root
deployments are changed in the same repository. Teams may rely on local relative
paths or unreleased module code instead of immutable module versions.

### Access-Control and CODEOWNERS Implications

The repository must support highly granular ownership across docs, modules,
foundation, and connectivity. This is possible, but brittle. Contributors with
write access to the monorepo may have visibility or influence across areas that
should be operationally separated.

### Pipeline Isolation

Pipeline isolation depends on path filters and careful workflow design. That
adds risk because a single repository contains workflows for docs, module
validation, foundation plans, foundation applies, connectivity plans, and
connectivity applies.

### Cross-Repository Dependency Management

Cross-repository dependency management is minimal, but the reduction comes at
the cost of weaker release boundaries. Internal relative paths become tempting
and can undermine versioned module consumption.

## Option 2 - Two-Repository Model

### Description

Use one repository for architecture documentation, ADRs, standards, and roadmap,
and one repository for all Terraform implementation, including reusable modules,
foundation deployments, and connectivity deployments.

### Advantages

- Keeps architecture documentation separate from deployable code.
- Reduces risk that documentation changes trigger deployment pipelines.
- Easier to manage than a four-repository model.
- Allows Terraform implementation work to share local tooling and examples.
- Provides a clear documentation source of truth.

### Disadvantages

- Reusable modules remain colocated with environment deployments.
- Foundation and connectivity deployments share one implementation repository.
- Module release boundaries can be weakened by direct local consumption.
- CODEOWNERS still needs complex separation inside the implementation
  repository.
- Pipeline isolation still depends heavily on path filters.
- A single implementation repository becomes a broad operational control plane.

### State and Blast-Radius Implications

Architecture state risk is removed because docs are separated from Terraform
code. However, foundation and connectivity deployments still share repository
permissions, branch protection, and pipeline configuration. This increases the
chance that one repository-level issue affects multiple platform blast radii.

### Release and Versioning Implications

The implementation repository must enforce strict module release discipline to
prevent root deployments from consuming unreleased module source. This is
workable but fights the repository layout.

### Access-Control and CODEOWNERS Implications

Architecture reviewers can be separated from implementation reviewers. Within
the implementation repository, module, foundation, and connectivity ownership
must be separated by CODEOWNERS paths.

### Pipeline Isolation

Pipelines can be separated by path and workflow, but all implementation
pipelines live under one repository trust boundary. Foundation and connectivity
apply authorization must be carefully isolated.

### Cross-Repository Dependency Management

Only one implementation repository must consume architecture standards. Module
and deployment dependency management remains internal, which is simpler but less
explicit.

## Option 3 - Three-Repository Model

### Description

Use one repository for architecture documentation, one repository for reusable
Terraform modules, and one repository for combined platform deployments,
including foundation and connectivity.

### Advantages

- Separates architecture from implementation.
- Separates reusable modules from environment deployments.
- Supports independent module versioning and immutable module consumption.
- Reduces risk that environment-specific values enter reusable module code.
- Creates a clearer release boundary between module producers and deployment
  consumers.
- Easier to operate than a four-repository model.

### Disadvantages

- Foundation and connectivity still share one deployment repository.
- Deployment identity, state, and pipeline boundaries for foundation and
  connectivity must be separated inside one repository.
- CODEOWNERS for foundation and connectivity can become complex.
- Connectivity changes may be coupled to foundation release processes.
- One deployment repository can become too broad as platform capabilities grow.

### State and Blast-Radius Implications

Module state is not a concern because reusable modules should not own
environment state. Separating modules from deployments improves blast-radius
control.

The remaining weakness is that foundation and connectivity root deployments
share a repository. Their Terraform state can still be isolated, but repository
permissions, pipeline configuration, and branch protection are shared unless
carefully separated.

### Release and Versioning Implications

Reusable modules can be released independently and consumed by immutable
versions. Combined platform deployments can pin module versions.

Foundation and connectivity deployments may still have different release
cadence but must coordinate within one repository.

### Access-Control and CODEOWNERS Implications

Module CODEOWNERS can be separated cleanly from deployment CODEOWNERS.
Foundation and connectivity ownership still requires path-based control inside
the combined deployment repository.

### Pipeline Isolation

Module pipelines are cleanly isolated from deployment pipelines. Foundation and
connectivity pipelines still need isolation within the combined deployment
repository.

### Cross-Repository Dependency Management

Deployment code consumes released module versions. Both implementation
repositories reference shared architecture standards. Cross-repository
dependencies are manageable and visible.

## Option 4 - Four-Repository Model

### Description

Use four repositories:

| Repository | Responsibility |
| --- | --- |
| `azure-platform-architecture` | Architecture, standards, ADRs, diagrams, roadmap, and implementation guidance |
| `azure-platform-modules` | Reusable and independently versioned Terraform modules |
| `azure-platform-foundation` | Management groups, platform subscriptions, governance, identity, management, and observability deployments |
| `azure-platform-connectivity` | Enterprise hubs, routing, firewalls, DNS, hybrid connectivity, and spoke onboarding deployments |

### Advantages

- Matches the target repository model documented in the roadmap.
- Cleanly separates architecture, reusable modules, foundation deployments, and
  connectivity deployments.
- Keeps reusable modules independent from environment-specific configuration.
- Supports independent module release and semantic versioning.
- Supports separate state, identity, pipeline, and CODEOWNERS boundaries for
  foundation and connectivity.
- Reduces accidental coupling between governance/resource-organization work and
  network connectivity work.
- Makes repository purpose obvious to contributors and reviewers.
- Provides the clearest model for platform ownership and future scaling.

### Disadvantages

- Higher initial repository setup and governance effort.
- More cross-repository references and dependency updates.
- Risk of repository sprawl if additional repositories are created without a
  clear contract.
- Requires consistent standards for pipeline design, branching, CODEOWNERS,
  release notes, and version pinning.
- Coordinated platform changes may require pull requests across multiple
  repositories.

### State and Blast-Radius Implications

The four-repository model best supports state isolation by lifecycle and blast
radius. Reusable modules have no environment state. Foundation deployments own
state for management groups, platform subscriptions, governance, identity,
management, and observability concerns. Connectivity deployments own state for
network hubs, routing, firewalls, DNS, hybrid connectivity, and spoke onboarding
concerns.

This separation reduces the chance that a connectivity change affects
management group or policy state, or that a foundation change affects network
hub state. It also lets deployment identities and state access be scoped to the
minimum required repository responsibility.

### Release and Versioning Implications

The module repository releases immutable module versions. Deployment
repositories pin module versions and upgrade deliberately. Architecture
standards and ADRs define the rules but do not produce deployable artifacts.

Foundation and connectivity can have separate release cadence, approval gates,
rollback plans, and production windows. This is important because governance and
networking often have different operational risk profiles.

### Access-Control and CODEOWNERS Implications

Each repository can have CODEOWNERS aligned to its responsibility:

- Architecture owners approve standards, ADRs, roadmap, and diagrams.
- Module owners approve reusable module interfaces, implementation, tests, and
  releases.
- Foundation owners approve management group, policy, identity, management, and
  observability deployments.
- Connectivity owners approve hub, routing, firewall, DNS, hybrid, and spoke
  deployments.

Repository permissions can be scoped more cleanly than path-only controls inside
a monorepo. Apply permissions and protected environments can be tied to the
deployment repository that owns the relevant state.

### Pipeline Isolation

Pipelines are isolated by repository responsibility:

- Architecture pipelines validate Markdown, links, diagrams, and documentation
  conventions.
- Module pipelines run Terraform formatting, validation, linting, security
  scanning, tests, and release packaging.
- Foundation pipelines run Terraform validation, plans, applies, drift checks,
  and release gates for foundation state boundaries.
- Connectivity pipelines run Terraform validation, plans, applies, drift
  checks, and release gates for connectivity state boundaries.

This reduces accidental deployment from documentation changes and avoids
cross-triggering unrelated platform applies.

### Cross-Repository Dependency Management

Cross-repository dependencies must be explicit:

- Deployment repositories consume released module versions, not local module
  source.
- Module version updates are made through pull requests in deployment
  repositories.
- Architecture standards are referenced by link, release tag, or documented
  version, not copied into implementation repositories.
- ADR changes that alter implementation contracts must be reflected in module
  standards and deployment repository work items.
- Outputs from one deployment repository must not become undocumented
  integration contracts; cross-state dependency rules must be documented before
  use.

## Repository Contracts

### `azure-platform-architecture`

Responsibility:

- Platform roadmap.
- Architecture documents.
- ADRs.
- Standards.
- Diagrams.
- Implementation guidance.
- Decision backlog and milestone definitions.

Must not contain:

- Terraform root modules.
- Reusable Terraform module source.
- Terraform state files or state backups.
- Terraform plan files.
- Provider lock files for deployable root modules.
- Azure tenant IDs, subscription IDs, client secrets, private keys, or
  environment-specific deployment configuration.
- CI/CD workflows that apply Azure infrastructure.
- Generated deployment artifacts.

Contract:

- Defines what the platform will do and why.
- Publishes standards that implementation repositories must follow.
- Records decisions before implementation patterns are repeated.
- References implementation repositories without becoming one.

### `azure-platform-modules`

Responsibility:

- Reusable Terraform modules.
- Module examples.
- Module tests.
- Module documentation.
- Module release notes and semantic versioning.
- Azure Verified Module wrappers or compositions where approved.

Must not contain:

- Environment-specific tenant IDs, subscription IDs, regions, address ranges, or
  resource names.
- Root deployments for real platform environments.
- Remote state backend configuration for real platform deployments.
- Terraform state files or plan files.
- Long-lived credentials, secrets, private keys, or pipeline environment
  secrets.
- Hard-coded enterprise IP allocations.
- Foundation or connectivity environment configuration.

Contract:

- Produces independently versioned modules.
- Exposes typed inputs and intentional outputs.
- Avoids managing unrelated resources inside a module.
- Can be consumed by multiple deployment repositories without modifying module
  source.
- Treats Terraform as the authoritative supported engine, per ADR 0001.

### `azure-platform-foundation`

Responsibility:

- Terraform root deployments for management groups.
- Platform subscriptions and subscription placement.
- Governance and Azure Policy assignments.
- Identity and RBAC assignments.
- Management and observability resources.
- Foundation state boundaries and deployment pipelines.

Contract:

- Consumes released module versions.
- Provides environment-specific configuration.
- Owns foundation remote state and deployment identities.
- Does not modify module source to deploy an environment.
- Does not own enterprise hub networking, firewall, routing, or hybrid
  connectivity state except where a documented dependency requires coordination.

### `azure-platform-connectivity`

Responsibility:

- Terraform root deployments for enterprise network hubs.
- Routing, firewalls, firewall policy, DNS, private resolver, hybrid
  connectivity, and spoke onboarding.
- Connectivity state boundaries and deployment pipelines.
- Network-specific configuration such as hub regions, address allocations,
  routing intent, and DNS forwarding rules.

Contract:

- Consumes released module versions.
- Provides connectivity-specific environment configuration.
- Owns connectivity remote state and deployment identities.
- Coordinates with foundation through documented outputs, data contracts, or
  configuration inputs.
- Does not own management group hierarchy, global policy baseline, or platform
  identity state unless explicitly documented.

## Why Reusable Modules Must Remain Separate from Environment Deployments

Reusable modules are product artifacts. Environment deployments are product
consumers. Combining them encourages root deployments to consume unreleased local
module code, weakens semantic versioning, and makes it easier for
environment-specific values to enter module source.

Keeping modules separate ensures:

- Modules can be tested and released independently.
- Deployment repositories consume immutable module versions.
- Module interfaces are reviewed as stable contracts.
- Environment configuration remains outside module implementation.
- Rollbacks can target either module version changes or deployment
  configuration changes.
- Module changes do not automatically imply production deployment changes.

## Why Foundation and Connectivity Need Separate Deployment Repositories

Foundation and connectivity are both platform concerns, but they have different
failure modes, owners, permissions, release cadence, and state blast radius.

Foundation deployments affect management groups, policy inheritance,
subscription placement, identity, RBAC, and management resources. Connectivity
deployments affect hubs, routing, firewall policy, DNS, hybrid connectivity, and
spoke onboarding. A mistake in either area can be broad, but the recovery steps,
approvers, test criteria, and operational windows are different.

Separate deployment repositories allow:

- Separate CODEOWNERS.
- Separate deployment identities.
- Separate state backends, containers, and keys.
- Separate branch protection and environment approvals.
- Separate release calendars.
- Separate drift detection and recovery procedures.
- Reduced chance that a network change unintentionally couples to a governance
  change.

## Shared Standards Reference Model

Shared standards must be authored once in `azure-platform-architecture` and
referenced by implementation repositories. They must not be copied and forked.

Implementation repositories should reference standards by:

- Stable Markdown links to this repository.
- Release tags or commit SHAs when standards need a fixed baseline.
- Pull request checklists that point to the relevant standard.
- Pipeline documentation that links to the authoritative standard.
- ADR references in module or deployment design notes.

If an implementation repository needs a local template, the template must link
back to the authoritative standard and avoid changing the standard's meaning.

## Risks of Repository Sprawl

The four-repository model is intentionally bounded. Additional repositories
must not be created for every platform feature by default.

Repository sprawl risks include:

- Harder discovery for engineers.
- More duplicated pipeline and CODEOWNERS logic.
- More dependency update pull requests.
- More places for standards to drift.
- Unclear ownership if repository contracts are weak.
- Increased operational overhead for access reviews and branch protection.

New repositories require an ADR or explicit standards update when the proposed
responsibility cannot fit within the existing contracts.

## Decision

Adopt the four-repository model:

- `azure-platform-architecture`
- `azure-platform-modules`
- `azure-platform-foundation`
- `azure-platform-connectivity`

This model matches the roadmap and best supports the platform's goals:
configuration-driven deployments, independently versioned reusable modules,
clear separation between modules and environment deployments, state isolation by
lifecycle and blast radius, controlled CI/CD, and recoverable infrastructure
changes.

No documented constraint requires a smaller repository model. The additional
coordination cost is acceptable because it buys clearer ownership, safer
pipeline boundaries, stronger release contracts, and better operational
separation between foundation and connectivity.

## Consequences

- Implementation repositories must be created separately when their roadmap
  milestones begin.
- Architecture decisions and standards remain in
  `azure-platform-architecture`.
- Reusable modules are released from `azure-platform-modules` and consumed by
  version from deployment repositories.
- Foundation and connectivity deployments have separate repository permissions,
  CODEOWNERS, pipelines, state boundaries, and deployment identities.
- Cross-repository dependency management must be explicit and reviewed.
- Changes that span architecture, modules, foundation, and connectivity may
  require coordinated pull requests.
- Shared standards must be referenced from the architecture repository rather
  than duplicated.

## Risks

- Repository sprawl may increase if new repositories are created without clear
  contracts.
- Cross-repository changes may take longer to coordinate.
- Module version updates may lag in deployment repositories.
- Standards may be ignored if implementation repositories do not link to them
  clearly.
- Inconsistent branch protection, CODEOWNERS, or pipeline patterns could weaken
  the separation model.
- Foundation and connectivity teams may create undocumented state or output
  dependencies.
- Contributors may find the model harder to navigate without clear repository
  READMEs and onboarding guidance.

## Validation Criteria

The decision is valid when:

- Each repository has a documented responsibility contract before it is used.
- `azure-platform-architecture` contains no Terraform root deployments,
  reusable module source, Terraform state, plan files, secrets, or Azure
  environment configuration.
- `azure-platform-modules` contains no real environment root deployments,
  tenant-specific values, subscription-specific values, state files, plan files,
  or deployment secrets.
- Deployment repositories consume released module versions.
- Foundation and connectivity have separate Terraform state boundaries.
- Foundation and connectivity have separate CODEOWNERS and apply approval paths.
- CI/CD workflows are isolated by repository responsibility.
- Shared standards are referenced from `azure-platform-architecture`.
- Cross-repository dependencies are documented and reviewed.

## Revisit Conditions

- Repository count creates more operational burden than it removes.
- A platform ownership model emerges that combines foundation and connectivity
  under one team with one release cadence and one state strategy.
- Enterprise tooling cannot support the required cross-repository dependency,
  CODEOWNERS, or pipeline patterns.
- Module release and consumption workflows become too slow for platform
  delivery.
- Security or compliance requirements demand stronger separation than the
  four-repository model provides.
- A future platform capability does not fit any existing repository contract and
  cannot be handled through standards or configuration.
- Production incidents show that repository separation caused confusion,
  delayed recovery, or undocumented dependencies.
