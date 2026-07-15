# Versioning Standard

## Purpose

This standard defines the versioning and dependency strategy for the Azure
Platform Framework.

It covers:

- Terraform CLI.
- Reusable Terraform modules.
- AzureRM provider.
- AzAPI provider.
- Other Terraform providers.
- Dependency lock files.
- Root deployment repositories.
- Reusable child modules.

Terraform is the authoritative and officially supported Infrastructure as Code
engine for this platform, per ADR 0001. OpenTofu compatibility is not part of
the required release, validation, or support matrix unless a future ADR changes
that decision.

This document does not select exact Terraform or provider versions. Exact
version selections require documented rationale and belong in M1 engineering
toolchain work, implementation repository setup, or a future ADR when the
decision has architecture-level impact.

## Scope

This standard applies to:

- `azure-platform-modules`.
- `azure-platform-foundation`.
- `azure-platform-connectivity`.
- Module examples and tests.
- Terraform provider constraints.
- Terraform CLI constraints.
- `.terraform.lock.hcl` ownership.
- Module source references used by deployment repositories.
- Release notes, changelogs, and upgrade guidance for reusable modules.

This standard does not create Terraform code, workflows, releases, tags, or
implementation repositories.

## Versioning Principles

Required principles:

- Terraform is the only supported execution engine.
- Root deployments optimize for reproducibility.
- Reusable child modules optimize for compatibility within the approved
  platform range.
- Deployment repositories consume immutable module versions.
- Provider upgrades are explicit, reviewed, tested, and reversible.
- Lock files belong where repeatable deployment execution is required.
- Module releases must communicate compatibility, behavior changes, and
  migration steps.
- Version constraints must be as tight as necessary and as loose as safely
  supportable.
- Exact versions must not be selected without rationale, validation, and an
  upgrade path.

## Terraform CLI Strategy

The platform distinguishes between two Terraform CLI concepts:

- Approved execution version: the Terraform CLI version used by CI/CD,
  controlled apply workflows, release validation, and recommended developer
  tooling.
- Minimum supported module version: the lowest Terraform CLI version a reusable
  module declares because of the Terraform language features it actually uses.

### Approved Execution Version

The approved execution version is the operational Terraform version for the
platform. CI/CD runners, plan jobs, apply jobs, release validation, and
developer setup documentation should use this version or an explicitly approved
patch-level variant.

The platform should pin an exact Terraform CLI version for CI/CD and controlled
apply workflows. This reduces ambiguity in plan output, provider installation,
lock file behavior, test behavior, and incident troubleshooting.

Developer tooling should use the same approved execution version wherever
practical. If local development allows a narrow compatible range, CI/CD remains
authoritative and must catch differences before merge or release.

No exact version is selected in this standard. The exact version must be chosen
in M1 with rationale that considers:

- Current Terraform support status.
- Compatibility with required providers.
- Compatibility with native Terraform tests used by the platform.
- Compatibility with Azure Verified Modules selected for adoption.
- Known defects, security issues, and upgrade notes.
- Enterprise workstation and CI runner support.
- Time available for non-production validation before production use.

### Minimum Supported Module Version

Reusable child modules should declare the minimum Terraform version required by
the features they actually use.

Examples of feature-driven constraints include Terraform language syntax,
variable validation behavior, optional object attributes, native test features,
or provider requirement syntax. A module must not claim support for a Terraform
version that cannot run its code or tests.

Reusable modules should avoid unnecessary upper bounds on Terraform CLI
versions. Upper bounds are allowed only when there is a documented
compatibility risk, known Terraform defect, unsupported language behavior, or
platform-wide decision that requires the bound.

### CLI Upgrade Evaluation

Terraform CLI upgrades must be evaluated as platform upgrades.

Required evaluation:

- Review Terraform release notes and upgrade guides.
- Confirm provider compatibility.
- Confirm native Terraform test compatibility.
- Validate representative reusable modules.
- Validate representative foundation and connectivity root deployments.
- Confirm lock file behavior and provider resolution.
- Compare plan output for unexpected changes.
- Run security, linting, and documentation checks where applicable.
- Promote through non-production before production use.
- Update developer setup, CI runner configuration, and runbooks together.

CLI upgrades should be introduced through explicit pull requests in affected
repositories. Pull requests must include rationale, validation evidence,
expected impact, rollback notes, and affected module or root deployment scope.

