# Engineering Validation Standard

## Purpose

This standard defines the engineering validation requirements for reusable
Terraform modules in the Azure Platform Framework.

Reusable modules are platform product artifacts. They must be formatted,
validated, tested, linted, scanned, documented, and reviewable before they are
released for consumption by deployment repositories.

This standard defines the required validation outcomes and quality gates. It
does not create Terraform code, GitHub Actions workflows, runner
configuration, or release automation.

## Scope

This standard applies to reusable Terraform modules in
`azure-platform-modules`, including:

- Module source code.
- Module examples.
- Module tests.
- Module documentation.
- Pull request validation.
- Release validation.

This standard does not define root deployment plan/apply workflows for
`azure-platform-foundation` or `azure-platform-connectivity`. Deployment
repositories have additional state, identity, plan, approval, and apply
requirements.

Terraform is the authoritative validation engine for this platform.

## Philosophy

Engineering validation exists to make module changes safe, predictable, and
easy to review.

Required principles:

- Validation must be repeatable locally and in CI.
- Pull requests must fail early when formatting, syntax, linting, tests, or
  security checks fail.
- Release validation must be stricter than basic local validation.
- Reusable modules must prove their public contract through examples and tests.
- Validation must check module behavior without embedding tenant-specific
  values.
- Tooling must support review, not replace engineering judgment.
- Exceptions must be explicit, documented, and approved.
- CI implementation details are deferred to M1 and repository setup work.

## Required Tools

The standard toolchain for reusable module validation is:

- `terraform fmt`
- `terraform validate`
- `terraform test`
- TFLint
- Trivy
- GitHub Actions

Terraform is used for formatting, static validation, and native module tests.
TFLint is used for Terraform linting and provider-aware quality checks. Trivy
is used for infrastructure security scanning where applicable. GitHub Actions
is the expected CI/CD automation platform for pull request and release
validation.

Exact tool versions, installation methods, action versions, cache behavior, and
runner images are implementation details for M1 and module repository setup.
Those choices must align with the versioning standard and the accepted
Terraform toolchain ADR.

## Local Developer Workflow

Developers should run the same categories of checks locally before opening a
pull request.

Minimum local workflow:

1. Format Terraform files.
2. Initialize examples or test fixtures as required.
3. Validate module syntax and provider requirements.
4. Run native Terraform tests.
5. Run TFLint.
6. Run Trivy security scanning.
7. Review generated documentation or README changes where applicable.

Local validation is intended to shorten feedback loops. CI remains
authoritative for merge and release decisions.

Local validation must not require real production tenant IDs, subscription IDs,
credentials, private IP allocations, resource names, or secrets. Examples and
tests must use placeholder values or safe test fixtures.

## Pull Request Validation Workflow

Every pull request that changes reusable module code, examples, tests, or
module documentation must provide validation evidence.

Required pull request validation categories:

- Terraform formatting.
- Terraform initialization for supported examples or test fixtures.
- Terraform validation.
- Native Terraform tests.
- TFLint.
- Trivy security scanning.
- Documentation review for changed module contracts.
- Version impact assessment.

Pull request validation must confirm that the module can be reviewed without
requiring the reviewer to run undocumented commands or infer hidden behavior.

Pull requests must identify whether the change is:

- Breaking.
- Backward-compatible functionality.
- Backward-compatible fix.
- Documentation-only.
- Test-only.

Pull requests that change inputs, outputs, defaults, provider constraints,
resource behavior, naming, tagging, identity, networking, diagnostics, or
security posture must include enough validation evidence for reviewers to
understand the impact.

## Release Validation Workflow

Release validation applies before a reusable module version is published for
deployment repository consumption.

Release validation must include all pull request validation categories and
additional release checks appropriate to the module.

Required release validation:

- `terraform fmt` passes.
- `terraform validate` passes for supported examples and test fixtures.
- `terraform test` passes under the approved Terraform execution version.
- TFLint passes using the platform linting configuration.
- Trivy passes using the platform security scanning configuration, or approved
  exceptions are documented.
- Module examples initialize and validate where required.
- README documentation accurately describes inputs, outputs, examples,
  compatibility, limitations, and operational notes.
- Version impact is recorded according to semantic versioning.
- Breaking changes include migration and rollback guidance.
- Release notes summarize behavior changes and compatibility impacts.

Release validation must use immutable source content. A released module version
must not depend on mutable branches or unreviewed local files.

## Required Quality Gates

The following quality gates are required for reusable Terraform modules:

- Formatting gate: Terraform files are formatted with `terraform fmt`.
- Syntax gate: Terraform configuration validates successfully.
- Test gate: Native Terraform tests pass for required module behavior.
- Lint gate: TFLint reports no unapproved findings.
- Security gate: Trivy reports no unapproved high-risk findings.
- Documentation gate: README and examples match the module interface.
- Compatibility gate: Terraform and provider constraints align with the
  versioning standard.
- Contract gate: input, output, and behavior changes are classified for
  versioning impact.
- Repository gate: no Terraform state, plan files, credentials, local override
  files, or tenant-specific configuration are committed.

