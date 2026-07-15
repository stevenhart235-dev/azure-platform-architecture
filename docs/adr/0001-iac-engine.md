# ADR 0001 - Infrastructure as Code Engine

## Status

Accepted

## Decision

Terraform is the authoritative and officially supported Infrastructure as Code
engine for this platform.

Terraform is the required engine for validation, planning, applying, state
management, CI/CD workflows, and release acceptance. A module or root deployment
is not considered platform-compliant unless it passes the required Terraform
validation and release checks.

Modules should avoid unnecessary Terraform-specific behavior when portable HCL
can satisfy the requirement cleanly. This is a maintainability preference, not a
support contract. OpenTofu compatibility is not officially supported and is not
part of the required test matrix.

OpenTofu migration or compatibility validation may be evaluated later as a
separate project if licensing, organizational policy, product direction, or
enterprise requirements justify the work.

## Context

The Azure Landing Zone roadmap defines a platform built from reusable,
standardized, and versioned Terraform/OpenTofu modules. The platform must
support configuration-driven deployments, clear separation between reusable
modules and environment deployments, Azure Verified Module adoption where
appropriate, automated validation and deployment pipelines, secured remote
state, and recoverable infrastructure changes.

Milestone 0 requires a decision on whether the platform standard is Terraform,
OpenTofu, or an explicitly tested compatibility strategy across both tools. This
decision affects module syntax, provider constraints, lock file handling,
developer workstation setup, CI/CD validation, module release requirements,
state operations, operational documentation, and future upgrade policy.

This ADR does not create Terraform code, pipelines, or implementation
repositories. It defines the IaC engine strategy that later standards and
implementation milestones must follow.

## Decision Drivers

- The platform must support reusable and independently versioned modules.
- Deployment repositories must consume immutable module versions.
- Root deployments must provide environment-specific configuration.
- Modules must avoid embedded tenant IDs, subscription IDs, regions, address
  ranges, and resource names.
- CI/CD must provide automated validation, planning, controlled apply
  operations, and clear release acceptance gates.
- State must be remote, secured, recoverable, and separated by blast radius.
- Azure Verified Modules should be adopted where they meet platform
  requirements.
- Engineers need one authoritative toolchain for documentation, troubleshooting,
  state handling, and operational support.
- The platform should reduce ambiguity before implementation repositories and
  pipelines are created.

## Options Considered

1. Terraform only.
2. OpenTofu only.
3. Terraform and OpenTofu with an explicitly tested compatibility strategy.

## Option 1 - Terraform Only

### Description

Standardize all module development, validation, planning, applying, state
management, CI/CD workflows, and release acceptance on Terraform. OpenTofu is
not a supported execution engine for this platform.

Modules may still use portable HCL patterns where those patterns are clear and
do not reduce capability, readability, or supportability. Portability is a
design preference only; Terraform behavior is authoritative.

### Advantages

- Broad enterprise adoption and familiarity across infrastructure teams.
- Mature Azure ecosystem, including documentation, examples, support paths,
  testing tools, and CI/CD integrations.
- Strong alignment with Azure Verified Modules and Terraform Registry module
  consumption patterns.
- Lower support burden because engineers, CI jobs, release gates, and runbooks
  use one engine.
- Lower testing burden because module releases do not require dual-engine
  validation.
- Clearer operational ownership for state, plans, applies, troubleshooting, and
  recovery.
- Reduced ambiguity in CI/CD, documentation, state handling, lock files, and
  release management.
- Less risk of subtle engine-specific behavior differences affecting production
  deployments.

### Disadvantages

- Creates a direct dependency on Terraform's licensing and product direction.
- Does not provide an official OpenTofu compatibility contract.
- Teams that prefer or have standardized on OpenTofu must use Terraform for
  this platform.
- A future OpenTofu migration would require a deliberate assessment and
  implementation project.
- Some portable HCL discipline could erode over time if Terraform-specific
  shortcuts are not reviewed.

### Operational Impact

Operational processes are direct and consistent. Workstations, CI runners,
module tests, plan jobs, apply jobs, state operations, release gates, and
runbooks standardize on Terraform.

This lowers ambiguity during incidents and recovery. Operators do not need to
determine which engine produced a plan, modified state, generated a lock file,
or created a release artifact. Terraform is the operational source of truth.

### Ecosystem Maturity

Terraform has the most mature ecosystem for Azure enterprise infrastructure.
Azure examples, Azure Verified Modules, provider documentation,
policy-as-code examples, testing tools, scanning tools, and CI/CD templates
commonly assume Terraform first.