## Child Module Constraints

Reusable child modules in `azure-platform-modules` must declare Terraform and
provider requirements without configuring providers or backends.

Required child module rules:

- Declare `required_version` based on Terraform features actually used.
- Avoid unnecessary Terraform CLI upper bounds.
- Declare provider source addresses in `required_providers`.
- Declare minimum compatible provider versions.
- Avoid tightly pinning provider patch versions.
- Do not configure provider blocks.
- Do not configure backend blocks.
- Do not assume environment-specific provider aliases unless the caller passes
  them explicitly.

Provider constraints in child modules should express compatibility, not
deployment reproducibility. Child modules should not force a provider patch
version unless there is a documented compatibility, security, or provider
behavior reason.

Child module examples may require providers for validation, but those examples
must still avoid real tenant IDs, subscription IDs, regions, credentials, or
environment-specific values.

## Root Deployment Constraints

Root deployment repositories own execution reproducibility.

Foundation and connectivity root deployments should use deliberate provider
constraints that reflect the approved platform provider baseline. Root
deployments may use tighter constraints than reusable child modules because root
deployments represent real platform environments, state, blast radius, and
controlled apply workflows.

Required root deployment rules:

- Configure providers only in root modules.
- Configure backends only in root modules.
- Commit dependency lock files for root deployments.
- Pin or constrain providers deliberately according to the approved baseline.
- Consume reusable modules through immutable source references.
- Upgrade provider constraints through explicit pull requests.
- Include plan output or plan summaries for dependency upgrades.
- Promote provider upgrades through lower-risk environments before production.
- Record consumed module versions and provider changes in deployment notes or
  release records.

Root deployment provider upgrade pull requests must summarize:

- Provider versions before and after.
- Module versions affected.
- Terraform CLI version used for validation.
- Upgrade guide and release note review.
- Plan impact.
- Known risks.
- Rollback approach.

## Provider Constraint Strategy

Provider constraints must balance compatibility and operational safety.

### Reusable Child Modules

Reusable child modules should declare broad minimum compatible constraints.

Expected pattern:

- Minimum provider version based on the resource features, arguments, data
  sources, and behaviors the module uses.
- No tight patch-level pin unless justified.
- No unnecessary upper bound unless a known breaking provider change or
  platform decision requires it.

This keeps modules consumable by foundation, connectivity, and future
deployment repositories that may upgrade providers at different times within
the approved platform range.

### Root Deployments

Root deployments should use tighter, deliberate provider constraints and commit
lock files.

Expected pattern:

- Provider constraints define the allowed upgrade window.
- `.terraform.lock.hcl` records the exact selected provider versions.
- Provider updates happen through explicit pull requests.
- Plan and validation evidence accompany dependency changes.

This gives reproducibility without forcing every reusable module to publish a
single provider patch version as a consumer contract.

## AzureRM Provider Strategy

AzureRM is the default provider for Azure resource management when it supports
the required Azure capability and operational behavior.

AzureRM provider upgrades must be assessed deliberately, especially major
version upgrades.

Required review for major AzureRM upgrades:

- Review official upgrade guides.
- Review release notes for breaking changes, deprecated resources, changed
  defaults, removed arguments, state migrations, and provider behavior changes.
- Validate representative modules and examples.
- Validate representative root deployments with real backend and provider
  configuration in non-production where available.
- Review plan output for replacements, drift, or default changes.
- Confirm Azure Verified Module compatibility where AVM modules are used.
- Confirm policy, diagnostics, identity, networking, and private endpoint
  behavior where relevant.

Major provider upgrades should not be bundled with unrelated architecture,
module interface, or environment changes unless the coupling is intentional and
documented.

## AzAPI Provider Strategy

AzAPI is acceptable when AzureRM does not yet support a required Azure resource,
property, API version, or operational capability needed by the platform.

Acceptable AzAPI use cases:

- Azure capability is required and not available in AzureRM.
- AzureRM support exists but lacks a required property or API behavior.
- A time-sensitive platform capability requires direct ARM API coverage.
- A documented module design requires temporary AzAPI use until AzureRM support
  matures.

AzAPI must not be used solely to bypass deliberate AzureRM behavior without
justification. If AzureRM enforces validation, lifecycle behavior, or resource
model constraints, bypassing those constraints with AzAPI requires a documented
reason and review.

AzAPI usage must document:

- Why AzureRM is insufficient.
- Which API version is used.
- Expected migration path to AzureRM if one exists.
- State and replacement risks.
- Validation and testing approach.
- Operational limitations or troubleshooting notes.

AzAPI provider upgrades require release note review and validation because API
schema behavior, provider behavior, and Azure service behavior may change.

## Other Providers

Other providers may be used when required for platform functionality, such as
identity, random value generation, testing helpers, or utility behavior.

Rules for other providers:

- Use providers only when they have a clear platform purpose.
- Declare source addresses and minimum compatible versions in child modules.
- Use tighter root constraints and lock files in deployment repositories.
- Review release notes and upgrade guides before major upgrades.
- Avoid providers that introduce credentials, network access, or operational
  dependencies without documented need.
- Do not add providers to reusable modules only for convenience if Terraform
  language features or root configuration can satisfy the requirement cleanly.

## Dependency Lock Files

Terraform dependency lock files serve different purposes in root deployments
and reusable module repositories.

### Root Deployment Lock Files

`.terraform.lock.hcl` must be committed for real root deployments in
`azure-platform-foundation` and `azure-platform-connectivity`.

Root deployment lock files provide:

- Reproducible provider selection.
- Reviewable provider upgrades.
- Consistent CI/CD and local plan behavior.
- Safer apply operations.
- Auditability during incidents and rollback.

Root deployment lock files are part of the deployment contract for that root
module and state boundary.

### Reusable Module Lock Files

Reusable module repositories may generate lock files during validation,
examples, and tests. Those lock files are not necessarily consumer contracts.

Reusable modules should not publish module-level lock files as the primary
dependency contract unless the module repository deliberately chooses that
pattern for examples or test fixtures. The reusable module contract is defined
by `required_version`, `required_providers`, inputs, outputs, documentation,
tests, and semantic versioning.

Module examples may commit lock files if the module repository standard later
requires reproducible example validation. If committed, example lock files must
not imply that consumers are required to use those exact provider selections in
their root deployments.

### Reproducibility Versus Compatibility

Root deployment reproducibility means the same root deployment should install
the same provider versions during plan and apply until a reviewed dependency
upgrade changes them.

Reusable module compatibility means the module should work across an approved
range of Terraform and provider versions. A reusable child module should not
use lock files to narrow consumer compatibility unless there is a documented
reason.

## Semantic Versioning

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

### Shared Repository Version Versus Independent Module Versions

The module repository can use one shared version for the entire repository or
independent versions for each module.

#### Shared Repository Version

In this approach, one semantic version tag represents the entire module
repository.

Advantages:

- Simple release discovery.
- Easy to reference a consistent module catalog baseline.
- Straightforward repository-level changelog.
- Useful when modules are tightly coupled and released together.

Disadvantages:

- Every module appears to change when only one module changes.
- Consumers may upgrade unrelated modules unnecessarily.
- Breaking changes in one module can force a major version signal for the whole
  repository.
- Release cadence can slow down as the module catalog grows.

#### Independent Module Versions

In this approach, each module has its own semantic version and release notes.

Advantages:

- Clearer compatibility contract per module.
- Smaller, safer upgrades for deployment repositories.
- Better support for independent module ownership and release cadence.
- Breaking changes are scoped to the affected module.
- Consumers can adopt fixes without upgrading unrelated modules.

Disadvantages:

- More release metadata to manage.
- Requires consistent tag naming, changelog structure, and release automation.
- Cross-module compatibility must be documented when modules are designed to
  work together.
- Discovery can be harder without clear repository documentation.

### Recommendation

Use independent module versions for `azure-platform-modules`.

Reusable modules are platform product artifacts with stable interfaces. They
will not all change at the same cadence. Independent module versioning best
supports immutable consumption, focused upgrades, safer rollback, and clearer
breaking-change communication.

The exact tag naming convention for independent module releases is deferred to
M3 module repository implementation. It must be documented before releases are
created and must support unambiguous module and version identification.

## Terraform Module Version Impact

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
- Raising minimum Terraform or provider versions outside the approved platform
  compatibility range.
- Replacing AzureRM resources with AzAPI resources, or the reverse, when state
  migration or behavior changes affect consumers.
- Changing naming, tagging, diagnostics, identity, RBAC, private networking, or
  policy behavior in a way that affects deployed resources.

