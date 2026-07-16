# ADR 0003 - Terraform Toolchain Baseline

## Status

Accepted

## Decision Date

2026-07-16

## Owners

Ordicor Platform

## Context

ADR 0001 establishes Terraform as the authoritative and officially supported
Infrastructure as Code engine for the Azure Platform Framework. ADR 0002
establishes the four-repository model, separating architecture, reusable
modules, foundation deployments, and connectivity deployments.

The versioning standard defines the platform dependency strategy:

- CI/CD and developer tooling should use one exact approved Terraform CLI
  version.
- Reusable child modules should declare minimum compatible versions rather than
  exact provider patch pins.
- Root deployment repositories own provider configuration, backend
  configuration, and committed dependency lock files.
- Root deployments may use tighter constraints than reusable child modules.
- Provider upgrades must happen through explicit pull requests with validation
  evidence.

This ADR selects the initial supported Terraform and Azure provider toolchain
baseline for M1 engineering toolchain work and M3 reusable module work. It does
not create Terraform files, workflows, releases, tags, or implementation
repositories.

Verified current facts used as inputs:

- Terraform is the authoritative IaC engine.
- Native Terraform provider mocking requires Terraform 1.7.0 or later.
- The current AzureRM provider major version is 4.x.
- The current official AzureRM release is 4.80.0 as of July 2026.
- Root deployment repositories will own committed dependency lock files.
- Reusable modules should declare minimum compatible versions rather than exact
  provider patch pins.
- CI/CD and developer tooling should use one exact approved Terraform CLI
  version.

The official Terraform install page lists Terraform 1.15.8 as the latest
available stable release as of this ADR. Terraform 1.15.8 is not a prerelease:
https://developer.hashicorp.com/terraform/install

The HashiCorp releases index for the AzureRM provider lists AzureRM 4.80.0 in
the 4.x release series, preserving the externally verified release reference
used for this baseline decision:
https://releases.hashicorp.com/terraform-provider-azurerm/

The first foundation root initialization selected AzureRM 4.81.0 under the
approved root constraint `~> 4.80`. This updates the initial
release-validation and lock-file-selected provider version without changing the
root constraint pattern or reusable child-module compatibility constraint.

## Decision Drivers

- Use Terraform as the single supported engine.
- Prefer a current stable Terraform release rather than the minimum version
  required for native provider mocking.
- Distinguish exact execution and release-validation versions from reusable
  module compatibility constraints.
- Keep the initial release-acceptance matrix small enough to implement during
  M1.
- Avoid claiming compatibility with Terraform and provider combinations that
  are not validated before stable module release.
- Support native Terraform tests and provider mocking from the beginning.
- Align module validation, release validation, CI/CD, plans, applies, and
  developer tooling on one operational version.
- Use AzureRM 4.x as the initial Azure provider baseline.
- Keep reusable modules environment-neutral and broadly compatible within the
  validated platform baseline.
- Let root deployment repositories own exact provider selections through lock
  files.
- Avoid adding AzAPI until a real platform capability requires it.

## Options Considered

1. Minimal Terraform baseline: approve Terraform 1.7.x because native provider
   mocking requires Terraform 1.7.0 or later.
2. Current stable Terraform baseline: approve Terraform 1.15.8 for execution
   and initial module compatibility.
3. Split baseline: approve Terraform 1.15.8 for execution while declaring a
   lower module compatibility floor such as Terraform 1.7.x.

## Option 1 - Minimal Terraform Baseline

### Description

Use the oldest Terraform version that supports native provider mocking as the
initial approved version.

### Advantages

- Maximizes theoretical compatibility with older Terraform installations.
- Meets the minimum provider mocking requirement.
- Reduces the chance that modules use newer Terraform language features too
  early.

### Disadvantages

- Starts the platform on an older Terraform line.
- Requires additional analysis to determine which newer fixes, test
  improvements, and language behavior are unavailable.
- Does not align with the preference for a current stable release.
- May increase friction with current provider and module ecosystem behavior.
- Provides little practical value because the platform is new and has no
  existing older Terraform consumers.

## Option 2 - Current Stable Terraform Baseline

### Description

Use Terraform 1.15.8 as both the exact approved execution version and the
initial minimum supported module version.

### Advantages

