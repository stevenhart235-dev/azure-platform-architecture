# M0 - Architecture and Standards

## Purpose and Objectives

Milestone 0 defines the architecture, standards, decision backlog, and implementation
boundaries required before Azure platform resources, reusable modules, or deployment
pipelines are created.

The objective is to make later implementation work repeatable and reviewable by
documenting the platform rules that engineers must follow when building the Azure
Landing Zone platform.

This milestone must:

- Define the platform scope and non-goals.
- Confirm repository responsibilities and separation of concerns.
- Define Terraform compatibility expectations.
- Establish the strategy for adopting Azure Verified Modules.
- Define reusable module composition principles.
- Document the required management group hierarchy and subscription model decisions.
- Define naming, tagging, state, identity, networking, IP address management, DNS,
  governance, and policy requirements.
- Create the ADR backlog required to record durable architecture decisions.
- Provide clear exit criteria for starting implementation milestones.

## Scope

In scope:

- Architecture and standards for the Azure Landing Zone platform.
- Repository strategy for architecture, reusable modules, foundation deployments, and
  connectivity deployments.
- Decision criteria for major platform architecture choices.
- Required documentation updates under `docs/architecture`, `docs/standards`,
  `docs/adr`, and `docs/roadmap`.
- Work item definition for future implementation milestones.

Out of scope:

- Terraform or OpenTofu code.
- CI/CD pipelines.
- Creation of the future implementation repositories.
- Azure resource deployment.
- Migration of existing workloads.
- Final production runbooks, except where runbook requirements must be identified.

## Assumptions

- This repository, `azure-platform-architecture`, remains the source of truth for
  architecture, standards, roadmap, and ADRs.
- Future implementation repositories will be created separately for reusable modules,
  platform foundation, and connectivity.
- Platform implementation will use Terraform module patterns.
- Azure Landing Zone design principles will guide management group, subscription,
  governance, networking, and operational decisions.
- Azure Verified Modules will be preferred where they meet platform requirements without
  creating unacceptable abstraction, support, or compatibility risk.
- Production Azure resources will not be deployed until M0 exit criteria are satisfied.
- Tenant-specific values, subscription IDs, regions, IP ranges, and resource names will
  be provided as configuration in deployment repositories, not embedded in reusable
  modules.

## Required Architecture Decisions

The following decisions must be made or explicitly queued as ADRs before implementation
begins:

- IaC engine: decide whether the platform standard is Terraform, OpenTofu, or
  dual-compatible execution. ADR 0001 resolved this as Terraform only.
- Repository separation: confirm the responsibility boundary between architecture,
  reusable modules, foundation deployments, and connectivity deployments.
- Module composition: decide how thin platform modules, Azure Verified Modules, and
  direct AzureRM/AzAPI resources are composed.
- Azure Verified Modules: decide when AVM modules are mandatory, preferred, optional, or
  avoided.
- Management group hierarchy: define the target hierarchy, inheritance boundaries, and
  policy assignment scopes.
- Subscription model: define required platform subscriptions, workload subscription
  types, ownership metadata, and lifecycle states.
- State strategy: define backend ownership, state isolation boundaries, state naming,
  locking, access, recovery, and cross-state data sharing rules.
- Deployment identity strategy: define local, CI plan, CI apply, break-glass, and
  bootstrap identities.
- Enterprise network topology: decide between hub-and-spoke, Virtual WAN, or a hybrid
  model using documented criteria.
- DNS architecture: define private DNS ownership, resolver placement, forwarding, hybrid
  integration, and zone lifecycle.
- Governance strategy: define policy assignment scopes, exemption process, enforcement
  lifecycle, and ownership.
- Naming and tagging: define required standards and validation expectations.

## Repository Strategy

The target repository model remains the four-repository model described in the roadmap
overview.

| Repository | Responsibility |
| --- | --- |
| `azure-platform-architecture` | Architecture, standards, ADRs, diagrams, roadmap, and implementation guidance |
| `azure-platform-modules` | Reusable, independently versioned Terraform modules |
| `azure-platform-foundation` | Management groups, platform subscriptions, governance, identity, management, and observability resources |
| `azure-platform-connectivity` | Enterprise hubs, routing, firewalls, DNS, hybrid connectivity, and spoke onboarding |

