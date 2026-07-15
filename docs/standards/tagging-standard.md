# Tagging Standard

## Purpose

This standard defines the enterprise Azure tagging requirements for the Azure
Platform Framework.

Tags provide structured metadata for ownership, cost management, operations,
governance, security, automation, and lifecycle management. They help platform
engineers, workload teams, finance, security, and operations understand who owns
a resource, why it exists, how it is supported, how it should be governed, and
how it should be reported.

Tags are not resource identity. Azure resource names identify resources and must
remain stable. Tags carry richer metadata that may change over time without
forcing resource replacement.

## Scope

This standard applies to:

- Platform resources deployed by foundation and connectivity repositories.
- Workload landing zones and subscription vending configuration.
- Reusable Terraform modules in the `azure-platform-modules` repository.
- Root deployments in `azure-platform-foundation` and
  `azure-platform-connectivity`.
- Examples and documentation that demonstrate tag usage.
- Azure Policy definitions, assignments, and initiatives that enforce or inherit
  tags.

This standard does not create Terraform implementation, Azure Policy
implementation, or CI/CD workflows.

## Tagging Principles

Required principles:

- Tags are metadata, not resource identity.
- Mutable information belongs in tags rather than names.
- Reusable modules must remain environment-neutral.
- Required tag values are supplied by root deployments, landing-zone
  configuration, or platform configuration.
- Modules may merge caller-provided tags but must not invent tenant-specific
  values.
- Tag keys must be consistent and centrally governed.
- Sensitive information must never be stored in tags.
- Tags must support automation without becoming the only control boundary.
- Tags must complement management group placement, subscription metadata,
  Azure Policy, RBAC, naming, and state boundaries.
- Tag values must use controlled vocabularies where governance, reporting, or
  automation depends on them.

Tags should help answer operational questions such as:

- Who owns this resource?
- Which workload or platform capability does it support?
- Which environment does it belong to?
- Which cost center or business unit funds it?
- How critical is it?
- What data classification applies?
- Who supports it?
- Is it active, temporary, sandbox, retired, or quarantined?
- Is it managed by Terraform, policy, or another approved platform process?

## Required Tag Categories

The following categories define the platform tagging model. This document does
not invent organization-specific tag values. Controlled values must be defined
by future governance, policy, subscription, and landing-zone design work.

### Environment

Purpose:

- Identifies the lifecycle or deployment classification of the resource, such
  as production, non-production, sandbox, shared platform, or similar controlled
  categories.
- Supports policy scoping, cost reporting, operational review, and blast-radius
  assessment.

Requirement:

- Mandatory for taggable resources unless the resource type does not support
  tags or a documented exemption applies.

Supplied by:

- Platform root deployments for platform resources.
- Landing-zone configuration for workload landing zones.
- Workload teams only through approved landing-zone or deployment
  configuration.

Policy integration:

- Azure Policy may enforce required presence.
- Azure Policy may inherit the value from subscription or resource group tags
  where the hierarchy and ownership model support inheritance.
- Deny enforcement should follow an audit and remediation period.

### Owner

Purpose:

- Identifies the accountable business or technical owner for the resource or
  workload.
- Supports review, escalation, lifecycle decisions, and exception ownership.

Requirement:

- Mandatory at subscription and resource group scopes.
- Conditional at individual resource scope when inherited ownership is reliable
  and documented.

Supplied by:

- Landing-zone configuration for workload resources.
- Platform root deployments for platform-owned resources.
- Workload teams through approved configuration, not ad hoc module edits.

Policy integration:

- Azure Policy may audit required presence.
- Inheritance from subscription or resource group scope may be used where
  resource-level enforcement would create avoidable duplication.

### Business Unit

Purpose:

- Identifies the business area, department, product group, or organizational
  unit responsible for the workload or platform capability.
- Supports reporting, governance accountability, and cost allocation.

Requirement:

- Mandatory for workload landing zones and workload subscriptions.
- Conditional for central platform resources where platform ownership is
  represented through another governed tag or subscription metadata field.

Supplied by:

- Landing-zone configuration for workload resources.
- Platform configuration for platform resources where required.

Policy integration:

- Azure Policy may audit presence and allowed values after the controlled
  business-unit vocabulary is defined.