This maturity matters for an enterprise Azure Landing Zone because the platform
will depend on repeatable module releases, provider behavior, remote state,
governance automation, and supportable operational patterns.

### Licensing Considerations

Terraform licensing and usage terms must be reviewed and accepted by the
organization before production use. Selecting Terraform does not remove legal or
procurement review responsibilities.

The decision accepts Terraform licensing risk in exchange for a mature
ecosystem, strong Azure alignment, and a lower operational support burden. If
licensing becomes unacceptable, OpenTofu migration can be evaluated as a
separate roadmap initiative.

### Long-Term Maintainability

Long-term maintainability is strong because the platform optimizes for one
engine, one validation path, one state management model, and one release
acceptance standard.

Modules should still avoid unnecessary engine-specific behavior when portable
HCL is sufficient. This keeps the codebase cleaner and may reduce future
migration cost, but it must not block Terraform-native functionality required
for the platform.

### Enterprise Adoption Considerations

Terraform is widely understood by enterprise infrastructure, cloud, security,
and operations teams. This improves onboarding, hiring, vendor support, peer
review, and integration with existing enterprise delivery practices.

The platform should document Terraform as the required tool clearly so workload
teams and platform contributors do not assume OpenTofu is supported.

### CI/CD Implications

CI/CD pipelines must install and use Terraform for formatting, initialization,
validation, planning, applying, state operations, and release acceptance.

Pipeline logs and release artifacts must record the Terraform version used.
OpenTofu jobs are not required for module acceptance, pull request validation,
or release approval.

### Module Compatibility Implications

Modules must be compatible with the approved Terraform version range and
provider constraints. Terraform behavior is authoritative for language features,
provider behavior, backend configuration, lock files, and state operations.

Portable HCL should be preferred when it is clear and sufficient. OpenTofu
compatibility must not be represented as supported unless a future ADR and
roadmap initiative establish an explicit compatibility test matrix.

### Upgrade Strategy

Terraform upgrades should be handled as scheduled platform upgrades:

- Pin an approved Terraform version range.
- Pin provider versions in root deployments according to the versioning
  standard.
- Test provider and module compatibility in CI before release.
- Validate representative root deployments before rollout.
- Promote upgrades through non-production before production.
- Record breaking changes in module release notes and standards.
- Update CI runner images, developer setup documentation, and runbooks as part
  of the upgrade.

## Option 2 - OpenTofu Only

### Description

Standardize all module development, validation, planning, applying, state
management, CI/CD workflows, and release acceptance on OpenTofu. Terraform would
not be a supported execution engine for platform modules or root deployments.

### Advantages

- Aligns with an open source, community-governed IaC engine.
- Avoids direct dependency on Terraform's current licensing model.
- Provides a clearer path if enterprise policy requires community-governed
  infrastructure tooling.
- Keeps the supported toolchain simple by validating one engine.

### Disadvantages

- Smaller enterprise ecosystem than Terraform.
- Less mature Azure-specific support story for enterprise platform delivery.
- Some vendor documentation, examples, support workflows, and third-party tools
  still assume Terraform.
- Azure Verified Module usage may require additional validation and exception
  handling.
- Teams and vendors already standardized on Terraform may face adoption
  friction.

### Operational Impact

Operational processes would still be simple because only one engine is
supported. However, platform engineers may need to translate Terraform-oriented
Azure documentation, support guidance, examples, and troubleshooting workflows
into OpenTofu usage.

For this platform, that translation work adds avoidable operational friction at
the same time the roadmap is trying to establish clear standards and reduce
implementation ambiguity.

### Ecosystem Maturity

OpenTofu has strong compatibility goals and an open source governance model, but
the surrounding enterprise Azure ecosystem is less mature than Terraform's.
Training materials, managed examples, module documentation, and vendor support
paths are more likely to assume Terraform.

### Licensing Considerations

OpenTofu may be attractive if enterprise policy prioritizes open source
licensing or community governance. Legal review would still be required, but
licensing is a primary advantage of this option.

### Long-Term Maintainability

Maintainability could be good if OpenTofu remains compatible with the provider
and module patterns the platform needs. The long-term concern is divergence from
Terraform-oriented Azure module ecosystems and vendor documentation.

### Enterprise Adoption Considerations

OpenTofu adoption may be a good fit for organizations that have already chosen
it as the enterprise standard. That is not the current platform decision.

Selecting OpenTofu now would introduce extra adoption and support questions
before the platform has completed its architecture, standards, toolchain, and
module milestones.

### CI/CD Implications

CI/CD pipelines would install and validate OpenTofu only. The platform would
need to confirm that formatting, initialization, validation, planning, applying,
state operations, security scanning, and release tooling support OpenTofu.