Quality gates may be implemented as separate jobs or combined jobs in GitHub
Actions. This standard does not prescribe workflow file structure.

## Failure Conditions

Validation must fail when any of the following are present:

- Terraform formatting changes are required.
- Terraform initialization fails for required examples or test fixtures.
- Terraform validation fails.
- Native Terraform tests fail.
- TFLint reports unapproved findings.
- Trivy reports unapproved findings that exceed the accepted risk threshold.
- Required examples are missing, invalid, or use real environment values.
- Required module documentation is missing or inconsistent with the module
  interface.
- Provider or Terraform constraints conflict with the versioning standard.
- A child module configures providers or backends.
- Terraform state files, plan files, credentials, secrets, or local override
  files are included.
- Tenant IDs, subscription IDs, production names, real private IP allocations,
  or organization-specific tag values are hard-coded.
- A breaking change lacks migration guidance, rollback notes, or version impact
  classification.

Approved exceptions must be documented in the pull request or release notes,
with the reason, scope, owner, and expected remediation path.

## Module Definition of Done

A reusable module is validation-complete when:

- It follows the Terraform module standard.
- It declares Terraform and provider requirements.
- It contains no provider or backend configuration.
- It exposes typed, documented, validated inputs.
- It exposes intentional documented outputs.
- It includes useful examples.
- It includes native Terraform tests for important behavior.
- It passes Terraform formatting and validation.
- It passes TFLint.
- It passes Trivy, or documented exceptions are approved.
- It contains no secrets, state, plan files, or tenant-specific values.
- It documents compatibility, limitations, and operational notes.
- It identifies semantic version impact.
- It can be consumed by a deployment repository without modifying module
  source.

## Future CI/CD Expectations

GitHub Actions is the expected automation platform for reusable module
validation.

Future CI/CD design should provide:

- Pull request checks for formatting, validation, tests, linting, security
  scanning, and documentation consistency.
- Release validation before immutable module tags are created.
- Clear check names that map to the quality gates in this standard.
- Consistent Terraform CLI usage aligned with the approved execution version.
- Provider dependency behavior aligned with the versioning standard.
- Minimal permissions for validation jobs.
- No long-lived credentials for routine validation.
- Artifact handling that avoids committing plans, state, secrets, or generated
  sensitive files.
- Reviewable logs that help engineers diagnose failures without exposing
  secrets.

Workflow file layout, runner selection, caching, matrix definitions, scheduled
checks, release automation, and exact action versions are deferred to M1 and
M3 implementation work.

## Best Practices

- Run validation locally before opening a pull request.
- Keep examples small, realistic, and free of real environment values.
- Add tests for input validation, important optional behavior, and expected
  outputs.
- Keep lint and security exceptions narrow and documented.
- Separate dependency upgrades from functional module changes where practical.
- Review provider and Terraform compatibility when module behavior changes.
- Treat validation failures as design feedback, not only tooling noise.
- Keep release notes focused on consumer impact.

## Anti-Patterns

The following patterns are prohibited:

- Skipping Terraform formatting because the change appears small.
- Treating `terraform validate` as a substitute for module tests.
- Publishing reusable modules that have never run `terraform test`.
- Ignoring TFLint or Trivy findings without documented approval.
- Embedding real tenant IDs, subscription IDs, private IP ranges, names, or
  tags in examples or tests.
- Adding provider or backend blocks to reusable child modules.
- Committing `.tfstate`, plan files, credentials, or local override files.
- Releasing breaking changes as patch or minor versions.
- Using mutable branches as release validation input.
- Hiding required validation steps in undocumented local scripts.

## Review Checklist

Use this checklist during module pull request and release reviews:

- [ ] Terraform is the authoritative validation engine.
- [ ] `terraform fmt` has passed.
- [ ] `terraform validate` has passed for required examples or fixtures.
- [ ] `terraform test` has passed for required module tests.
- [ ] TFLint has passed or findings have approved exceptions.
- [ ] Trivy has passed or findings have approved exceptions.
- [ ] The module follows the Terraform module standard.
- [ ] The module does not configure providers or backends.
- [ ] Terraform and provider constraints align with the versioning standard.
- [ ] Examples use placeholder values only.
- [ ] No state, plan files, secrets, credentials, or local overrides are
      committed.
- [ ] No tenant-specific or environment-specific values are hard-coded.
- [ ] README documentation matches the module interface.
- [ ] Inputs, outputs, defaults, and behavior changes are reviewed for version
      impact.
- [ ] Breaking changes include migration and rollback guidance.
- [ ] Validation evidence is included in the pull request or release record.

## Definition of Done

Engineering validation is compliant when:

- Reusable module changes run through local and pull request validation.
- Release validation passes before module versions are published.
- Formatting, validation, native tests, linting, and security scanning are
  required quality gates.
- GitHub Actions implements the required gates once CI/CD is introduced.
- Tool versions and dependency behavior align with the versioning standard.
- Validation results are visible to reviewers.
- Exceptions are documented, scoped, approved, and tracked.
- Modules can be released and consumed safely without hidden manual validation
  steps.