Repository requirements:

- Architecture decisions must be recorded in this repository before implementation
  patterns are copied into other repositories.
- Reusable modules must not contain environment-specific tenant IDs, subscription IDs,
  address ranges, regions, or resource names.
- Deployment repositories must consume immutable module versions after the module
  repository exists.
- Foundation and connectivity repositories must own their own root deployments and
  remote state boundaries.
- Shared standards must be documented once in this repository and referenced by
  implementation repositories.

ADR 0002 resolved the repository model as separate architecture, modules,
foundation, and connectivity repositories. Foundation bootstrap begins in
`azure-platform-foundation`; connectivity remains separate for the later
connectivity milestone.

## Terraform Compatibility Strategy

The platform must define a compatibility standard before module development starts.
ADR 0001 resolved Terraform as the authoritative engine. OpenTofu compatibility is
not part of the supported validation or release matrix unless a future ADR changes
that decision.

Required decisions:

- Approved Terraform CLI version range.
- Minimum AzureRM provider version and provider upgrade policy.
- Required use of AzAPI provider for Azure capabilities not yet available in AzureRM.
- Whether lock files are committed per root module, per example, or only in deployment
  repositories.

Compatibility requirements:

- Modules must avoid language features that prevent execution under the approved engine
  or engines.
- Modules must declare provider requirements and version constraints.
- Root deployments must pin provider versions according to the approved versioning
  standard.
- Module examples must show configuration-driven usage without embedding tenant-specific
  values.
- CI validation in M1 must enforce the compatibility decision made in this milestone.

## Azure Verified Modules Adoption Strategy

Azure Verified Modules are the preferred starting point when they meet platform
requirements, but adoption must be deliberate rather than automatic.

Adoption criteria:

- The AVM module supports the required Azure resource type and production configuration
  surface.
- The module exposes required diagnostics, identity, network, tagging, lock, role
  assignment, and private endpoint settings.
- The module versioning and maintenance posture are acceptable for enterprise use.
- The module works with the approved Terraform strategy.
- The module does not force environment-specific values into reusable module code.
- The module can be wrapped without hiding important operational controls.

Required classifications:

- Mandatory: use AVM unless a documented exception exists.
- Preferred: use AVM when it meets functional and operational requirements.
- Optional: use AVM or direct provider resources based on complexity and
  maintainability.
- Avoid: do not use AVM for the case because it creates unacceptable constraints or
  risk.

Every non-trivial exception to the preferred AVM strategy must be documented in an ADR
or module design note.

## Module Composition Principles

Reusable modules must be product-quality building blocks, not copied root deployments.

Principles:

- Modules accept configuration through variables and return useful outputs.
- Modules do not embed tenant IDs, subscription IDs, regions, CIDR ranges, names, or
  environment labels.
- Modules have clear ownership boundaries and avoid managing unrelated resources.
- Modules should compose lower-level modules only when composition reduces repeated
  implementation complexity.
- Root modules coordinate environment-specific deployments and state boundaries.
- Platform modules may wrap AVM modules to enforce naming, tagging, diagnostics, locks,
  private networking, and policy expectations.
- Modules must not read remote state directly unless the platform standard explicitly
  permits a controlled pattern.
- Inputs must be typed, validated where practical, and documented.
- Outputs must be intentional and avoid leaking unnecessary implementation detail.
- Module versioning must follow the repository versioning standard.

## Management Group Hierarchy Design

The management group hierarchy must support policy inheritance, delegated ownership,
subscription lifecycle, and separation between platform and workload concerns.

Required design decisions:

- Tenant root group handling and whether policy assignments are allowed at tenant root.
- Top-level platform, landing zone, sandbox, and decommissioned group structure.
- Placement of identity, management, connectivity, and other platform subscriptions.
- Production and non-production workload grouping.
- Sandbox scope, expiration expectations, and policy differences.
- Decommissioned subscription handling and retained controls.
- Scope for policy initiatives, RBAC assignments, budgets, and diagnostic requirements.
- Naming convention for management groups.

