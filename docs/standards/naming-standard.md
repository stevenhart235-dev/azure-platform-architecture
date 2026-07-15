# Naming Standard

## Purpose

This standard defines the enterprise Azure naming requirements for the Azure
Platform Framework.

Naming is part of the platform contract. Resource names must help engineers,
operators, security reviewers, and automation understand what a resource is,
where it belongs, and how it should be managed. Names must be consistent enough
to validate, predictable enough to review, and stable enough that normal
business or ownership changes do not cause avoidable resource replacement.

This standard is authoritative for every future Terraform module in the
`azure-platform-modules` repository and for naming decisions made by the
foundation and connectivity deployment repositories.

This document does not define a final enterprise naming convention or token
order. That decision is intentionally deferred until the required platform
metadata, Azure constraints, and initial foundational module patterns are better
understood.

## Scope

This standard applies to:

- Azure resource names.
- Resource group names.
- Management group names.
- Subscription display names and subscription aliases where applicable.
- Terraform module inputs that accept or influence names.
- Deployment identity names.
- Remote state keys and related platform artifact names when the state standard
  references this document.
- Examples and documentation that show naming patterns.

This standard does not create Terraform code, pipelines, or a naming module.

## Naming Philosophy

Names must balance human readability, automation requirements, and Azure
resource-provider constraints.

Required principles:

- Consistency: similar resources should be named using the same metadata model
  and vocabulary.
- Predictability: engineers should be able to infer the naming approach without
  reverse-engineering custom logic from each module.
- Human readability: names should help people identify purpose, environment,
  resource type, and ownership context where those values are appropriate.
- Automation friendliness: names should be derived from structured metadata
  where practical, validated before deployment, and easy for CI/CD and policy
  checks to evaluate.
- Azure compatibility: names must respect Azure resource-specific uniqueness,
  length, case, and character rules.
- Stability: names must avoid metadata that changes frequently, because many
  Azure resources cannot be renamed without replacement or migration.
- Least surprise: module consumers must be able to see and review the final
  name or the structured inputs used to produce it.

Names are not a complete inventory system. They should carry enough stable
context to make resources understandable, while tags and configuration carry
full ownership, cost, compliance, and lifecycle metadata.

## Standard Naming Components

The platform naming model must consider the following metadata components. A
future naming convention may include some, all, or none of these components for
a specific resource type depending on Azure constraints and operational value.

### Organization

An organization token can identify the enterprise or platform boundary that
owns the resource.

Tradeoffs:

- Useful when resources are globally visible or globally unique.
- Can help distinguish enterprise resources from vendor, sandbox, or acquired
  environment resources.
- May be unnecessary inside a single tenant or subscription boundary.
- Can consume scarce characters for resources with short name limits.
- Should be stable; avoid legal entity names or brand names that may change.

### Platform or Workload

A platform or workload token can identify the capability, service, application,
or landing zone associated with the resource.

Tradeoffs:

- Useful for human readability and workload association.
- Helps when many resource groups or subscriptions exist in the same scope.
- Can become unstable if application names, product names, or team ownership
  changes frequently.
- Should use a durable capability identifier rather than a temporary project
  name where possible.

### Environment

An environment token can identify lifecycle or deployment classification, such
as production, non-production, development, test, sandbox, or shared platform.

Tradeoffs:

- Useful for quick operational recognition and blast-radius review.
- Helps avoid confusing production and non-production resources.
- Must align with the subscription and management group model.
- Should not replace required environment tags or state boundaries.
- Should use controlled values once the environment taxonomy is finalized.

### Region

A region token can identify Azure regional placement.

Tradeoffs:

- Useful for regional services, networking, observability, and disaster
  recovery planning.
- Helps distinguish multiple regional instances of the same capability.
- May be inappropriate for global resources, tenant-level resources, or
  resources whose scope is not regional.
- Region abbreviations must be standardized before use.
- Renaming a resource to reflect a move between regions is not a migration
  strategy; regional replacement must be planned explicitly.

### Resource Type

A resource type token can identify the Azure resource kind or platform artifact
type.

Tradeoffs:

- Improves scanability in the Azure Portal, CLI output, logs, and cost exports.
- Helps distinguish related resources with the same workload and environment.
- Can be redundant where Azure already displays the resource type clearly.
- Must account for short or restrictive Azure naming rules.
- Abbreviations must be centrally defined to avoid drift.

