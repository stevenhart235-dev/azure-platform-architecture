# Terraform Module Standard

## Purpose

This standard defines the engineering requirements for reusable Terraform
modules in the `azure-platform-modules` repository.

Module consistency is critical to long-term platform maintainability. The Azure
platform will be extended over time by different engineers, teams, and release
cycles. Modules that follow a common structure, naming model, input pattern,
output contract, documentation format, and testing baseline are easier to
review, version, upgrade, troubleshoot, and consume safely from deployment
repositories.

Reusable modules are platform product artifacts. They must provide stable,
documented contracts that can be consumed by foundation and connectivity root
deployments without copying implementation code or embedding environment-specific
configuration.

## Scope

This standard applies to every reusable Terraform module in the
`azure-platform-modules` repository.

It applies to:

- Platform-owned modules.
- Modules that wrap Azure Verified Modules.
- Modules that compose multiple lower-level modules.
- Module examples.
- Module tests.
- Module documentation.

This standard does not define root deployment repository structure, remote state
backend implementation, or CI/CD workflow implementation. Those concerns belong
to deployment and pipeline standards.

## Module Philosophy

Reusable modules must be small, focused, explicit, predictable, and safe to
consume.

Required principles:

- Small focused modules: a module must manage a clear platform capability or
  resource pattern. It must not become a broad deployment root.
- Composition over giant modules: compose smaller modules or Azure Verified
  Modules where composition reduces repeated implementation complexity.
- Explicit inputs: required decisions must be represented as input variables,
  not hidden in locals or hard-coded values.
- Explicit outputs: modules must publish intentional outputs that form useful
  downstream contracts.
- No hidden behavior: modules must not create optional or unrelated resources
  unless the behavior is controlled by explicit inputs and documented.
- Predictable plans: module changes should produce clear and explainable plan
  output. Avoid dynamic behavior that makes plans difficult to review.
- Idempotent deployments: repeated applies with unchanged inputs must converge
  without unexplained drift.
- Secure defaults: defaults must prefer secure, private, least-privilege, and
  observable configurations where the module can safely decide.
- Backward compatibility where practical: additive changes are preferred.
  Breaking interface or behavior changes require a major version increment.

Modules must be designed for reuse across environments. They must not contain
tenant-specific, subscription-specific, region-specific, address-specific, or
workload-specific assumptions.

## Required Directory Structure

Every module must use a predictable directory structure. The exact set of files
may vary by module complexity, but the standard layout is:

```text
<module-name>/
  main.tf
  variables.tf
  outputs.tf
  versions.tf
  locals.tf
  README.md
  examples/
    basic/
    complete/
  tests/
```

Required files and directories:

- `main.tf`: declares the primary resources, module calls, and data sources
  managed by the module.
- `variables.tf`: declares all input variables, including types, descriptions,
  defaults, and validation blocks where appropriate.
- `outputs.tf`: declares all intentional outputs, including descriptions and
  sensitive markings where required.
- `versions.tf`: declares required Terraform and provider version constraints.
  Child modules must declare requirements but must not configure providers.
- `locals.tf`: declares derived values used to keep implementation readable.
  Locals must not hide required caller decisions.
- `README.md`: documents the module purpose, usage, inputs, outputs, examples,
  compatibility, notes, and limitations.
- `examples/basic`: provides the smallest meaningful example of module usage.
- `examples/complete`: provides a fuller example that demonstrates optional
  features and production-relevant configuration patterns.
- `tests`: contains module tests or test fixtures appropriate to the module.

Optional files may be added when useful:

- `data.tf`: for data sources when separating them improves readability.
- `providers.tf`: only for required provider declarations if the repository
  chooses that convention. It must not contain provider configuration blocks.
- `diagnostics.tf`: for diagnostic settings if separating them improves
  readability.
- `role_assignments.tf`: for RBAC resources if separating them improves
  readability.

Optional files must not fragment small modules unnecessarily. Readability is the
goal, not file count.

## Provider Rules

Child modules must never configure providers.

Child modules must never configure Terraform backends.

Providers and backends are configured only in root modules.

Required rules:

- Do not add `provider` blocks to reusable child modules.
- Do not add `terraform.backend` configuration to reusable child modules.
- Do declare `required_providers` in `versions.tf`.
- Do declare the required Terraform version in `versions.tf`.
- Use provider aliases only when the caller explicitly passes aliased providers
  from a root module.