### Module Compatibility Implications

Modules would need to target OpenTofu behavior, not Terraform behavior. Azure
Verified Modules and third-party modules would require explicit validation
before adoption.

Terraform-specific examples, support documentation, and troubleshooting steps
would need to be rewritten or excluded.

### Upgrade Strategy

OpenTofu upgrades would need to be handled as scheduled platform upgrades:

- Pin an approved OpenTofu version range.
- Validate AzureRM, AzAPI, AzureAD, and supporting providers.
- Test representative Azure Verified Modules and platform modules before
  rollout.
- Promote upgrades through non-production before production.
- Track divergence from Terraform behavior as an explicit compatibility risk.

## Option 3 - Terraform and OpenTofu with Explicitly Tested Compatibility

### Description

Design reusable modules and root deployment patterns to remain compatible with
both Terraform and OpenTofu. CI/CD would validate an approved engine matrix for
modules, examples, and representative root deployments.

### Advantages

- Preserves future optionality between Terraform and OpenTofu.
- Reduces strategic risk from licensing, ecosystem, or product-direction
  changes.
- Allows adoption by teams with either Terraform or OpenTofu experience.
- Encourages module authors to avoid unnecessary engine-specific assumptions.

### Disadvantages

- Higher CI/CD complexity because more than one engine must be validated.
- Higher support burden because documentation, runbooks, and troubleshooting
  must account for more than one engine.
- Slower module release process when compatibility issues arise.
- More complex provider lock file and state handling guidance.
- Greater ambiguity around which engine is authoritative for plan, apply, and
  state operations.
- Azure Verified Module adoption may be slowed by compatibility checks that are
  not required for the platform's immediate delivery goals.
- Compatibility can erode unless it is continuously tested and enforced.

### Operational Impact

Dual compatibility increases operational complexity at the exact layer where
the roadmap needs clarity. Engineers must know which engine is used for
production applies, which engine generated a plan, which lock file is
authoritative, and whether a state operation is supported by both engines.

For an enterprise landing zone, ambiguity in state handling, release acceptance,
and recovery procedures is more expensive than the optionality gained from
dual-engine support.

### Ecosystem Maturity

Terraform remains the more mature ecosystem, especially for Azure enterprise
examples, Azure Verified Modules, provider documentation, and vendor support.
OpenTofu is credible, but dual support would require the platform to absorb
ecosystem differences rather than standardize on the strongest Azure-aligned
path.

### Licensing Considerations

Dual compatibility reduces single-engine licensing concentration risk. However,
Terraform licensing would still need review if Terraform is used in CI/CD or
production applies, and OpenTofu licensing would also need review.

This option does not eliminate licensing work; it increases the number of
supported tools that require governance.

### Long-Term Maintainability

Maintainability depends on disciplined compatibility testing. Without continuous
matrix validation, dual support becomes partial and unreliable. With continuous
matrix validation, the platform accepts ongoing test, documentation, and support
cost that does not directly advance the initial Azure Landing Zone delivery.

### Enterprise Adoption Considerations

Dual support may appeal to a broad audience, but it also makes the platform
harder to explain and operate. Enterprise consumers need a clear answer for
which tool is required, which release gates matter, and which state operations
are supported.

For this platform, a single authoritative engine provides a better adoption
path than a compatibility promise that must be qualified in every workflow.

### CI/CD Implications

CI/CD would need to install, cache, and validate more than one engine. Module
release acceptance would require matrix jobs, compatibility reporting, and
exception handling.

This adds runtime and support cost to every release. It also increases the risk
that pipeline failures reflect engine compatibility issues rather than platform
quality issues.

### Module Compatibility Implications

Modules would need to avoid features that are unsupported or behaviorally
different between Terraform and OpenTofu. Azure Verified Modules would require
engine compatibility review before adoption.

This constraint may be appropriate in a future migration assessment, but it is
not required for the platform's current implementation roadmap.

### Upgrade Strategy

Dual-engine upgrades would require a matrix-based process:

- Pin approved Terraform and OpenTofu version ranges.
- Test engine upgrades independently before combining them with provider
  upgrades.
- Validate representative module and root deployments under both engines.
- Track compatibility gaps and exceptions.
- Block module releases when compatibility failures affect supported use cases.

This strategy is more expensive than the platform currently needs.

## Recommendation

Adopt option 1: Terraform only.