Decision criteria:

- Minimize policy duplication.
- Keep inheritance predictable.
- Avoid assigning broad controls at scopes where exceptions will be common.
- Support future subscription vending.
- Support separate release cadence for platform foundation and connectivity.

## Subscription Model

The subscription model must define how platform and workload subscriptions are created,
placed, owned, tagged, governed, and retired.

Required subscription categories:

- Identity.
- Management.
- Connectivity.
- Security, if separate from management.
- Shared services, if required.
- Workload production.
- Workload non-production.
- Sandbox.
- Decommissioned or quarantine.

Required metadata:

- Business owner.
- Technical owner.
- Cost center.
- Environment classification.
- Data classification.
- Workload or platform capability name.
- Operational support group.
- Lifecycle state.

The unresolved decision is whether subscription creation is initially manual with
documented enrollment requirements or automated through a subscription vending process
in M9. The criteria are enterprise agreement constraints, identity permissions, billing
integration, approval workflow, and operational readiness.

## Naming Standard Requirements

The naming standard must define predictable, validated names for Azure resources and
platform artifacts.

Requirements:

- Define name components, allowed values, ordering, separators, and abbreviations.
- Account for Azure resource-specific length and character restrictions.
- Include rules for globally unique names.
- Include naming rules for management groups, subscriptions, resource groups, state
  keys, module releases, and deployment identities.
- Define environment, region, workload, instance, and capability tokens.
- Define when generated suffixes are allowed and how they are produced.
- Require validation in modules, root deployments, or CI where practical.
- Avoid names that require manual interpretation to identify owner, environment, or
  purpose.

## Tagging Standard Requirements

The tagging standard must support governance, cost management, operations, ownership,
and automation.

Required tag categories:

- Ownership.
- Cost allocation.
- Environment.
- Data classification.
- Business criticality.
- Operational support.
- Managed-by indicator.
- Lifecycle state.
- Compliance or regulatory scope, if applicable.

Requirements:

- Define required and optional tags.
- Define allowed values for controlled tags.
- Define inheritance and merge behavior between platform defaults and workload-specific
  tags.
- Define where tags are enforced by policy and where they are validated by code.
- Define exception handling for resource types that do not support tags.
- Define whether tags are applied at subscription, resource group, and resource scopes.

## Remote State and State Isolation Strategy

Remote state must be secured, recoverable, and isolated by ownership, lifecycle, and
blast radius.

Required decisions:

- Backend storage subscription and resource group ownership.
- Storage account naming and regional placement.
- Container and key strategy.
- State boundaries for management groups, policy, identity, management, connectivity,
  DNS, regional hubs, and workload landing zones.
- Access model for plan, apply, read-only, break-glass, and bootstrap identities.
- Use of storage account immutability, soft delete, versioning, private endpoints,
  customer-managed keys, and network restrictions.
- Cross-state dependency pattern.
- State recovery and restore process.

Requirements:

- Local state must not be used for shared platform deployments after bootstrap.
- State must be separated where independent rollback, ownership, or failure domains are
  required.
- Deployment identities must receive least-privilege access to only the state they
  require.
- State outputs must not become an undocumented integration contract.

## Deployment Identity Strategy

Deployment identities must separate read, plan, apply, bootstrap, and emergency access.

Required identity decisions:

- Whether CI/CD uses workload identity federation.
- Whether platform foundation and connectivity use separate service principals or
  managed identities.
- Scope of plan-only identities.
- Scope of apply identities.
- Bootstrap identity ownership and retirement expectations.
- Break-glass process and audit requirements.
- Required Azure AD groups for human approval and emergency access.

Requirements:

- Long-lived client secrets must be avoided unless an exception is documented.
- Apply identities must be scoped to the minimum required management group,
  subscription, and resource scopes.
- CI identities must not have broad owner permissions by default.
- Identity permissions must align with state access boundaries.
- Role assignments must be documented and later codified.