- Uses a current stable Terraform release.
- Avoids prerelease tooling.
- Supports native Terraform tests and provider mocking.
- Gives developers, CI, test jobs, plans, applies, and release validation one
  exact version.
- Avoids claiming untested compatibility with older Terraform versions.
- Reduces initial CI matrix complexity.
- Aligns with a new platform that has no legacy Terraform runtime requirement.

### Disadvantages

- Modules initially require the same Terraform version used for execution.
- Older Terraform installations cannot consume platform modules unless they
  upgrade.
- Future lowering of the compatibility floor would require deliberate testing.
- Current-version adoption concentrates early validation on one Terraform
  version.

## Option 3 - Split Baseline

### Description

Use Terraform 1.15.8 for execution and release validation while declaring a
lower reusable module compatibility floor, such as Terraform 1.7.x.

### Advantages

- Preserves optionality for consumers using older Terraform versions.
- Allows current CI/CD execution while suggesting broader module compatibility.
- Keeps provider mocking available if the floor is at least 1.7.0.

### Disadvantages

- Requires the declared minimum combination to be tested before stable module
  releases claim compatibility.
- Risks claiming support for versions not validated in CI if the matrix is not
  implemented.
- Forces module authors to reason about older Terraform behavior before the
  module catalog exists.
- Adds some complexity before M1 and M3 have established the baseline
  toolchain.

## Decision

Adopt option 3: split execution and compatibility baseline.

The initial approved Terraform execution version is:

```text
Terraform 1.15.8
```

Terraform 1.15.8 is the required version for:

- Developer tooling.
- CI validation.
- Native Terraform tests.
- Provider mocking.
- Plans.
- Applies.
- Release validation.
- State operations.
- Troubleshooting and runbooks.

The initial minimum supported Terraform module version is:

```text
>= 1.7.0
```

Terraform 1.7.0 is the reusable module compatibility floor because it is a
deliberate modern baseline and supports native provider mocking if adopted by
module tests. Reusable modules must not claim stable compatibility with
Terraform 1.7.0 until the declared minimum combination is included in the test
matrix. If the project is unwilling to test Terraform 1.7.0, the compatibility
floor must be raised to the lowest Terraform version actually tested.

The initial AzureRM child-module compatibility constraint is:

```text
>= 4.0.0, < 5.0.0
```

AzureRM 4.0.0 establishes the supported provider-major contract for reusable
modules. AzureRM 5.x requires explicit compatibility review before support is
claimed.

The initial AzureRM release-validation and root example lock version is:

```text
4.81.0
```

AzAPI is not included in the initial baseline. It will be introduced later only
when a real platform capability requires an Azure resource, property, API
version, or operational behavior that AzureRM does not support.

## Rationale

Terraform 1.15.8 is selected because it is a current stable release, is newer
than the Terraform 1.7.0 provider mocking requirement, and lets the platform
start with a simple, consistent toolchain. The platform has no existing module
consumers that require an older Terraform version. Selecting one exact version
for developer tooling, CI, testing, plans, applies, and release validation
keeps early implementation work clear and reproducible.

The reusable module minimum starts at Terraform 1.7.0 because child modules
should declare the minimum supported feature baseline, not force every consumer
onto the current execution patch. Terraform 1.7.0 is modern enough to support
native provider mocking if adopted. Stable module releases must validate this
declared minimum before claiming it.

AzureRM 4.81.0 is selected as the initial release-validation and
lock-file-selected provider version because the first foundation root
initialization selected it under the approved root constraint `~> 4.80`.
Reusable child modules declare AzureRM `>= 4.0.0, < 5.0.0` because the module
compatibility contract is the supported AzureRM major version, not the exact
provider patch selected by root lock files.

Child modules and root deployments still use different constraint policies.
Child modules declare minimum compatible versions and avoid exact patch pins
unless there is a documented reason. Root constraints define the permitted
provider range. Committed root dependency lock files record the exact selected
provider version for reproducible plans, applies, and release validation.
Future provider updates require explicit dependency-upgrade pull requests with
validation evidence.

AzAPI is excluded from the initial baseline because adding it before a concrete
need would increase provider surface area, test requirements, and operational
complexity. AzureRM remains the default Azure provider.

## Constraint Patterns