Terraform is selected because it provides the clearest and most supportable path
for the Azure Landing Zone platform described in the roadmap. The platform must
deliver reusable modules, automated validation, remote state management,
controlled CI/CD, Azure Verified Module adoption, and repeatable foundation and
connectivity deployments. Those outcomes benefit from one authoritative engine
with mature Azure ecosystem alignment.

Terraform has broad enterprise adoption, mature Azure provider and module
ecosystem support, strong alignment with Azure Verified Modules, and extensive
documentation and tooling support. Selecting Terraform reduces the support and
testing burden by avoiding a dual-engine compatibility matrix. It also creates
clear operational ownership: Terraform is the engine used for validation,
planning, applying, state management, CI/CD workflows, release acceptance,
runbooks, and troubleshooting.

This decision intentionally favors operational clarity over optionality. The
platform should still avoid unnecessary Terraform-specific behavior when
portable HCL satisfies the requirement cleanly, because readable and portable
HCL is easier to maintain. However, OpenTofu compatibility is not a supported
contract, not a release gate, and not part of the required test matrix.

OpenTofu remains a future option if enterprise conditions change. If licensing,
organizational policy, product direction, or enterprise requirements justify it,
the platform can evaluate Terraform-to-OpenTofu migration or compatibility as a
separate roadmap initiative with its own scope, risks, tests, and rollback
plan.

## Future Consideration - Terraform-to-OpenTofu Migration Assessment

A Terraform-to-OpenTofu migration assessment may become a separate roadmap
initiative if business or technical drivers justify it. That assessment must
not be treated as an informal module compatibility check. It must evaluate the
full platform operating model.

The assessment should include:

- Terraform state compatibility and state migration or rollback procedures.
- AzureRM, AzAPI, AzureAD, and supporting provider compatibility.
- HCL syntax differences and behavior differences between approved engine
  versions.
- Azure Verified Module compatibility.
- Pipeline changes for validation, planning, applying, release acceptance, and
  artifact generation.
- Provider lock file handling and regeneration requirements.
- Backend behavior and remote state access patterns.
- Module test changes and required compatibility test coverage.
- Documentation and runbook updates.
- Production rollout and rollback planning.
- Criteria for accepting or rejecting migration.

## Consequences

- Terraform is the required engine for all platform validation, planning,
  applying, state management, CI/CD workflows, and release acceptance.
- M1 must define approved Terraform versions, provider constraints, lock file
  strategy, and CI runner setup.
- CI/CD pipelines must not require OpenTofu validation for platform compliance.
- Module release acceptance is based on Terraform validation and testing.
- Azure Verified Module adoption should optimize for Terraform-supported usage.
- Documentation, standards, examples, runbooks, and troubleshooting guidance
  must use Terraform terminology and commands.
- OpenTofu compatibility must not be advertised as supported unless a future ADR
  changes this decision.
- Portable HCL remains preferred where practical, but Terraform capability and
  platform supportability take precedence.

## Risks

- Terraform licensing, product direction, or procurement constraints may become
  unacceptable later.
- Teams that prefer OpenTofu may need to adjust to the platform's Terraform
  requirement.
- Terraform-specific patterns may accumulate and increase future migration cost.
- The platform may need a future migration project if organizational policy
  changes.
- Depending on one engine concentrates operational dependency on Terraform
  releases, provider compatibility, and Terraform state behavior.
- Legal or security review delays could affect platform delivery if Terraform
  approval is not completed early.

## Validation Criteria

The decision is valid when:

- Approved Terraform version ranges are documented.
- Required provider constraints are documented.
- CI/CD validation, planning, applying, and release acceptance use Terraform.
- Module standards define Terraform as authoritative.
- State management standards define Terraform as the supported state engine.
- Representative modules pass Terraform format, initialization, validation,
  linting, and tests.
- Representative root deployments can initialize and produce Terraform plans.
- Lock file handling is documented for modules, examples, and root deployments.
- Azure Verified Module adoption reviews confirm Terraform compatibility.
- Documentation and runbooks do not imply OpenTofu is supported.

## Conditions That Would Cause This Decision to Be Revisited

- Enterprise legal, procurement, security, or compliance policy prohibits or
  materially restricts Terraform use.
- Terraform product direction or licensing changes create unacceptable platform
  risk.
- The organization establishes OpenTofu as the enterprise-wide IaC standard.
- Azure Verified Modules, Azure providers, or required tooling materially shift
  support away from Terraform.
- Platform operating costs or support issues show that Terraform no longer
  meets enterprise requirements.
- A formal roadmap initiative is approved to assess Terraform-to-OpenTofu
  migration or compatibility validation.
- A production incident or recovery exercise identifies Terraform-specific
  behavior as an unacceptable operational risk.