## Enterprise Networking Decision Areas

M0 must define decision criteria for enterprise networking before connectivity
implementation begins.

Required decision areas:

- Hub-and-spoke versus Virtual WAN versus hybrid topology.
- Number of initial regions and expansion pattern.
- Centralized internet egress requirements.
- Azure Firewall and firewall policy ownership.
- Routing model, including user-defined routes, route tables, BGP propagation, and
  inspection paths.
- Private endpoint and private link approach.
- Network security boundary between platform and workload teams.
- Hybrid connectivity readiness for VPN and ExpressRoute.
- Spoke onboarding pattern.
- Network observability and flow logging requirements.
- Multi-region resiliency expectations.

These decisions should be recorded in architecture documentation and promoted to ADRs
when they constrain implementation.

## IP Address Management Requirements

IP address management must prevent overlap and support subscription vending, spoke
onboarding, hybrid routing, and future regional expansion.

Requirements:

- Define source of truth for Azure address allocations.
- Define enterprise CIDR ranges available for Azure.
- Define reserved ranges for hubs, firewalls, gateways, private endpoints, DNS
  resolvers, shared services, and workloads.
- Define allocation size standards for hubs and spokes.
- Define overlap detection requirements.
- Define process for requesting, approving, reserving, and retiring address ranges.
- Define integration expectations with existing enterprise IPAM tooling, if any.
- Define how address allocations are represented in configuration.

Unresolved decision: whether IPAM is initially document-driven, configuration-driven in
Git, or integrated with an enterprise IPAM system. Criteria are current tooling,
approval process, automation needs, and risk of address conflicts.

## DNS Architecture Requirements

DNS architecture must support private endpoints, hybrid resolution, platform services,
and workload landing zones.

Requirements:

- Define ownership of Azure private DNS zones.
- Define which private DNS zones are centrally managed.
- Define zone link strategy for hubs, spokes, and platform subscriptions.
- Define Azure DNS Private Resolver placement and scaling approach, if required.
- Define inbound and outbound forwarding rules.
- Define hybrid DNS integration with on-premises resolvers.
- Define split-horizon DNS requirements.
- Define naming requirements for platform and workload zones.
- Define operational ownership for DNS records, resolver rules, and troubleshooting.

Unresolved decision: whether DNS is deployed with connectivity, foundation, or a
dedicated root deployment. Criteria are lifecycle, ownership, dependency direction, and
blast radius.

## Governance and Policy Strategy

Governance must be inherited, measurable, and exception-driven.

Requirements:

- Define policy initiative structure and assignment scopes.
- Define audit, deny, deployIfNotExists, modify, and disabled rollout process.
- Define required policy exemptions, expiration, approval, and review process.
- Define allowed locations and allowed resource types strategy.
- Define tag enforcement strategy.
- Define diagnostic settings enforcement strategy.
- Define public network access restriction strategy.
- Define Defender for Cloud and security baseline expectations.
- Define RBAC assignment standards.
- Define ownership for policy remediation tasks.
- Define reporting expectations for compliance drift.

Policy must be introduced in a controlled lifecycle: design, audit, remediation,
enforcement, and periodic review.

## Accepted ADR Baseline

M0 produced the accepted ADR baseline listed in
[../adr/README.md](../adr/README.md).

Accepted ADRs:

- `0001-iac-engine.md`: Terraform execution standard.
- `0002-repository-separation.md`: target repository model and ownership
  boundaries.
- `0003-terraform-toolchain-baseline.md`: Terraform, provider, root constraint,
  lock-file, and AzAPI baseline.
- `0004-remote-state-strategy.md`: backend, state boundaries, access, recovery,
  and cross-state dependency rules.
- `0005-management-group-hierarchy.md`: management group structure and
  inheritance model.
- `0006-deployment-identity-strategy.md`: bootstrap, plan, apply, and
  break-glass identity model.
- `0007-enterprise-networking-strategy.md`: hub-and-spoke networking reference
  architecture.
- `0008-root-deployment-repository-structure.md`: deployment repository layout
  for Foundation, connectivity, and future landing-zone repositories.