These examples illustrate the recommended patterns. They are not Terraform
files and do not create implementation.

### Reusable Child Modules

Reusable child modules should declare Terraform and provider compatibility:

```hcl
terraform {
  required_version = ">= 1.7.0"

  required_providers {
    azurerm = {
      source  = "hashicorp/azurerm"
      version = ">= 4.0.0, < 5.0.0"
    }
  }
}
```

Child modules must not configure providers or backends.

### Root Deployments

Root deployments should use deliberate constraints and committed lock files:

```hcl
terraform {
  required_version = "= 1.15.8"

  required_providers {
    azurerm = {
      source  = "hashicorp/azurerm"
      version = "~> 4.80"
    }
  }
}
```

The root deployment `.terraform.lock.hcl` records the exact selected provider
version used for plan and apply. During the initial M1 baseline, the approved
root constraint `~> 4.80` defines the permitted AzureRM provider range, and
the first foundation root lock file selected AzureRM 4.81.0.

### Exact Execution Tooling

Developer setup, CI runners, test jobs, plan jobs, apply jobs, and release
validation must install and report:

```text
Terraform 1.15.8
```

## Consequences

- M1 toolchain work must install Terraform 1.15.8 for CI and developer
  tooling.
- Native Terraform tests and provider mocking can be used from the beginning.
- Reusable modules initially declare `required_version = ">= 1.7.0"`.
- Reusable modules initially declare AzureRM `>= 4.0.0, < 5.0.0` unless a
  module requires a narrower or newer constraint for a documented reason.
- Root examples and root deployment validation initially lock AzureRM 4.81.0
  after the first foundation root selected that version under `~> 4.80`.
- Root deployment repositories commit `.terraform.lock.hcl`.
- The initial release-acceptance matrix is intentionally small.
- The declared minimum Terraform and AzureRM combination must be tested before
  stable module releases claim that compatibility.
- AzAPI is not present in the initial provider baseline.

## Risks

- Terraform 1.15.8 may reveal compatibility issues with third-party tooling,
  scanners, or Azure Verified Modules during M1.
- Declaring Terraform 1.7.0 compatibility adds matrix responsibility before
  stable module releases.
- AzureRM 4.81.0 may contain behavior that later patch releases fix.
- Declaring AzureRM 4.0.0 compatibility may expose differences between early
  and later AzureRM 4.x behavior.
- If the minimum combination is not tested, the platform may overstate module
  compatibility.
- Excluding AzAPI may require a later baseline update when AzureRM lacks
  support for a required platform capability.
- A single-version initial matrix may miss compatibility issues that would be
  visible across multiple Terraform or provider versions.

## Validation Criteria

This decision is valid when:

- Developer tooling documentation identifies Terraform 1.15.8.
- CI validation reports Terraform 1.15.8.
- Native Terraform tests run under Terraform 1.15.8.
- Module release validation runs under Terraform 1.15.8.
- Plan and apply workflows use Terraform 1.15.8.
- Reusable module `required_version` constraints start at `>= 1.7.0`.
- Reusable module AzureRM constraints start at `>= 4.0.0, < 5.0.0` unless a
  documented module reason requires a narrower or newer constraint.
- Root examples and root deployments lock AzureRM 4.81.0 during the initial M1
  baseline.
- Stable module release validation includes the declared minimum combination:
  Terraform 1.7.0 with AzureRM 4.0.0.
- If Terraform 1.7.0 with AzureRM 4.0.0 is not tested, module constraints are
  raised to the lowest tested Terraform and AzureRM versions.
- Root deployment repositories commit `.terraform.lock.hcl`.
- No AzAPI provider dependency is introduced without a documented capability
  requirement.
- Documentation does not imply OpenTofu support.

## Upgrade Process

All toolchain upgrades require explicit pull requests with validation evidence.

### Terraform CLI Patch Upgrades

Terraform patch upgrades may be evaluated when a new patch release provides
bug fixes, security fixes, test improvements, or provider compatibility
improvements.

Required review:

- Review Terraform release notes.
- Run module validation and native Terraform tests.
- Validate representative root plans.
- Confirm lock file behavior.
- Update developer and CI version references together.

### Terraform CLI Minor Upgrades

Terraform minor upgrades require broader review.

