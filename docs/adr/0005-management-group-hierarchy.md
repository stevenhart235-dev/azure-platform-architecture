# ADR 0005 - Management Group Hierarchy

## Status

Accepted

## Context

The Azure Platform Framework must provide a management group hierarchy that can
support enterprise Azure Landing Zone governance while still being practical for
the Ordicor Platform Lab.

The roadmap requires a standardized management group hierarchy, policy-driven
governance, dedicated platform subscriptions, repeatable landing-zone
deployment, future subscription vending, and clear separation between platform
and workload concerns. ADR 0001 establishes Terraform as the authoritative
Infrastructure as Code engine. ADR 0002 establishes the four-repository model,
including separate foundation and connectivity deployment repositories. ADR
0003 establishes the Terraform toolchain baseline, and ADR 0004 establishes the
initial remote state strategy.

This ADR records the accepted management group hierarchy and ownership model.
It does not create Terraform code, management groups, subscriptions, policies,
workflows, or repositories.

## Decision Drivers

- Optimize for enterprise architecture first.
- Scale down for the Ordicor Platform Lab without changing the architecture.
- Keep policy inheritance predictable.
- Minimize policy assignment duplication.
- Separate platform-owned subscriptions from application-owned landing-zone
  subscriptions.
- Treat identity as a first-class platform pillar.
- Give future subscription vending an obvious placement model.
- Avoid unnecessary application management-group layers before there is a real
  governance or ownership need.
- Support production, non-production, sandbox, and decommissioned lifecycle
  patterns.

## Options Considered

1. Flat subscription-only hierarchy.
2. Minimal lab-only hierarchy.
3. Enterprise platform and landing-zone hierarchy.
4. Deep application hierarchy beneath landing zones.

## Option 1 - Flat Subscription-Only Hierarchy

Use few or no purpose-specific management groups and manage controls directly
at subscription scopes.

Advantages:

- Simple for a very small lab.
- Requires fewer management groups at the beginning.
- Easy to understand when only one subscription exists.

Disadvantages:

- Does not scale well to enterprise governance.
- Duplicates policy and RBAC assignments across subscriptions.
- Makes future subscription vending ambiguous.
- Blurs platform, workload, sandbox, and decommissioned lifecycle boundaries.
- Forces restructuring later as the platform grows.

## Option 2 - Minimal Lab-Only Hierarchy

Create only the management groups required by the current lab subscription.

Advantages:

- Matches the current lab footprint.
- Reduces initial implementation overhead.
- Avoids unused-looking hierarchy branches.

Disadvantages:

- Optimizes for current size instead of target architecture.
- Requires disruptive restructuring when enterprise subscriptions are added.
- Makes future policy inheritance and subscription vending less predictable.
- Can encourage one-off placement decisions that do not match the platform
  model.

## Option 3 - Enterprise Platform and Landing-Zone Hierarchy

Define the target enterprise hierarchy first, then allow the lab implementation
to contain only the subscriptions and branches it currently needs.

Advantages:

- Provides a durable enterprise target.
- Keeps the lab aligned with future production architecture.
- Supports platform and workload ownership separation.
- Supports policy inheritance with less duplication.
- Gives subscription vending obvious placement destinations.
- Allows additional subscriptions to expand under existing branches without
  redesign.

Disadvantages:

- Some hierarchy branches may be empty or lightly used during the lab phase.
- Requires contributors to understand that small implementation size does not
  change the target architecture.
- May feel more formal than a single-subscription lab needs.

## Option 4 - Deep Application Hierarchy Beneath Landing Zones

Create additional management-group layers for application portfolios,
applications, teams, or business units under Production, Non-Production, and
Sandbox by default.

Advantages:

- Can support delegated governance for large organizations.
- May help organizations with strong portfolio or business-unit policy
  differences.
- Can create narrower policy scopes when justified.

Disadvantages:

- Adds complexity before requirements justify it.
- Increases policy inheritance depth and exception handling.
- Makes subscription vending placement less obvious unless the business
  taxonomy is mature.
- Risks encoding organizational structure that may change more frequently than
  the platform architecture.
- Creates unnecessary management-group churn for the initial platform.

## Decision

Adopt option 3: define the enterprise platform and landing-zone hierarchy first
and allow the Ordicor Platform Lab to scale down without changing the
architecture.

The accepted top-level hierarchy is:

```text
Tenant
|-- Platform
|-- Landing Zones
`-- Decommissioned
```

The `Platform` management group contains:

```text
Platform
|-- Management
|-- Identity
|-- Connectivity
`-- Shared Services
```

The `Landing Zones` management group contains:

```text
Landing Zones
|-- Production
|-- Non-Production
`-- Sandbox
```

Subscriptions are placed directly beneath the appropriate platform pillar or
landing-zone lifecycle group.

Do not introduce additional application management-group layers by default.
Additional layers require a documented governance, ownership, policy, or
subscription vending need.

Identity is a first-class platform pillar. It is not simply another shared
service.

## Rationale

The hierarchy should express durable platform architecture, not the temporary
size of the lab. A single-subscription lab can still follow an enterprise
target model by using only the branches it needs at first. This avoids
redesigning management groups when new platform or workload subscriptions are
added.

Separating `Platform`, `Landing Zones`, and `Decommissioned` creates clear
policy and lifecycle boundaries. Platform subscriptions receive platform-owned
governance and operational controls. Landing-zone subscriptions receive
workload-focused policies based on production, non-production, or sandbox
lifecycle. Decommissioned subscriptions can retain restrictive controls without
polluting active platform or workload scopes.

Within `Platform`, separate `Management`, `Identity`, `Connectivity`, and
`Shared Services` pillars reflect different ownership, permissions, blast
radius, deployment cadence, and operational risk. Identity is treated as a
first-class pillar because identity services, privileged access, directory
integration, and identity-related controls are foundational platform concerns,
not generic shared services.

Within `Landing Zones`, direct placement under `Production`,
`Non-Production`, or `Sandbox` keeps subscription vending simple and
predictable. It avoids introducing application management-group layers before
there are clear policy inheritance, ownership, or delegation requirements.

## Ownership Model

Platform teams own the platform management groups:

- `Management`
- `Identity`
- `Connectivity`
- `Shared Services`

Platform ownership includes the architecture, policy expectations, deployment
patterns, operational controls, and state boundaries for platform-owned
subscriptions placed under those groups.

Application teams own landing-zone subscriptions. Their ownership applies to
the workloads and resources deployed inside their subscriptions, subject to
inherited platform governance, security controls, landing-zone configuration,
and operational requirements.

Platform teams own the landing-zone management group structure and inherited
controls. Application teams do not own the management groups themselves by
default.

## Policy Model

Policy inheritance follows the management group hierarchy:

```text
Tenant
  -> Platform / Landing Zones / Decommissioned
    -> Child management groups
      -> Subscriptions