### Instance Number

An instance token can distinguish multiple resources with the same metadata.

Tradeoffs:

- Useful for active-active pairs, regional scale units, repeated spokes, or
  multiple instances of the same resource type.
- Stable numbering avoids accidental replacement when resources are added.
- Sequential numbering can become misleading if resources are removed.
- Instance numbers should not encode capacity or ordinal meaning unless the
  lifecycle is documented.

### Capability or Function

A capability token can identify the functional role of a resource, such as hub,
spoke, firewall, resolver, logging, policy, or diagnostics.

Tradeoffs:

- Useful when a workload contains multiple resources of the same Azure type.
- More stable than team names or project names when based on platform
  capability.
- Requires a controlled vocabulary to prevent synonyms and abbreviations from
  drifting.

### Lifecycle State

Lifecycle state, such as active, sandbox, quarantine, or decommissioned, may be
tempting to include in a name.

Tradeoffs:

- Usually better represented by tags, management group placement, subscription
  metadata, and state boundaries.
- Often changes during the life of a resource.
- Can create expensive rename or replacement pressure.
- Should appear in names only when the lifecycle state is a stable scope, such
  as a dedicated sandbox subscription naming pattern.

## Names Versus Tags

Names and tags serve different purposes.

Names should contain stable, compact, operationally useful identifiers that help
people and automation recognize the resource. Names are best for metadata that
rarely changes and that must be visible even when tags are not available.

Tags should contain richer metadata used for governance, cost management,
ownership, automation, compliance, and operations. Tags are better for values
that may change over time or that require full fidelity.

Information that may belong in names:

- Resource type or platform artifact type.
- Durable platform capability or workload identifier.
- Environment classification where it is stable and important for review.
- Region where the resource is regional.
- Instance number or scale-unit identifier.
- Organization or platform identifier where global uniqueness or shared tenancy
  makes it valuable.

Information that should usually belong in tags:

- Business owner.
- Technical owner.
- Cost center.
- Operational support group.
- Data classification.
- Business criticality.
- Lifecycle state.
- Compliance or regulatory scope.
- Managed-by indicator.
- Request, ticket, or approval reference.
- Expiration date or review date.

Information that must not be encoded in names:

- Secrets, credentials, or sensitive values.
- Personal names or individual employee identifiers.
- Full cost center detail where a tag is more appropriate.
- Values expected to change frequently.
- Data that would expose sensitive business, security, or regulatory context.

Names and tags must work together. A resource name should provide quick context;
tags should provide authoritative metadata.

## Azure Naming Considerations

Azure naming rules are not uniform. Each resource provider and resource type may
have different uniqueness, length, character, case, and rename constraints.

Required considerations:

- Global uniqueness: some resource names must be globally unique across Azure,
  such as many public endpoints or storage-related names.
- Scope uniqueness: some names only need to be unique within a resource group,
  subscription, management group, DNS zone, or parent resource.
- Length restrictions: Azure resource name limits vary widely and may force
  shorter tokens, abbreviations, or generated suffixes.
- Character restrictions: some resources allow hyphens, underscores, or mixed
  case; others require only lowercase letters and numbers.
- Case behavior: some Azure APIs preserve case, some compare case
  insensitively, and some require lowercase.
- Leading and trailing characters: some resources restrict starting or ending
  characters.
- Consecutive separators: some resources reject repeated separators or names
  ending in separators.
- Rename support: many Azure resources cannot be renamed in place. A name
  change may require replacement, migration, DNS updates, identity updates, or
  downstream reconfiguration.
- DNS exposure: names used in endpoints may become public or private DNS names
  and must avoid sensitive or misleading information.
- Provider validation: Terraform provider validation may not catch every Azure
  API naming rule before apply.

Modules and deployment repositories must validate naming inputs where practical,
but Azure resource-specific rules must still be checked during design and
review.

## Separators, Abbreviations, and Suffixes

The final separator, abbreviation, and suffix rules are deferred until the
enterprise naming convention is selected.

Interim requirements:

- Use only resource-type-compatible separators.
- Do not assume hyphens, underscores, or mixed case are valid for every
  resource type.