Rationale:

- Root deployments own authentication, subscription targeting, provider aliases,
  and backend state.
- Child modules must be reusable across foundation, connectivity, and future
  deployment repositories.
- Provider configuration inside child modules prevents callers from controlling
  identity, subscription, tenant, and alias behavior.
- Backend configuration inside child modules would violate state isolation and
  make modules unsafe to reuse.

## Variable Standards

Variables define the module contract. They must be explicit, typed, documented,
and validated where practical.

Required standards:

- Every variable must include a `description`.
- Every variable must declare a concrete `type`.
- Use `object` variables where a group of related settings forms one cohesive
  configuration contract.
- Use `map(object(...))` for repeated named configurations.
- Use `optional(...)` object attributes where Terraform version support allows
  clear optional configuration.
- Use validation blocks for allowed values, naming patterns, CIDR formats,
  mutually exclusive options, minimum lengths, maximum lengths, and other
  caller-facing constraints.
- Use `nullable = false` for variables that must never be null.
- Prefer clear variable names over abbreviations.
- Do not expose variables that mirror every provider argument if the module is
  intended to enforce a platform pattern.

Defaults are allowed when:

- The default is secure and broadly valid.
- The default represents a platform standard.
- The default reduces caller noise without hiding an important decision.
- The caller can override the value when appropriate.

Defaults are prohibited when:

- The value is environment-specific.
- The value contains a tenant ID, subscription ID, region, address range,
  workload name, business unit, cost center, or owner.
- The value controls a material security, networking, identity, or governance
  decision that callers must make intentionally.
- A default would cause unexpected resource creation.

Sensitive values:

- Variables that accept secrets or credentials must be marked `sensitive`.
- Long-lived secrets should not be accepted unless the module has a documented
  requirement and no better identity pattern is available.
- Prefer managed identity, workload identity federation, Azure RBAC, and
  Key Vault references over raw secret values.

## Output Standards

Outputs are module contracts. They should expose values that consumers need for
composition, deployment outputs, operational visibility, or documented
cross-module integration.

Required standards:

- Every output must include a `description`.
- Outputs must be named clearly and consistently.
- Outputs must expose useful contracts such as resource IDs, names, principal
  IDs, endpoint addresses, diagnostic workspace IDs, subnet IDs, or policy
  assignment IDs when those values are expected integration points.
- Outputs must not expose implementation detail that callers should not depend
  on.
- Outputs containing secrets or sensitive identifiers must be marked
  `sensitive`.
- Avoid outputting entire resource objects unless there is a strong documented
  reason.
- Avoid outputs that encourage remote state to become an undocumented
  integration contract.

Outputs must remain stable across minor and patch releases unless a breaking
change is intentionally released through semantic versioning.

## Naming Standards

Names must be predictable, readable, and consistent across modules.

Variable naming:

- Use lowercase snake_case.
- Use nouns or noun phrases.
- Use plural names for collections.
- Use positive boolean names such as `enable_diagnostics`, not
  `disable_diagnostics`.
- Avoid unclear abbreviations.

Output naming:

- Use lowercase snake_case.
- Name outputs by the contract they expose, such as `resource_group_id`,
  `subnet_ids`, or `principal_id`.
- Avoid provider-specific internal names unless they are the expected Azure
  contract.

Local naming:

- Use lowercase snake_case.
- Use locals for derived values, normalization, and repeated expressions.
- Do not use locals to hide required caller decisions.

Resource naming:

- Terraform resource labels must be descriptive and stable.
- Use `this` only when the module manages a single primary resource and the
  meaning is obvious.
- Use descriptive labels when a module manages multiple resources of the same
  type.
- Resource labels must not include environment names, regions, or instance
  numbers unless those are part of a repeated internal pattern.

File naming:

- Use standard Terraform file names for common concerns:
  `main.tf`, `variables.tf`, `outputs.tf`, `versions.tf`, and `locals.tf`.
- Use focused filenames such as `diagnostics.tf`, `role_assignments.tf`, or
  `private_endpoints.tf` only when they improve readability.
- Do not create many tiny files that make the module harder to review.

Azure resource naming:

- Modules may accept a fully formed name or accept naming components based on
  the platform naming standard.
- Modules must not invent environment-specific names.
- Modules must not hard-code organization, workload, environment, region, or
  subscription naming values.

## Tag Handling

Modules must accept tags but must not invent environment-specific tag values.

Required standards:

- Modules that create taggable resources must expose a `tags` variable unless a
  documented Azure resource limitation prevents tagging.
- The `tags` variable must be typed as `map(string)`.
- The default may be `{}` when tags are optional for the module.
- Modules may merge required module-level tags only when those tags are
  platform-defined and documented.
- Modules must not hard-code business owner, cost center, environment,
  application, data classification, or operational support tags.
- Tag validation should enforce required formats only when the platform tagging
  standard defines controlled values.

Deployment repositories are responsible for supplying environment-specific tags
and ensuring those tags align with the platform tagging standard.

## Module Composition

Module composition must reduce real complexity. It must not create abstraction
for its own sake.

When to wrap Azure Verified Modules:

- Wrap an Azure Verified Module when the platform needs to enforce standard
  naming, tagging, diagnostics, locks, private networking, identity, RBAC, or
  policy-related configuration.
- Wrap an Azure Verified Module when doing so provides a stable platform
  interface that deployment repositories can consume consistently.
- Wrap an Azure Verified Module when the wrapper materially reduces repeated
  configuration across foundation or connectivity deployments.

When to compose multiple modules:

- Compose modules when a higher-level platform capability requires several
  resources to be deployed together consistently.
- Compose modules when the composition creates a meaningful platform contract,
  not merely a pass-through wrapper.
- Compose modules when it reduces repeated root deployment logic while
  preserving clear ownership and predictable plans.

When not to wrap Azure Verified Modules:

- Do not wrap an Azure Verified Module if the wrapper only renames inputs and
  outputs without enforcing a platform standard.
- Do not wrap an Azure Verified Module if the wrapper hides important
  operational controls.
- Do not wrap an Azure Verified Module if direct use is clearer, safer, and
  still conforms to platform standards.
- Do not wrap an Azure Verified Module if its interface is unstable for the
  platform use case.

Avoid unnecessary abstraction:

- Do not create modules that only call one resource with all arguments exposed.
- Do not create modules that obscure the Azure resource model without adding a
  platform contract.
- Do not create generic "do everything" modules.
- Do not make consumers reverse-engineer hidden defaults or implicit resource
  creation.

## Testing Requirements

Every module must meet the minimum validation baseline before release.

Required checks:

- `terraform fmt` must pass.
- `terraform validate` must pass for supported examples.
- TFLint must pass using the platform linting configuration.
- Security scanning must pass using the platform-approved scanner
  configuration.
- Module examples must initialize successfully.
- Basic deployment validation must be possible through at least one example or
  test fixture.
- Tests must verify required input validation, expected outputs, and important
  optional behaviors where practical.

Testing must use Terraform as the authoritative engine, per ADR 0001. OpenTofu
compatibility is not part of the required module test matrix.

This standard defines required validation outcomes only. It does not prescribe
CI/CD implementation details.

## Documentation Requirements

Every module must include a `README.md` that documents:

- Purpose: what the module creates and why it exists.
- Usage: basic example showing expected module consumption.
- Inputs: variable names, descriptions, types, defaults, and required values.
- Outputs: output names, descriptions, and sensitivity.
- Examples: links to `examples/basic` and `examples/complete` where applicable.
- Version compatibility: supported Terraform and provider version ranges.
- Notes: operational or design notes that affect consumers.
- Limitations: unsupported scenarios, assumptions, and known constraints.

Documentation must be written for platform engineers who consume modules from
deployment repositories. It must be concise, accurate, and aligned with the
module interface.

Documentation must not include real tenant IDs, subscription IDs, secrets,
private IP allocations, production names, or customer-specific values.

## Versioning

Reusable modules must use semantic versioning.

Version format:

```text
MAJOR.MINOR.PATCH
```

Version expectations:

- Increment `MAJOR` for breaking changes.
- Increment `MINOR` for backward-compatible new functionality.
- Increment `PATCH` for backward-compatible fixes, documentation corrections,
  and internal implementation changes that do not change the module contract.

Breaking changes include:

- Removing a variable.
- Renaming a variable.
- Changing a variable type incompatibly.
- Removing an output.
- Renaming an output.
- Changing output meaning or structure incompatibly.
- Changing default behavior in a way that can replace, destroy, or materially
  reconfigure resources.
- Changing resource addressing in a way that causes replacement unless a
  migration path is documented.
- Tightening validation in a way that rejects previously valid supported input.
- Changing required provider versions in a way that forces consumers to upgrade
  outside the documented platform range.

Non-breaking changes may include:

- Adding an optional variable with a safe default.
- Adding an output.
- Adding support for a new optional feature that is disabled by default.
- Fixing documentation.
- Refactoring internals without changing resource addresses or behavior.

Every release must include release notes that identify breaking changes,
migration steps, and notable operational impacts.

## Anti-Patterns

The following patterns are prohibited:

- Hard-coded subscription IDs.
- Hard-coded tenant IDs.
- Hard-coded regions.
- Hard-coded resource names for real environments.
- Hard-coded address ranges or enterprise IP allocations.
- Environment-specific naming inside reusable module code.
- Business-unit, workload, owner, cost center, or support tags embedded in a
  module.
- Provider blocks in child modules.
- Backend blocks in child modules.
- Terraform state files or plan files committed to the module repository.
- Long-lived credentials, secrets, private keys, or tokens in module code,
  examples, tests, or documentation.
- Magic values that are not documented or exposed through clear inputs.
- Hidden resource creation controlled only by locals or implicit behavior.
- Giant "do everything" modules.
- Modules that expose every provider argument without enforcing a platform
  pattern.
- Modules that require consumers to modify module source for environment use.
- Outputs that expose entire resource objects without a documented need.
- Examples that use real production identifiers.

## Module Review Checklist

Use this checklist during pull request review:

- [ ] The module has a clear, focused purpose.
- [ ] The module belongs in `azure-platform-modules` and is not a root
      deployment.
- [ ] Required files and directories are present or omissions are justified.
- [ ] No provider blocks are present in the child module.
- [ ] No backend blocks are present in the child module.
- [ ] `versions.tf` declares Terraform and provider requirements.
- [ ] Variables are strongly typed and documented.
- [ ] Required variables do not have unsafe or environment-specific defaults.
- [ ] Validation blocks are present where they would prevent invalid input.
- [ ] Outputs are intentional, documented, and not excessive.
- [ ] Sensitive variables and outputs are marked sensitive.
- [ ] Naming follows the module naming expectations.
- [ ] Tags are accepted for taggable resources.
- [ ] The module does not invent environment-specific tag values.
- [ ] No tenant IDs, subscription IDs, regions, address ranges, or real
      environment names are hard-coded.
- [ ] Azure Verified Module usage is justified and does not hide required
      controls.
- [ ] Module composition reduces meaningful duplication or enforces a platform
      contract.
- [ ] Examples are present and use placeholder values only.
- [ ] Terraform formatting and validation pass.
- [ ] TFLint and security scanning pass or documented exceptions are approved.
- [ ] README documentation is accurate and complete.
- [ ] Version impact is identified as major, minor, or patch.
- [ ] Breaking changes include migration guidance.
- [ ] The module can be consumed without modifying module source.

## Definition of Done

A reusable production-quality module is complete when it:

- Has a focused platform purpose and clear ownership.
- Follows the required directory structure.
- Uses Terraform as the authoritative supported engine.
- Declares Terraform and provider version requirements.
- Contains no provider or backend configuration.
- Exposes explicit, typed, validated, and documented inputs.
- Exposes intentional, documented outputs.
- Avoids hidden behavior and unnecessary abstraction.
- Produces predictable plans and idempotent deployments.
- Uses secure defaults where defaults are appropriate.
- Accepts tags without inventing environment-specific values.
- Contains no tenant-specific, subscription-specific, region-specific,
  address-specific, workload-specific, or secret values.
- Includes useful basic and complete examples.
- Passes formatting, validation, linting, security scanning, and module test
  expectations.
- Includes accurate README documentation.
- Follows semantic versioning.
- Can be consumed by deployment repositories through an immutable released
  version.
- Can be reviewed, upgraded, and operated by another qualified platform engineer
  without relying on undocumented knowledge.