- Inheritance may be appropriate from subscription or resource group scope.

### Cost Center

Purpose:

- Identifies the financial allocation target for chargeback, showback, budget
  reporting, and cost governance.

Requirement:

- Mandatory for workload landing zones, workload subscriptions, and shared
  services where cost allocation is required.
- Conditional for platform resources where central platform funding is handled
  outside resource-level tagging.

Supplied by:

- Landing-zone configuration.
- Platform root deployments for platform resources when required.
- Enterprise billing or approval workflow integration when subscription vending
  is implemented.

Policy integration:

- Azure Policy may audit required presence and allowed formats.
- Enforcement should consider resources created by Azure services that cannot
  accept tags or cannot inherit tags consistently.

### Workload or Application

Purpose:

- Identifies the workload, application, platform capability, or landing-zone
  service the resource supports.
- Supports inventory, incident response, cost reporting, and ownership review.

Requirement:

- Mandatory for workload landing zones and application resources.
- Mandatory for platform resources using a platform capability value.

Supplied by:

- Landing-zone configuration for workload resources.
- Platform root deployments for platform resources.

Policy integration:

- Azure Policy may enforce presence and may inherit from resource group or
  subscription scope.
- Allowed values should be governed by the future landing-zone request schema
  or platform configuration model.

### Criticality

Purpose:

- Identifies the business or operational criticality of the resource or
  workload.
- Supports incident prioritization, resilience expectations, monitoring
  severity, backup requirements, and change risk review.

Requirement:

- Mandatory for production workload landing zones.
- Conditional for non-production, sandbox, and platform resources until the
  operational criticality model is finalized.

Supplied by:

- Landing-zone configuration.
- Platform root deployments for platform services where criticality is
  meaningful.

Policy integration:

- Azure Policy may audit required presence and allowed values after the
  criticality taxonomy is defined.
- Enforcement should align with monitoring, backup, and production-readiness
  standards.

### Data Classification

Purpose:

- Identifies the sensitivity or classification of data processed or stored by
  the workload or resource.
- Supports security review, policy assignment, compliance reporting, and
  handling requirements.

Requirement:

- Mandatory for workload landing zones and data-bearing resources.
- Conditional for platform infrastructure resources that do not process or
  store workload data, unless security policy requires classification anyway.

Supplied by:

- Landing-zone configuration based on approved workload intake.
- Workload teams through governed configuration.
- Platform root deployments for platform data stores and observability
  resources.

Policy integration:

- Azure Policy may audit presence and allowed values.
- Enforcement must be coordinated with security and compliance requirements.
- Tag values must not contain regulated data, customer data, or sensitive
  details.

### Managed By

Purpose:

- Identifies the approved management mechanism for the resource, such as
  Terraform, Azure Policy, platform automation, or another approved system.
- Supports drift review, operational ownership, and incident triage.

Requirement:

- Mandatory for platform-managed resources.
- Conditional for workload resources where management ownership is delegated
  and documented.

Supplied by:

- Platform root deployments.
- Landing-zone configuration or workload deployment configuration when workload
  teams manage resources under platform guardrails.

Policy integration:

- Azure Policy may audit presence.
- Values must align with the repository and deployment ownership model.

### Lifecycle

Purpose:

- Identifies the lifecycle state of the resource or landing zone, such as
  active, temporary, sandbox, decommissioning, decommissioned, quarantine, or
  similar controlled states.
- Supports cleanup, retention, exception review, and subscription lifecycle
  management.

Requirement:

- Mandatory for sandbox, temporary, decommissioned, or quarantined scopes.
- Conditional for standard active production and non-production resources if
  lifecycle is already represented by subscription metadata or management group
  placement.

Supplied by:

- Platform root deployments for platform resources.
- Landing-zone configuration for workload resources.
- Subscription vending or retirement workflow when implemented.

Policy integration:

- Azure Policy may audit presence for scopes where lifecycle tracking is
  required.
- Policy remediation and cleanup automation may use this tag only when values
  are controlled and ownership is clear.

### Support Contact

Purpose:

- Identifies the operational support group, queue, or team responsible for
  responding to incidents, alerts, and service requests.

Requirement:

- Mandatory for production workload landing zones and platform resources with
  operational alerts.
- Conditional for non-production and sandbox resources based on support model.