- Use abbreviations only when they are documented or required by Azure length
  limits.
- Avoid local synonyms for the same concept.
- Use generated suffixes only for resources that require uniqueness beyond the
  selected metadata components.
- Generated suffixes must be deterministic where stable names are required.
- Random suffixes must be used only when uniqueness is required and the
  resource lifecycle can tolerate a non-human token.
- Suffix generation must be visible to the caller or documented by the module.

## Module Expectations

Reusable Terraform modules must treat naming as part of their public contract.

Modules must:

- Accept names as inputs where a caller needs direct control over Azure resource
  names.
- Accept structured naming metadata only when the module clearly documents how
  that metadata is used.
- Avoid inventing environment-specific names.
- Avoid hard-coded organization, workload, environment, region, subscription,
  owner, or cost values.
- Avoid hidden naming logic in locals that callers cannot understand or
  override.
- Use composition instead of ad hoc string concatenation where practical.
- Validate names or naming metadata where practical.
- Return final names as outputs when consumers or operators need them.
- Document naming behavior in the module README.
- Keep Terraform resource labels stable even when Azure resource names are
  caller supplied.

Modules must not:

- Build production names from undocumented defaults.
- Derive Azure resource names from repository names, branch names, local user
  names, or workspace names unless an approved deployment standard explicitly
  permits it.
- Depend on Terraform workspace names for enterprise environment naming.
- Generate different names on repeated plans with unchanged inputs.
- Hide random naming behavior inside a module.
- Force one naming convention onto all resource types before the enterprise
  naming convention is accepted.

Root deployment repositories are responsible for supplying environment-specific
names or naming metadata. The module repository is responsible for exposing
clear, validated interfaces that allow deployment repositories to follow this
standard.

## Naming Approaches

The platform may use one or more naming approaches depending on resource type,
module maturity, and deployment repository needs.

### Fully Caller-Supplied Names

In this approach, the caller passes the final Azure resource name to the module.

Advantages:

- Maximum caller control.
- Simple module interface for resources with complex or unusual Azure naming
  rules.
- Easy to handle imported resources or brownfield environments.
- Avoids hiding naming decisions inside modules.
- Works before a full enterprise naming module exists.

Disadvantages:

- Higher risk of inconsistent names across repositories and teams.
- More repeated naming logic in root deployments.
- Validation may be uneven unless root deployments and CI enforce it.
- Callers must understand Azure naming restrictions for every resource type.
- Refactoring names later can be expensive because many resources cannot be
  renamed in place.

### Convention-Generated Names

In this approach, a module or helper generates names from structured metadata
and a documented convention.

Advantages:

- Strong consistency across modules and deployment repositories.
- Easier validation and automation.
- Less repeated naming logic in root deployments.
- Better support for subscription vending and configuration-driven landing
  zones.
- Reduces review noise once the convention is mature.

Disadvantages:

- Requires a well-governed metadata model and abbreviation catalog.
- Can hide important decisions if implemented poorly.
- May not fit every Azure resource type.
- Can create breaking changes if the convention changes after resources are
  deployed.
- May make brownfield adoption or special cases harder.

### Hybrid Names

In this approach, modules support both caller-supplied names and
convention-generated names from structured metadata.

Advantages:

- Provides consistency where the convention is mature while preserving escape
  hatches for special cases.
- Supports gradual adoption across foundational modules, connectivity modules,
  and future landing-zone automation.
- Allows modules to expose explicit final-name overrides for resources with
  strict Azure constraints.
- Works well while the platform is still defining state, identity, subscription,
  and deployment patterns.

Disadvantages:

- Interface design is more complex.
- Requires clear precedence rules between supplied names and generated names.
- Validation must prevent ambiguous inputs.
- Poorly designed hybrid modules can become inconsistent across the module
  catalog.

## Recommendation

Use a hybrid approach as the platform direction.

During early foundational module development, modules should prefer explicit
caller-supplied names unless there is a clear, documented platform pattern for a
specific resource type. This keeps module behavior transparent while the final
enterprise naming convention is still being defined.

As the platform matures, modules may accept structured naming metadata or use a
dedicated naming module when doing so improves consistency and reduces repeated
root deployment logic. Hybrid modules must define clear precedence rules. A
caller-supplied final name should either be mutually exclusive with naming
metadata or explicitly documented as an override.