Required review:

- Review Terraform release notes and upgrade guidance.
- Validate representative modules, examples, and root deployments.
- Confirm native test and provider mocking behavior.
- Confirm scanner and linting compatibility.
- Compare plan output for unexpected changes.
- Promote through non-production before production applies.

### Terraform CLI Major Upgrades

Terraform major upgrades require an ADR or explicit architecture review before
adoption.

Required review:

- Review upgrade guides and breaking changes.
- Validate state, providers, modules, tests, CI/CD, runbooks, and root
  deployments.
- Define rollback and recovery expectations.
- Coordinate across module, foundation, and connectivity repositories.

### AzureRM Patch Upgrades

AzureRM patch upgrades may be handled through explicit dependency pull
requests.

Required review:

- Review release notes.
- Update root dependency lock files.
- Run module validation.
- Run representative root plans.
- Record plan impact and rollback approach.

### AzureRM Minor Upgrades

AzureRM minor upgrades require release note review and broader validation,
especially when new resources, changed defaults, or changed validation behavior
are involved.

Required review:

- Review release notes.
- Validate module examples and tests.
- Validate representative foundation and connectivity plans.
- Confirm Azure Verified Module compatibility where applicable.
- Promote through non-production before production applies.

### AzureRM Major Upgrades

AzureRM major upgrades require an ADR or explicit architecture review.

Required review:

- Review official upgrade guides.
- Review breaking changes, deprecated resources, removed arguments, changed
  defaults, state migration behavior, and provider validation changes.
- Validate representative modules and root deployments.
- Identify migration and rollback steps.
- Avoid bundling the upgrade with unrelated platform changes.

### AzAPI Introduction

AzAPI may be introduced later through an ADR, module design note, or
architecture-approved implementation decision when AzureRM lacks required
support.

The introduction must document:

- Required Azure capability.
- Why AzureRM is insufficient.
- API version.
- Provider version baseline.
- State and replacement risks.
- Migration path back to AzureRM if expected.
- Validation and testing approach.

## Testing Matrix

The initial mandatory test matrix distinguishes compatibility validation from
release acceptance:

| Purpose | Terraform version | AzureRM version |
| --- | --- | --- |
| Minimum compatibility validation | 1.7.0 | 4.0.0 |
| Standard module validation | 1.15.8 | 4.81.0 |
| Native Terraform tests | 1.15.8 | 4.81.0 |
| Provider mocking | 1.15.8 | 4.81.0 |
| Root example validation | 1.15.8 | 4.81.0 locked |
| Release acceptance | 1.15.8 | 4.81.0 locked |

Stable module releases must not claim compatibility with Terraform 1.7.0 and
AzureRM 4.0.0 unless the minimum compatibility validation row is implemented.
If that row is not implemented, the reusable module constraints must be raised
to the lowest Terraform and AzureRM versions actually tested.

The release-acceptance path remains Terraform 1.15.8 with AzureRM 4.81.0
selected by root lock files. Additional Terraform or provider versions may be
added later through explicit dependency-upgrade pull requests if there is a
real consumer need and the CI matrix is expanded.

## Related ADRs

- [ADR 0001 - Infrastructure as Code Engine](0001-iac-engine.md)
- [ADR 0002 - Repository Separation](0002-repository-separation.md)
- [ADR 0004 - Remote State Strategy](0004-remote-state-strategy.md)
- [ADR 0006 - Deployment Identity Strategy](0006-deployment-identity-strategy.md)

## Revisit Conditions

This decision should be revisited if:

- Terraform 1.15.8 has a defect, security issue, or compatibility issue that
  materially affects the platform.
- A newer Terraform patch or minor release provides important fixes or
  capabilities.
- Required Azure Verified Modules do not support this baseline.
- AzureRM 4.81.0 has provider behavior that blocks required platform work.
- Terraform 1.7.0 or AzureRM 4.0.0 cannot be validated for stable module
  releases.
- AzureRM releases a major version that the platform needs to evaluate.
- A platform capability requires AzAPI.
- A consumer requirement justifies supporting an older Terraform version.
- M1 implementation shows that the initial test matrix is too narrow or too
  strict.
- Enterprise legal, procurement, security, or support requirements alter the
  approved Terraform or provider baseline.