Minor changes include:

- Adding an optional variable with a safe default.
- Adding an output.
- Adding support for an optional feature that is disabled by default.
- Expanding allowed values without changing existing behavior.
- Adding support for a newer provider feature while preserving compatibility.
- Adding tests, examples, or documentation for supported behavior.

Patch changes include:

- Fixing defects without changing the public contract.
- Correcting documentation.
- Refactoring internals without changing resource addresses or behavior.
- Improving validation messages without changing accepted supported input.
- Updating tests that do not change module behavior.

## Module Source References

Deployment repositories must consume reusable modules through immutable source
references.

Required rules:

- Use immutable tags or commit SHAs for production deployment references.
- Do not reference mutable branches such as `main` for production deployments.
- Do not consume local relative module source from `azure-platform-modules` in
  production root deployments.
- Do not move or rewrite released module tags.
- Record consumed module versions in deployment documentation or release notes.

Module upgrade pull requests must include:

- Current module version.
- Target module version.
- Changelog or release note summary.
- Breaking changes and migration steps where applicable.
- Terraform CLI and provider versions used for validation.
- Plan summary and expected impact.
- Rollback approach.

Rollback expectations:

- Rollback should generally revert the deployment repository to the previously
  consumed module version and provider lock file state.
- Rollback must consider state migrations, resource replacements, and provider
  behavior changes.
- Breaking module upgrades must include explicit rollback or recovery guidance.

## Testing Matrix

The platform testing model distinguishes between supported module compatibility
and approved execution behavior.

Definitions:

- Minimum supported Terraform version: the lowest Terraform version declared by
  a reusable module based on features it actually uses.
- Approved execution version: the exact Terraform version used by CI/CD,
  release validation, controlled applies, and recommended developer tooling.

Reusable modules should be validated with:

- The approved execution version.
- The minimum supported Terraform version where practical and where tooling
  supports matrix validation.
- Provider versions selected by representative root constraints and lock files.
- Provider versions at or near the declared minimum when practical for module
  compatibility testing.

Provider compatibility is validated through:

- `terraform init`.
- `terraform validate`.
- Native Terraform tests where applicable.
- Example initialization and validation.
- Linting and security scanning.
- Representative root deployment plans for integration-sensitive modules.

Native Terraform tests must run under the approved execution version. If a
module declares a minimum Terraform version lower than the version required for
native Terraform tests, the module README or test documentation must explain
the distinction between module runtime compatibility and test harness
compatibility.

The final CI/CD testing matrix is deferred to M1 and M3 implementation work.

## Release Process

Reusable module releases must be deliberate, immutable, and documented.

### Prerelease Versions

Prerelease versions may be used before a module interface is stable or when a
change needs validation before stable release.

Examples:

```text
1.0.0-alpha.1
1.0.0-beta.1
1.0.0-rc.1
```

Prerelease versions:

- Must not be used for production deployments unless a documented exception is
  approved.
- Must clearly identify instability or release-candidate status.
- Must include release notes.
- Must not be treated as stable compatibility contracts.

### Stable Releases

Stable releases represent supported module contracts.

Stable release requirements:

- Terraform formatting and validation pass.
- Module tests pass according to the approved testing matrix.
- Examples initialize and validate where required.
- Linting and security scanning pass or documented exceptions are approved.
- README documentation is accurate.
- Version impact is identified.
- Changelog and release notes are published.
- Breaking changes include migration and rollback guidance.

### Changelog Expectations

Each reusable module must maintain changelog or release note history sufficient
for deployment repositories to evaluate upgrades.

Changelog entries should include:

- Added functionality.
- Changed behavior.
- Deprecated inputs or outputs.
- Removed inputs or outputs.
- Fixed defects.
- Provider or Terraform compatibility changes.
- Migration steps.
- Known risks.

### Rollback Expectations

Module releases and deployment upgrades must be reversible where practical.

Rollback planning must consider:

- Previous module version.
- Previous provider lock file.
- State migrations.
- Resource replacements.
- Changed defaults.
- Provider major version behavior.
- Azure API behavior.
- Manual remediation steps if rollback cannot be fully automated.

## Best Practices