The platform must not implement hidden, module-local naming conventions that
conflict with this standard or force future deployment repositories to reverse
engineer naming behavior.

## Best Practices

- Prefer stable capability identifiers over temporary project names.
- Keep names short enough to allow resource-specific prefixes, suffixes, and
  future expansion.
- Use controlled vocabularies for environment, region, resource type, and
  capability tokens.
- Validate name length and character rules as close to the module interface as
  practical.
- Keep final generated names visible in plans and outputs where useful.
- Document any resource-specific naming limitations in module READMEs.
- Use tags for ownership, cost, support, lifecycle, compliance, and other rich
  metadata.
- Review rename impact before changing a naming input.
- Treat generated suffix algorithms as part of the module contract.
- Use examples with placeholder values only.

## Anti-Patterns

The following patterns are prohibited:

- Hard-coded production resource names in reusable modules.
- Hard-coded tenant IDs, subscription IDs, regions, workload names, address
  ranges, owners, or cost centers in reusable modules.
- Hidden name generation that callers cannot inspect, override, or validate.
- Random suffixes that change on repeated plans.
- Names derived from local machine names, user names, branch names, or
  Terraform workspace names without an approved deployment standard.
- Encoding secrets, sensitive business data, or regulatory details in names.
- Encoding frequently changing ownership or lifecycle data in names.
- Creating different naming conventions for each module without documenting the
  reason.
- Using abbreviations that are not documented or obvious to platform reviewers.
- Depending on names as the only source of ownership, cost, or compliance
  metadata.
- Renaming resources casually without migration and rollback planning.

## Review Checklist

Use this checklist when reviewing modules, deployment configuration, examples,
and architecture changes that affect names:

- [ ] The naming behavior follows this standard.
- [ ] The module accepts a name or documented naming metadata as an explicit
      input.
- [ ] The module does not invent environment-specific names.
- [ ] The module does not hard-code organization, workload, environment,
      region, subscription, owner, cost, or support values.
- [ ] The final name or generated-name behavior is visible to the caller.
- [ ] Naming inputs are typed, documented, and validated where practical.
- [ ] Azure resource-specific length, character, case, and uniqueness rules
      have been considered.
- [ ] Globally unique resources have a documented uniqueness strategy.
- [ ] Generated suffixes are deterministic unless random uniqueness is
      explicitly justified.
- [ ] Caller-supplied names and generated names have clear precedence or are
      mutually exclusive.
- [ ] Names avoid frequently changing metadata.
- [ ] Tags carry ownership, cost, support, lifecycle, compliance, and other
      rich metadata.
- [ ] Example names use placeholders and cannot be mistaken for production
      values.
- [ ] Any rename impact has been reviewed for replacement, migration, DNS,
      identity, and downstream dependency risk.
- [ ] Module README documentation explains naming behavior and limitations.

## Definition of Done

A naming implementation is complete when:

- It follows the naming philosophy in this standard.
- It uses explicit caller-supplied names, documented structured metadata, or an
  approved hybrid pattern.
- It avoids hidden naming logic and environment-specific assumptions.
- It accounts for Azure resource-specific naming restrictions.
- It validates naming inputs where practical.
- It separates stable identifiers in names from richer metadata in tags.
- It documents generated suffix behavior and uniqueness strategy where used.
- It avoids resource replacement risk from unstable naming components.
- It can be reviewed without undocumented knowledge.
- It supports configuration-driven deployment from the foundation and
  connectivity repositories.

## Future Work

A dedicated naming module may eventually generate resource names from
structured metadata such as organization, platform or workload, environment,
region, resource type, capability, and instance number.

That decision is deferred until after foundational modules are implemented and
the platform has enough real module interfaces to validate the right metadata
model. The future naming module decision should consider:

- Final management group and subscription models.
- Final environment taxonomy.
- Region abbreviation standards.
- Resource type abbreviation standards.
- Global uniqueness requirements.
- State key and deployment identity naming requirements.
- Subscription vending requirements.
- Brownfield or imported resource needs.
- Whether a naming module reduces repeated root deployment logic without hiding
  important operational controls.

Until that decision is made, reusable modules must keep naming behavior
explicit, documented, and compatible with this standard.