Supplied by:

- Landing-zone configuration for workload resources.
- Platform root deployments for platform resources.
- Operations or service management integration where available.

Policy integration:

- Azure Policy may audit presence.
- Values should identify support groups or queues, not individual people.

## Platform Tags Versus Workload Tags

Tag ownership must align with repository responsibilities and operational
ownership.

### Platform Root Deployments

Platform root deployments own tags for platform resources they deploy.

They should supply:

- Environment.
- Platform capability or workload/application equivalent.
- Managed by.
- Platform owner or support group.
- Platform cost allocation where required.
- Data classification for platform data stores and observability resources.
- Lifecycle for temporary, sandbox, decommissioned, or quarantined platform
  resources.

Foundation and connectivity repositories may also compute effective platform
tags from shared configuration once the deployment architecture is defined.

### Landing-Zone Configuration

Landing-zone configuration owns workload-level metadata used to stamp resource
groups, spoke resources, diagnostics, budgets, and other landing-zone
resources.

It should supply:

- Environment.
- Owner.
- Business unit.
- Cost center.
- Workload or application.
- Criticality.
- Data classification.
- Lifecycle.
- Support contact.

Subscription vending should eventually validate these values before deployment.

### Workload Teams

Workload teams own workload-specific metadata that cannot be known by the
central platform team.

They may supply:

- Workload or application identifiers.
- Business owner and technical owner values through approved fields.
- Support contact values through approved fields.
- Data classification and criticality through governed intake.
- Additional workload-specific optional tags where allowed by the platform.

Workload teams must not bypass central tag keys, invent duplicate semantic
keys, or store sensitive information in tags.

### Individual Reusable Modules

Individual reusable modules do not own enterprise tag values.

They may:

- Accept `tags` as an input.
- Pass tags to taggable Azure resources.
- Add documented technical tags only when approved by this standard or a module
  design note.
- Return effective tags only when that output is useful to consumers.

They must not:

- Hard-code tenant-specific, organization-specific, workload-specific, owner,
  cost, or support values.
- Silently overwrite caller-provided tags.
- Invent environment-specific tags.
- Require consumers to edit module source to change tags.

## Module Behavior

Reusable Terraform modules must expose predictable tag behavior.

Required expectations:

- Accept `tags` as `map(string)` where the Azure resource supports tags.
- Use `{}` as the default only when tags are optional for the module and the
  caller can provide them.
- Do not hard-code organization-specific tags.
- Do not silently overwrite caller-provided tags.
- Clearly document any module-generated technical tags.
- Prefer tag merging in root or composition modules rather than every primitive
  module.
- Return effective tags only when they are part of a useful module contract.
- Document Azure resources in the module that do not support tags.
- Preserve caller tag values unless an explicitly documented precedence rule
  applies.
- Validate tag input where practical after controlled key and value rules are
  defined.

Technical tags may be appropriate when they describe module-managed behavior
and are not tenant-specific. Examples include a documented managed-by marker or
module identifier if approved by the platform. Technical tags must not replace
required enterprise metadata.

## Tag Merging Strategy

The platform may use several tag merging approaches. The chosen strategy must
keep modules reusable and environment-neutral.

### Caller Supplies All Tags

In this approach, each module caller passes the complete set of tags to each
module.

Advantages:

- Simple primitive module behavior.
- Maximum transparency.
- Avoids hidden module-level defaults.
- Works before platform-wide tag composition exists.

Disadvantages:

- Repeats tag merging logic across root deployments.
- Increases risk of inconsistent required tags.
- Makes it harder to enforce common platform defaults.
- Creates more review noise in deployment configuration.

### Module Merges Required and Caller Tags

In this approach, reusable modules merge platform-required tags with caller
tags.

Advantages:

- Can enforce module-specific technical tags.
- Reduces caller effort for some repeated tags.
- Keeps effective tags close to the resources being created.

Disadvantages:

- Risks embedding environment-specific values in reusable modules.
- Can produce inconsistent precedence rules across modules.
- Makes primitive modules more complex.
- Can silently overwrite caller values if not designed carefully.
- Spreads enterprise tagging policy across the module catalog.

### Root Deployment Computes Effective Tags

In this approach, root deployments or high-level composition modules compute
the effective tag map and pass it to primitive modules.