```

Tenant-level policy should be used carefully for controls that truly apply
across the entire tenant. Broad controls assigned too high can create avoidable
exceptions.

Top-level management groups define broad policy intent:

- `Platform` receives controls for platform-owned infrastructure and operating
  model requirements.
- `Landing Zones` receives controls that apply to workload subscriptions.
- `Decommissioned` receives restrictive lifecycle and protection controls for
  retired or quarantined subscriptions.

Child management groups refine policy by platform pillar or landing-zone
lifecycle:

- `Management`, `Identity`, `Connectivity`, and `Shared Services` can inherit
  shared platform controls and receive pillar-specific assignments.
- `Production`, `Non-Production`, and `Sandbox` can inherit workload controls
  and receive lifecycle-specific assignments.

This minimizes duplication because common controls are assigned once at the
highest appropriate scope and inherited by child scopes. Exceptions and
specialized controls are applied lower in the hierarchy only when needed.

## Subscription Placement

Platform subscriptions are placed under the relevant `Platform` child
management group:

- Management and observability subscriptions go under `Management`.
- Identity-related platform subscriptions go under `Identity`.
- Network hub, routing, firewall, DNS, and hybrid connectivity subscriptions go
  under `Connectivity`.
- Shared platform services that are not identity, management, or connectivity
  concerns go under `Shared Services`.

Landing-zone subscriptions are placed directly under:

- `Production` for production workload subscriptions.
- `Non-Production` for development, test, staging, or other non-production
  workload subscriptions.
- `Sandbox` for sandbox, experimentation, training, or temporary workload
  subscriptions.

Retired, quarantined, or decommissioning subscriptions are moved to
`Decommissioned` when their lifecycle requires restrictive controls and removal
from active landing-zone or platform policy inheritance.

Do not create application, team, product, or portfolio management groups under
landing zones by default. Use tags, subscription metadata, naming, and future
subscription vending configuration for ownership and reporting unless policy
inheritance or delegated administration requires additional management groups.

## Future Subscription Vending

The hierarchy must support future subscription vending.

New subscriptions should have an obvious placement destination:

- Platform capability subscriptions map to the appropriate platform pillar.
- Workload production subscriptions map to `Landing Zones/Production`.
- Workload non-production subscriptions map to `Landing Zones/Non-Production`.
- Sandbox subscriptions map to `Landing Zones/Sandbox`.
- Retired or quarantined subscriptions map to `Decommissioned`.

Subscription vending implementation is deferred. Future vending design must
define request schema, approval workflow, subscription metadata, tags,
management group placement, policy assignments, RBAC, networking, DNS, budgets,
and diagnostics. This ADR defines the placement model only.

## Enterprise Scaling

The Ordicor Platform Lab may initially contain only a single subscription. The
architecture does not change simply because the implementation is smaller.

Enterprise growth happens by adding subscriptions beneath the existing
hierarchy:

- Additional production workload subscriptions are added under `Production`.
- Additional non-production workload subscriptions are added under
  `Non-Production`.
- Additional sandbox subscriptions are added under `Sandbox`.
- Additional platform subscriptions are added under `Management`, `Identity`,
  `Connectivity`, or `Shared Services`.
- Decommissioned or quarantined subscriptions are moved under
  `Decommissioned`.

This lets the platform scale without restructuring the hierarchy. New
management-group layers should be added only when a documented policy,
delegation, regulatory, ownership, or subscription vending requirement cannot
be met through the accepted hierarchy, tags, subscription metadata, or
configuration.

## Consequences

- The platform has a durable enterprise management group target.
- The lab can start small without creating a different lab-specific
  architecture.
- Platform and workload ownership boundaries are visible in the hierarchy.
- Identity is treated as a dedicated platform pillar.
- Subscription placement is simple enough for future vending.
- Common policies can be assigned higher in the hierarchy and inherited by
  child scopes.
- Additional application management-group layers are not created by default.
- Future deployment repositories must align root deployments, state boundaries,
  and permissions with this hierarchy.

## Risks

- Empty or lightly used management groups may appear unnecessary during the lab
  phase.
- Assigning policies too high in the hierarchy could create avoidable
  exceptions.
- Avoiding application layers by default may require later hierarchy expansion
  for large organizations with delegated business-unit governance.
- Subscription metadata, tags, and vending configuration must be strong enough
  to support ownership and reporting without default application management
  groups.
- Moving existing subscriptions later may require careful policy, RBAC, and
  compliance review.
- If platform ownership is unclear, `Shared Services` can become a catch-all
  for resources that should belong under a more specific pillar.

## Validation Criteria

This decision is valid when:

- The documented target hierarchy contains `Platform`, `Landing Zones`, and
  `Decommissioned` beneath the tenant.
- `Platform` contains `Management`, `Identity`, `Connectivity`, and
  `Shared Services`.
- `Landing Zones` contains `Production`, `Non-Production`, and `Sandbox`.
- Identity is documented as a first-class platform pillar.
- Subscriptions are placed directly beneath platform pillars or landing-zone
  lifecycle groups.
- No default application, team, product, or portfolio management-group layers
  are introduced.
- Platform teams own platform management groups.
- Application teams own landing-zone subscriptions, not the landing-zone
  management groups by default.
- Policy inheritance is documented from tenant to top-level groups to child
  groups to subscriptions.
- Future subscription vending has clear placement destinations.
- The lab implementation can contain a single subscription without changing
  the target architecture.

## Revisit Conditions

Revisit this decision if:

- Enterprise governance requires delegated business-unit or application
  management groups.
- Policy exceptions become common because the hierarchy is too broad or too
  shallow.
- Subscription vending requires additional placement dimensions that cannot be
  represented through metadata, tags, or configuration.
- Regulatory boundaries require separate management-group branches.
- Platform ownership changes materially.
- Identity, connectivity, management, or shared services ownership no longer
  maps cleanly to the accepted platform pillars.
- Sandbox, production, or non-production lifecycle controls require a different
  hierarchy.
- Decommissioned subscription handling requires additional quarantine,
  retention, or archive scopes.
- Production incidents or governance reviews show that policy inheritance is
  unclear or unsafe.