- Pin exact Terraform CLI versions for CI/CD and controlled applies.
- Keep child module constraints broad enough for compatibility.
- Keep root deployment lock files committed and reviewed.
- Upgrade providers through dedicated pull requests.
- Separate provider upgrades from unrelated functional changes.
- Review Terraform and provider release notes before upgrades.
- Prefer AzureRM when it supports the required Azure capability.
- Use AzAPI only with documented justification and migration notes.
- Use immutable module references in deployment repositories.
- Keep module changelogs clear and consumer-focused.
- Test module upgrades in non-production before production rollout.

## Anti-Patterns

The following patterns are prohibited:

- Advertising OpenTofu compatibility as supported without a future ADR.
- Referencing `main` or another mutable branch from production root
  deployments.
- Moving or rewriting released module tags.
- Committing Terraform state or plan files to Git.
- Configuring providers or backends inside reusable child modules.
- Tightly pinning provider patch versions inside child modules without a
  documented reason.
- Adding unnecessary Terraform CLI upper bounds to child modules.
- Bundling provider major upgrades with unrelated platform changes.
- Using AzAPI to bypass AzureRM behavior without justification.
- Treating module example lock files as mandatory consumer lock files.
- Releasing breaking changes without migration guidance.
- Upgrading providers without reviewing upgrade guides and release notes.

## Review Checklist

Use this checklist during architecture, module, dependency, and deployment
reviews:

- [ ] Terraform is treated as the authoritative execution engine.
- [ ] The change distinguishes approved execution version from minimum
      supported module version.
- [ ] Child modules declare only the Terraform minimum required by actual
      features used.
- [ ] Child modules avoid unnecessary Terraform CLI upper bounds.
- [ ] Child modules declare provider source addresses and minimum compatible
      provider versions.
- [ ] Child modules do not tightly pin provider patch versions without
      documented reason.
- [ ] Providers and backends are configured only in root modules.
- [ ] Root deployments use deliberate provider constraints.
- [ ] Root deployment `.terraform.lock.hcl` changes are reviewed.
- [ ] Provider upgrades are explicit and include release note or upgrade guide
      review.
- [ ] AzureRM is preferred where it supports the required capability.
- [ ] AzAPI usage is justified and documented.
- [ ] Deployment repositories use immutable module tags or commit SHAs.
- [ ] Production deployments do not reference mutable branches.
- [ ] Module version impact is identified as major, minor, patch, or
      prerelease.
- [ ] Breaking changes include migration and rollback guidance.
- [ ] Native Terraform tests run under the approved execution version.
- [ ] Changelog and release notes are updated where required.

## Definition of Done

Versioning and dependency management are compliant when:

- Terraform CLI version policy is documented for CI/CD, developer tooling, and
  module compatibility.
- Reusable modules declare feature-based Terraform minimum versions and
  compatible provider constraints.
- Root deployments own provider selection, provider configuration, backend
  configuration, and lock files.
- `.terraform.lock.hcl` is committed for real root deployments.
- Reusable module lock file behavior is documented and does not override
  consumer root dependency control.
- Deployment repositories consume modules through immutable references.
- `azure-platform-modules` releases modules with semantic versioning,
  changelogs, release notes, and upgrade guidance.
- Provider upgrades are explicit pull requests with validation evidence.
- AzureRM and AzAPI use follows the provider strategy in this standard.
- Testing validates both module compatibility and approved execution behavior
  according to the implemented matrix.
- Rollback expectations are documented for module and provider upgrades.

## Decisions Still Requiring ADRs or M1 Implementation

The following decisions remain open:

- Exact approved Terraform CLI execution version.
- Approved Terraform CLI version range for developer tooling.
- Minimum Terraform version supported by the initial module catalog.
- AzureRM provider baseline version and allowed upgrade window.
- AzAPI provider baseline version and allowed upgrade window.
- AzureAD and other provider baseline versions, if required.
- Whether root deployments use exact provider constraints, pessimistic
  constraints, or another approved constraint pattern.
- Whether module examples commit `.terraform.lock.hcl`.
- Exact independent module tag naming convention.
- CI/CD matrix for approved execution version, minimum supported module
  version, provider minimums, and provider baselines.
- Native Terraform test version requirements.
- Release automation and changelog format for `azure-platform-modules`.
- Dependency update cadence and ownership.
- Emergency provider or Terraform CLI upgrade process.

These decisions should be resolved during M1 engineering toolchain design and
M3 module repository implementation. Architecture-level tradeoffs should be
recorded in ADRs when they constrain repository, release, state, or operational
behavior.