Advantages:

- Keeps reusable primitive modules environment-neutral.
- Centralizes tag merging near environment and landing-zone configuration.
- Makes required tags easier to validate before module calls.
- Reduces inconsistent precedence rules across primitive modules.
- Aligns with repository separation and configuration-driven deployment.

Disadvantages:

- Requires root deployment or composition-module discipline.
- Primitive modules cannot independently ensure required enterprise tags are
  present.
- Early implementations may need repeated helper patterns until shared
  composition emerges.

## Recommendation

Root deployments and high-level composition modules should compute effective
tags and pass them to reusable primitive modules.

Primitive modules should accept `tags` as `map(string)` and apply them to
taggable resources without inventing enterprise values. Composition modules may
merge platform defaults, landing-zone metadata, and caller-provided optional
tags when the merge rule is explicit and documented.

Recommended precedence, from lowest to highest:

- Platform default tags.
- Landing-zone or platform capability tags.
- Resource-specific caller tags.
- Explicit break-glass or exception tags where approved and documented.

Precedence must be documented before implementation. Required enterprise tags
should not be removed or overwritten by optional caller tags unless a documented
exception process allows it.

## Azure Policy Integration

Azure Policy is expected to enforce and inherit tags as part of the governance
strategy, but enforcement must be introduced carefully.

Required-tag enforcement:

- Required tags may be audited, denied, modified, or remediated by policy.
- Policy should start in audit mode before deny or modify enforcement.
- Deny policies must account for deployment pipelines, Azure-created child
  resources, and resource types that do not support tags.

Tag inheritance:

- Tags may be inherited from subscription or resource group scope when the
  inheritance model is documented.
- Inheritance must not hide missing landing-zone metadata in configuration.
- Inherited values must be predictable and aligned with ownership boundaries.

Audit before deny:

- New required tag policies should generally follow design, audit,
  remediation, enforcement, and periodic review.
- Existing resources should be remediated before deny controls block normal
  platform work.

Remediation considerations:

- Modify and deployIfNotExists policies may require managed identities and
  role assignments.
- Remediation tasks must have clear owners.
- Remediation must not overwrite intentional workload-provided values without
  a documented rule.

Policy exemptions:

- Exemptions must be documented, justified, time-limited where practical, and
  periodically reviewed.
- Exemptions should identify the resource type, scope, reason, owner, and
  expiration or review date.

Limitations:

- Not every Azure resource supports tags.
- Some child resources do not inherit tags automatically.
- Some Azure services create resources outside normal deployment paths.
- Tag behavior may differ across Azure resource providers.
- Azure Policy cannot replace correct module interfaces and root deployment
  configuration.

## Validation

Tag validation should be implemented in root deployments, composition modules,
primitive modules, and CI where practical after the controlled tag catalog is
defined.

Validation expectations:

- Tag keys must be non-empty.
- Tag values must be non-empty for required tags unless a documented exception
  permits an empty value.
- Required tag keys must use centrally governed spelling and casing.
- Reserved tag keys must be documented before enforcement.
- Duplicate semantic keys with different casing must be prohibited.
- Tag key casing must be consistent across all repositories.
- Tag value casing must be consistent for controlled values.
- Tag keys and values must respect Azure maximum supported lengths.
- Tag maps must avoid unsupported characters where policy, reporting, or
  automation requires restrictions.
- Optional tags must not conflict with required enterprise tag keys.
- Values used for automation must come from controlled vocabularies.

This standard does not choose final tag key casing, exact key names, allowed
values, or reserved key lists. Those decisions are deferred to governance,
policy, subscription, and landing-zone design work.

## Security and Privacy

Tags are widely visible to users, APIs, logs, policy evaluations, cost tools,
inventory exports, and third-party integrations. Tags must be treated as
non-secret metadata.

Tags must never contain:

- Credentials.
- Secrets.
- Tokens.
- Connection strings.
- Private keys.
- Personal access tokens.
- Passwords.
- Personal information about individual employees or customers.
- Incident details.
- Vulnerability details.
- Regulated data.
- Customer data.
- Legal, investigative, or security-sensitive context.

Use approved secret management, ticketing, incident, compliance, and data
governance systems for sensitive details. Tags may reference approved
non-sensitive identifiers only when the reference format is governed.