Deferred decision areas remain visible in ADR revisit conditions, standards,
and future roadmap milestones. Do not create placeholder ADR files merely to
reserve numbers.

## Deliverables

- Completed M0 roadmap specification in this document.
- Architecture documentation for target architecture, management groups, subscription
  model, identity and RBAC, governance, network topology, and DNS.
- Standards documentation for repository structure, Terraform modules, naming,
  tagging, state management, versioning, testing, and pipelines.
- ADRs for major architecture decisions.
- Updated diagrams where needed to explain repository boundaries, management groups,
  deployment flow, and enterprise networking.
- Prioritized implementation work items for M1 through M3 where M0 decisions directly
  affect them.

## Work Items

- [ ] Define platform scope, assumptions, and non-goals.
- [x] Complete repository responsibility model.
- [x] Decide Terraform compatibility strategy.
- [x] Define provider and versioning expectations.
- [ ] Define Azure Verified Modules adoption rules.
- [ ] Define module composition principles.
- [ ] Draft management group hierarchy.
- [ ] Draft subscription model.
- [ ] Define naming standard requirements.
- [ ] Define tagging standard requirements.
- [x] Define remote state isolation and access strategy.
- [x] Define deployment identity strategy.
- [x] Define enterprise networking decision criteria.
- [ ] Define IP address management requirements.
- [ ] Define DNS architecture requirements.
- [ ] Define governance and policy rollout strategy.
- [x] Complete accepted ADR baseline `0001` through `0008`.
- [x] Remove unused empty ADR placeholders.
- [ ] Review architecture and standards for consistency with the roadmap overview.
- [ ] Confirm M0 exit criteria before starting deployable implementation.

## Dependencies

- Agreement on enterprise Azure tenant and billing constraints.
- Access to current identity, security, network, DNS, and compliance requirements.
- Confirmation of existing enterprise IPAM and DNS tooling.
- Input from platform engineering, security, networking, identity, operations, and
  application representatives.
- Decision on Terraform support expectations.
- Agreement on repository ownership and lifecycle.

## Risks

- Delayed IaC engine decision can cause incompatible module and pipeline work.
- Weak repository boundaries can blur ownership and increase deployment blast radius.
- Overly broad management group policy assignment can create avoidable exceptions.
- Underspecified state boundaries can make recovery and concurrent delivery difficult.
- Using AVM modules without evaluation can hide unsupported operational requirements.
- Naming and tagging standards that are not validated will drift quickly.
- IPAM decisions made too late can block networking and subscription vending.
- DNS ownership ambiguity can cause private endpoint and hybrid resolution failures.
- Deployment identities with excessive permissions can create security and audit risk.

## Exit Criteria

M0 is complete when:

- Required architecture decisions are documented or explicitly listed as unresolved with
  decision criteria.
- ADRs `0001` through `0006` are completed or updated to reflect current direction.
- Required ADRs beyond `0006` are created or added to the backlog with owners.
- Repository responsibilities are clear enough to create future implementation
  repositories without redesign.
- Terraform compatibility expectations are clear enough for validation design.
- Module composition and AVM adoption strategies are clear enough for M3 module design.
- Management group and subscription model decisions are clear enough for M4
  implementation planning.
- Naming, tagging, state, and identity standards are clear enough to prevent ad hoc
  implementation.
- Networking, IPAM, and DNS decision areas are documented before M7 design work begins.
- Governance and policy rollout expectations are documented before M5 implementation
  planning.
- No Terraform code, pipelines, or additional repositories have been created as part of
  this milestone.

## Accomplished Looks Like

Engineers can start the engineering toolchain, bootstrap, state, and module milestones
without inventing repository structures, module conventions, naming rules, tag rules,
state boundaries, deployment identities, management group placement, subscription
categories, or network architecture while writing code.

The platform has a documented decision framework: unresolved choices are visible, tied
to criteria, and queued as ADRs instead of being hidden in implementation. Future
repositories can be created with clear ownership, and future Terraform work can
be reviewed against documented enterprise standards.