## Best Practices

- Define tag keys once and reuse them consistently.
- Use controlled values for tags that drive policy, automation, reporting, or
  routing.
- Keep tag values short, stable, and meaningful.
- Prefer group, queue, or service identifiers over individual names.
- Apply tags at subscription and resource group scopes when inheritance is part
  of the governance model.
- Pass effective tags into modules rather than repeating merge logic in every
  primitive module.
- Document module resources that do not support tags.
- Review tag changes for cost, policy, automation, and alert-routing impact.
- Use Azure Policy audit mode before deny or modify enforcement.
- Keep examples generic and free of real owner, cost, tenant, or support data.

## Anti-Patterns

The following patterns are prohibited:

- Hard-coded organization-specific tag values in reusable modules.
- Hard-coded business owner, cost center, environment, workload, or support
  values in reusable modules.
- Storing credentials, secrets, personal information, incident details, or
  regulated data in tags.
- Using tags as the only source of resource identity.
- Depending on tags as the only access-control boundary.
- Creating duplicate semantic tags with different casing or spelling.
- Allowing every team to invent tag keys independently.
- Silently overwriting caller-provided tags in modules.
- Spreading required enterprise tag merge logic across every primitive module.
- Using free-form values for tags that drive automation or policy.
- Treating Azure Policy remediation as a substitute for correct deployment
  configuration.
- Failing deployments for missing tags before an audit and remediation period
  has been completed.

## Review Checklist

Use this checklist during architecture, module, and deployment reviews:

- [ ] The tagging behavior follows this standard.
- [ ] Required tag categories are supplied by root deployment, landing-zone
      configuration, or approved platform configuration.
- [ ] Reusable modules accept `tags` as `map(string)` where resources support
      tags.
- [ ] Reusable modules do not hard-code organization-specific tag values.
- [ ] Reusable modules do not invent tenant-specific, workload-specific, cost,
      owner, support, or environment tag values.
- [ ] Tag merging happens in root deployments or high-level composition modules
      unless a module-specific reason is documented.
- [ ] Tag precedence is explicit and documented.
- [ ] Caller-provided tags are not silently overwritten.
- [ ] Module-generated technical tags are documented and justified.
- [ ] Resources that do not support tags are documented.
- [ ] Tag keys use consistent spelling and casing.
- [ ] Required tag values are non-empty or have documented exceptions.
- [ ] Tags do not contain secrets, personal information, incident details, or
      regulated data.
- [ ] Controlled tag values are validated where practical.
- [ ] Azure Policy enforcement or inheritance implications have been reviewed.
- [ ] Policy exemptions are documented where required.
- [ ] Example tags use placeholders and cannot be mistaken for production
      values.

## Definition of Done

Tagging is compliant when:

- Root deployments or approved high-level composition modules compute effective
  tags from platform configuration, landing-zone metadata, and resource-specific
  optional tags.
- Reusable modules accept `tags` as `map(string)` for taggable resources and
  apply them without inventing environment-specific values.
- Required and conditional tag categories are supplied at the correct scope.
- Tag keys and controlled values are consistent across repositories.
- Sensitive information is excluded from all tag values.
- Azure resources that do not support tags are documented and handled through
  policy exceptions or alternate metadata where required.
- Azure Policy audits, inherits, modifies, or denies tags only according to the
  approved governance rollout lifecycle.
- Policy exemptions are explicit, owned, and reviewed.
- Tag behavior can be reviewed from configuration and module documentation
  without undocumented knowledge.
- Tagging supports ownership, cost management, operations, governance,
  security, automation, and lifecycle management without conflicting with the
  naming, repository, or Terraform module standards.

## Deferred Decisions

The following decisions are deferred to governance, policy, subscription, and
landing-zone design work:

- Final tag key names and casing.
- Final controlled values for environment, business unit, criticality, data
  classification, lifecycle, managed-by, and support contact.
- Required versus inherited tag enforcement by management group, subscription,
  resource group, and resource scope.
- Azure Policy initiative structure for tag audit, inheritance, modify, deny,
  remediation, and exemption.
- Reserved tag key catalog.
- Exact policy rollout schedule from audit to enforcement.
- Subscription vending schema fields that supply required tag values.
- Handling model for Azure resource types that do not support tags.
